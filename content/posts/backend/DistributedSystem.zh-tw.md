---
title: 分散式系統
date: 2026-01-05T00:00:00+08:00
menu:
  sidebar:
    name: 分散式系統
    identifier: DistributedSystem
    weight: 20
    parent: backend
tags:
- Distributed
- backend
- Stateless
---

# Distributed System

分散式系統設計，Net 8 服務，啟用 nginx upstream 從單機到多實例後，遇到的設計問題。

## 核心概念：Stateless Service

### 什麼是 Stateless？

**定義**：服務的每個實例不保留任何與特定請求或使用者相關的狀態資訊。

### 為什麼需要 Stateless？

```
┌─────────┐     ┌──────────────┐      ┌──────────┐
│ Client  │────▶│ Load Balancer│────▶│Instance A│
└─────────┘     └──────────────┘      ├──────────┤
                      │               │Instance B│
                      │               ├──────────┤
                      └─────────────▶ │Instance C│
                                      └──────────┘
```

當你的系統從 1 台機器擴展到 N 台時：

- **請求分散**：用戶的連續請求可能落在不同實例
- **實例隨時變動**：Auto-scaling 會新增/移除實例
- **故障轉移**：某個實例掛掉,流量會轉到其他實例

### ❌ 禁止使用的狀態儲存方式

| 方式                 | 為什麼不行                 | 實際案例                                           |
| -------------------- | -------------------------- | -------------------------------------------------- |
| `IMemoryCache`       | 只存在單一實例             | User A 的資料在 Instance A,但下次請求到 Instance B |
| `static` 變數        | 無法跨實例共享             | 計數器在每個實例都是獨立的                         |
| `Singleton` 內部狀態 | 每個實例有自己的 Singleton | Session 資料只存在啟動該請求的實例                 |
| 檔案系統             | 本地儲存                   | 上傳的檔案只在一台機器上                           |

### ✅ 狀態必須外部化

所有狀態都必須存放在**所有實例都能存取的外部系統**：

```
┌──────────┐     ┌─────────┐
│Instance A│────▶│         │
├──────────┤     │  Redis  │◀──── 所有狀態的單一來源
│Instance B│────▶│   or    │
├──────────┤     │   DB    │
│Instance C│────▶│         │
└──────────┘     └─────────┘
```

## 狀態管理：最容易出錯的地方

### 高風險狀態類型

#### 身份驗證流程狀態

**OAuth/OIDC 流程**

```csharp
// ❌ 錯誤：存在記憶體
public class AuthController : Controller
{
    private static Dictionary<string, OAuthState> _states = new();
    
    public IActionResult Login()
    {
        var state = Guid.NewGuid().ToString();
        _states[state] = new OAuthState { /* ... */ };  // 問題！
        // Redirect 到 OAuth provider...
    }
    
    public IActionResult Callback(string state)
    {
        if (!_states.TryGetValue(state, out var oauthState))  // 可能找不到
            return BadRequest();
    }
}
```

**問題**：

1. Login 請求到 Instance A,產生 `state` 存在 A 的記憶體
2. OAuth Provider callback 到 Instance B
3. Instance B 的 `_states` 裡沒有這個 `state` → 驗證失敗

**✅ 正確做法**

```csharp
public class AuthController : Controller
{
    private readonly IDistributedCache _cache;
    
    public async Task<IActionResult> Login()
    {
        var state = Guid.NewGuid().ToString();
        var oauthState = new OAuthState { /* ... */ };
        
        await _cache.SetStringAsync(
            $"oauth:state:{state}",
            JsonSerializer.Serialize(oauthState),
            new DistributedCacheEntryOptions 
            { 
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(10) 
            }
        );
        
        // Redirect...
    }
    
    public async Task<IActionResult> Callback(string state)
    {
        var json = await _cache.GetStringAsync($"oauth:state:{state}");
        if (json == null) return BadRequest();
        
        var oauthState = JsonSerializer.Deserialize<OAuthState>(json);
        await _cache.RemoveAsync($"oauth:state:{state}");  // 用完即刪
        // ...
    }
}
```

#### OTP / 驗證碼

```csharp
// ❌ 錯誤
private static Dictionary<string, (string Code, DateTime Expiry)> _otpCodes = new();

public void SendOTP(string phone)
{
    var code = GenerateOTP();
    _otpCodes[phone] = (code, DateTime.UtcNow.AddMinutes(5));
    // 發送簡訊...
}

public bool VerifyOTP(string phone, string code)
{
    if (!_otpCodes.TryGetValue(phone, out var data)) return false;
    return data.Code == code && data.Expiry > DateTime.UtcNow;
}
```

**✅ 正確做法**

```csharp
public class OTPService
{
    private readonly IDistributedCache _cache;
    
    public async Task SendOTP(string phone)
    {
        var code = GenerateOTP();
        
        await _cache.SetStringAsync(
            $"otp:{phone}",
            code,
            new DistributedCacheEntryOptions 
            { 
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5) 
            }
        );
        
        // 發送簡訊...
    }
    
    public async Task<bool> VerifyOTP(string phone, string code)
    {
        var storedCode = await _cache.GetStringAsync($"otp:{phone}");
        if (storedCode == null) return false;
        
        var isValid = storedCode == code;
        if (isValid)
        {
            await _cache.RemoveAsync($"otp:{phone}");  // 驗證成功後立即刪除
        }
        
        return isValid;
    }
}
```

#### 3. 防重放攻擊 (Replay Attack Prevention)

```csharp
// ❌ 錯誤
private static HashSet<string> _usedNonces = new();

public bool ValidateRequest(string nonce)
{
    if (_usedNonces.Contains(nonce)) return false;
    _usedNonces.Add(nonce);
    return true;
}
```

**✅ 正確做法**

```csharp
public class ReplayProtectionService
{
    private readonly IConnectionMultiplexer _redis;
    
    public async Task<bool> ValidateNonce(string nonce, TimeSpan validity)
    {
        var db = _redis.GetDatabase();
        var key = $"nonce:{nonce}";
        
        // SET NX (只在 key 不存在時設定)
        var wasSet = await db.StringSetAsync(key, "used", validity, When.NotExists);
        
        return wasSet;  // true = 首次使用,有效; false = 重複使用,無效
    }
}
```

```csharp
// ❌ 錯誤
private static ConcurrentDictionary<string, int> _requestCounts = new();

public bool CheckRateLimit(string userId)
{
    var count = _requestCounts.AddOrUpdate(userId, 1, (key, oldValue) => oldValue + 1);
    return count <= 100;  // 每分鐘 100 次
}
```

**問題**：每個實例有自己的計數器,實際上用戶可以發送 `100 × 實例數量` 的請求

**✅ 正確做法**

~~~csharp
public class RateLimitService
{
    private readonly IConnectionMultiplexer _redis;
    
    public async Task<bool> CheckRateLimit(string userId, int limit, TimeSpan window)
    {
        var db = _redis.GetDatabase();
        var key = $"ratelimit:{userId}:{DateTime.UtcNow.Ticks / window.Ticks}";
        
        var count = await db.StringIncrementAsync(key);
        
        if (count == 1)
        {
            await db.KeyExpireAsync(key, window);
        }
        
        return count <= limit;
    }
}
```

---

## Load Balancer 的真實行為

### ❌ 常見的錯誤假設

很多開發者會假設：
> "同一個使用者的請求會一直打到同一台伺服器"

### ✅ 現實情況

**Round Robin (預設)**
```
Request 1 → Instance A
Request 2 → Instance B
Request 3 → Instance C
Request 4 → Instance A
...
```

**Least Connections**
```
Request 1 → Instance A (目前連線數: 10)
Request 2 → Instance B (目前連線數: 5)  ← 選這個
Request 3 → Instance B (目前連線數: 6)
...
~~~

### Sticky Session 不是解決方案

雖然可以設定 Sticky Session (Session Affinity),但這有很多問題：

| 問題             | 說明                                |
| ---------------- | ----------------------------------- |
| **單點故障**     | 該實例掛掉,所有被綁定的用戶都會失敗 |
| **負載不均**     | 某些實例可能負載過高                |
| **擴展困難**     | 新增實例時無法立即分攤負載          |
| **不可靠**       | Cookie 可能被清除、IP 可能變動      |
| **雲原生反模式** | 違背容器化的設計理念                |

### 正確的心態

**永遠假設下一個請求會到不同的實例**

這樣設計出來的系統才能真正做到：

- 水平擴展
- 自動容錯
- 無縫部署

------

## ASP.NET Core DI Singleton 風險

Singleton 內部如有

- Dictionary
- ConcurrentDictionary
- In-memory state

```csharp
// ❌ 看似無害,實則有問題
public class CacheService
{
    private readonly ConcurrentDictionary<string, UserData> _cache = new();
    
    public void Set(string key, UserData value) 
    { 
        _cache[key] = value; 
    }
    
    public UserData Get(string key) 
    { 
        _cache.TryGetValue(key, out var value);
        return value;
    }
}

// Startup.cs
services.AddSingleton<CacheService>();
```

**問題**：

- 每個實例有自己的 `CacheService` Singleton
- `_cache` 只在單一實例內有效
- 跨實例完全看不到彼此的資料

### ✅ Singleton 的正確用法

**只能用於 Stateless 或外部資源包裝**

```csharp
// ✅ 正確：只包裝外部資源
public class RedisCacheService
{
    private readonly IConnectionMultiplexer _redis;  // 無狀態
    
    public RedisCacheService(IConnectionMultiplexer redis)
    {
        _redis = redis;
    }
    
    public async Task Set(string key, UserData value)
    {
        var db = _redis.GetDatabase();
        await db.StringSetAsync(key, JsonSerializer.Serialize(value));
    }
}

services.AddSingleton<IConnectionMultiplexer>(sp => 
    ConnectionMultiplexer.Connect("localhost:6379"));
services.AddSingleton<RedisCacheService>();
```

## 背景任務的分散式挑戰

### 問題場景

```csharp
// ❌ 危險的背景任務
public class OrderCleanupHostedService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // 清理過期訂單
            await CleanupExpiredOrders();
            
            await Task.Delay(TimeSpan.FromHours(1), stoppingToken);
        }
    }
}
```

**在單機環境**：正常運作

**在分散式環境**：
```
┌──────────┐
│Instance A│ ── 每小時執行清理
├──────────┤
│Instance B│ ── 每小時執行清理  ← 重複執行！
├──────────┤
│Instance C│ ── 每小時執行清理  ← 重複執行！
└──────────┘
~~~

### ✅ 解決方案

#### 方案 1：分散式鎖

```csharp
public class OrderCleanupHostedService : BackgroundService
{
    private readonly IDistributedLockProvider _lockProvider;
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await using (var @lock = await _lockProvider.TryAcquireLockAsync(
                "order-cleanup-job", 
                TimeSpan.FromMinutes(5)))
            {
                if (@lock != null)
                {
                    // 只有獲得鎖的實例會執行
                    await CleanupExpiredOrders();
                }
            }
            
            await Task.Delay(TimeSpan.FromHours(1), stoppingToken);
        }
    }
}
```

#### 方案 2：專屬 Worker Service
```csharp
┌──────────┐     ┌──────────┐
│  Web API │     │  Worker  │ ← 只有這個執行背景任務
│Instance A│     │ Service  │
├──────────┤     └──────────┘
│  Web API │
│Instance B│
├──────────┤
│  Web API │
│Instance C│
└──────────┘
~~~

#### 方案 3：訊息佇列

```csharp
// Producer (任一 Web Instance)
public async Task CreateOrder(Order order)
{
    await _db.SaveAsync(order);
    await _messageQueue.PublishAsync("order.created", order.Id);
}

// Consumer (專屬 Worker)
public class OrderProcessorWorker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await _messageQueue.SubscribeAsync("order.created", async (orderId) =>
        {
            await ProcessOrder(orderId);
        });
    }
}
```

## 冪等性設計

### 為什麼需要冪等性？

**現實場景**：

1. 使用者點擊"購買"按鈕
2. 網路延遲,沒看到回應
3. 使用者再點一次
4. 實際上兩個請求都到了伺服器 → **重複下單！**

### ✅ Idempotency Key 模式

```csharp
public class PaymentController : Controller
{
    private readonly IDistributedCache _cache;
    private readonly IPaymentService _paymentService;
    
    [HttpPost]
    public async Task<IActionResult> CreatePayment(
        [FromBody] PaymentRequest request,
        [FromHeader(Name = "Idempotency-Key")] string idempotencyKey)
    {
        if (string.IsNullOrEmpty(idempotencyKey))
        {
            return BadRequest("Idempotency-Key header is required");
        }
        
        var cacheKey = $"idempotency:{idempotencyKey}";
        
        // 檢查是否已處理過
        var cachedResult = await _cache.GetStringAsync(cacheKey);
        if (cachedResult != null)
        {
            return Ok(JsonSerializer.Deserialize<PaymentResult>(cachedResult));
        }
        
        // 處理付款
        var result = await _paymentService.ProcessPayment(request);
        
        // 儲存結果 (24小時)
        await _cache.SetStringAsync(
            cacheKey,
            JsonSerializer.Serialize(result),
            new DistributedCacheEntryOptions 
            { 
                AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(24) 
            }
        );
        
        return Ok(result);
    }
}
```

**Client 使用方式**：

```javascript
const idempotencyKey = uuidv4();

fetch('/api/payment', {
    method: 'POST',
    headers: {
        'Idempotency-Key': idempotencyKey,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(paymentData)
});

// 即使重試多次,也只會處理一次
```

## 時間同步與過期處理

### **問題**：依賴應用程式時間

- Instance A 時間: `2025-01-07 10:00:00` Instance B 時間: `2025-01-07 09:59:30` (慢30秒)
- Token 在 A 創建,在 B 驗證 → 可能誤判為過期
- 雖然可以透過容器讀取宿主機時區來達到容器間時間區一致，但時間判斷處理也是`狀態`的一種，``狀態外部化`仍是 Stateless 重要觀念。

### 讓儲存系統處理過期

**Redis TTL**

```csharp
public class TokenService
{
    private readonly IConnectionMultiplexer _redis;
    
    public async Task StoreToken(string token, TokenData data)
    {
        var db = _redis.GetDatabase();
        await db.StringSetAsync(
            $"token:{token}",
            JsonSerializer.Serialize(data),
            TimeSpan.FromMinutes(30)  // Redis 自動過期
        );
    }
    
    public async Task<bool> IsValid(string token)
    {
        var db = _redis.GetDatabase();
        var data = await db.StringGetAsync($"token:{token}");
        return !data.IsNullOrEmpty;  // Redis 已處理過期邏輯
    }
}
```

### 解決方案：Correlation ID + Distributed Tracing

**ASP.NET Core Middleware**

```csharp
public class CorrelationIdMiddleware
{
    private readonly RequestDelegate _next;
    
    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers["X-Correlation-ID"].FirstOrDefault() 
                         ?? Guid.NewGuid().ToString();
        
        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        
        using (LogContext.PushProperty("CorrelationId", correlationId))
        {
            await _next(context);
        }
    }
}
// Program.cs
builder.Host.UseNLog();
app.UseMiddleware<CorrelationIdMiddleware>();
```

#### nlog.config

~~~xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd">
  
  <targets>
    <!-- Console -->
    <target name="console" xsi:type="Console"
            layout="[${longdate}] [${level:uppercase=true}] [${event-properties:item=CorrelationId}] [${event-properties:item=InstanceId}] ${logger} - ${message} ${exception}" />
    
    <!-- JSON File -->
    <target name="jsonfile" xsi:type="File" 
            fileName="/var/log/app/${shortdate}.json">
      <layout xsi:type="JsonLayout" includeEventProperties="true">
        <attribute name="timestamp" layout="${longdate}" />
        <attribute name="level" layout="${level:uppercase=true}" />
        <attribute name="correlationId" layout="${event-properties:item=CorrelationId}" />
        <attribute name="instanceId" layout="${event-properties:item=InstanceId}" />
        <attribute name="logger" layout="${logger}" />
        <attribute name="message" layout="${message}" />
        <attribute name="exception" layout="${exception:format=toString}" />
      </layout>
    </target>

    <!-- Seq -->
    <target name="seq" xsi:type="Seq" serverUrl="http://seq:5341">
      <property name="CorrelationId" value="${event-properties:item=CorrelationId}" />
      <property name="InstanceId" value="${event-properties:item=InstanceId}" />
    </target>
  </targets>

  <rules>
    <logger name="*" minlevel="Info" writeTo="console,jsonfile,seq" />
  </rules>
</nlog>

**日誌輸出範例**：
```
[2025-01-07 10:30:15] [INFO] [abc-123-def] [instance-A] OrderController - Creating order for user user-456
[2025-01-07 10:30:16] [INFO] [abc-123-def] [instance-A] OrderService - Order order-789 created
~~~

## 流量控制的正確做法

### 記憶體型 Rate Limiting

**問題**：每個實例獨立計數 → 實際限制是 `100 × 實例數`

```csharp
services.AddMemoryCache();
services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("api", opt =>
    {
        opt.Window = TimeSpan.FromMinutes(1);
        opt.PermitLimit = 100;
    });
});
```

### Redis-based Rate Limiting

```csharp
public class RedisRateLimitingMiddleware
{
    private readonly IConnectionMultiplexer _redis;
    
    public async Task InvokeAsync(HttpContext context)
    {
        var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (string.IsNullOrEmpty(userId))
        {
            await _next(context);
            return;
        }
        
        var allowed = await CheckRateLimit(userId, limit: 100, window: TimeSpan.FromMinutes(1));
        
        if (!allowed)
        {
            context.Response.StatusCode = 429;
            await context.Response.WriteAsync("Too Many Requests");
            return;
        }
        
        await _next(context);
    }
    
    private async Task<bool> CheckRateLimit(string key, int limit, TimeSpan window)
    {
        var db = _redis.GetDatabase();
        var redisKey = $"ratelimit:{key}:{DateTimeOffset.UtcNow.ToUnixTimeSeconds() / (int)window.TotalSeconds}";
        
        var count = await db.StringIncrementAsync(redisKey);
        if (count == 1)
        {
            await db.KeyExpireAsync(redisKey, window);
        }
        
        return count <= limit;
    }
}
```

### Nginx Gateway 層處理 Rate Limiting

```nginx
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/m;

server {
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

**Kong / Azure API Management**

- 原生支援分散式 Rate Limiting
- 不佔用應用程式資源

## 檢查清單與遷移步驟

### 檢查清單

| 檢查項目       | 要求                                                         | ✓    |
| -------------- | ------------------------------------------------------------ | ---- |
| **狀態管理**   | ❌ 不使用 `IMemoryCache`   ✅ 使用 `IDistributedCache` (Redis) | ☐    |
|                | ❌ 不使用 `static` 變數儲存狀態                               | ☐    |
|                | `Singleton` 服務都是無狀態的                                 | ☐    |
| 背景任務       | `IHostedService` 有分散式鎖或移到專屬 Worker                 | ☐    |
|                | 排程任務只在單一實例執行                                     | ☐    |
| **冪等性**     | 關鍵操作(付款/下單)支援 Idempotency Key                      | ☐    |
|                | API 能處理重複請求                                           | ☐    |
| 時間處理       | ✅ 使用 Redis TTL 或 DB `GETUTCDATE()`  ❌ 不依賴應用程式 `DateTime.UtcNow` | ☐    |
| Log 監控與追蹤 | 實作 Correlation ID                                          | ☐    |
|                | Log 包含足夠的上下文資訊 包含 CorrelationId, InstanceId, UserId | ☐    |
|                | 有分散式追蹤系統( Seq/Jaeger/Zipkin )                        | ☐    |
| 流量控制       | ✅ Rate Limiting 使用 Redis 或 API Gateway❌ 不使用記憶體型限流 | ☐    |
| 檔案處理       | ✅ 使用 S3/Azure Blob/NFS   ❌ 不存在本地檔案系統              | ☐    |

### 遷移步驟

#### 階段 1：評估與規劃

1. 列出所有使用記憶體狀態的地方
2. 識別背景任務
3. 檢視 Singleton 服務
4. 確認 Redis/SQL 容量規劃

#### 階段 2：基礎設施準備

1. 部署 Redis Cluster (高可用)
2. 設定 Load Balancer
3. 建立分散式追蹤系統
4. 準備監控儀表板

#### 階段 3：程式碼改造

1. 替換 `IMemoryCache` → `IDistributedCache`
2. 遷移 OAuth/OTP 到 Redis
3. 加入 Idempotency Key 支援
4. 實作 Correlation ID

#### 階段 4：測試

1. 多實例環境測試
2. 重複操作行為測試

## 總結

從單機到分散式系統的核心轉變：

| 單機思維             | 分散式思維         |
| -------------------- | ------------------ |
| 記憶體是可靠的       | 記憶體是暫時的     |
| 時間是一致的         | 時間可能不同步     |
| 狀態可以本地保存     | 狀態必須外部化     |
| 背景任務只執行一次   | 需要協調機制       |
| 請求會回到同一台機器 | 請求可能到任何機器 |

設計原則：

> 永遠假設你的應用程式會在 N 個隨時變動的實例上運行,設計時不依賴任何單一實例的本地狀態
