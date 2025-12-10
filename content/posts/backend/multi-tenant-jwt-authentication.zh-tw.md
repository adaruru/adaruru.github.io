---
title: 多租戶 JWT 驗證架構
menu:
  sidebar:
    name: 多租戶 JWT 驗證架構
    identifier: multi-tenant-jwt-authentication 
    weight: 20
    parent: backend
tags:
- backend
- design-pattern
- jwt
---

# Multi-Tenant JWT Authentication 

多租戶（Multi-Tenant）JWT 雙 Token 認證機制（Access Token + Refresh Token），支援商店用戶（Store）與客戶用戶（Customer）兩種身份類型的獨立認證流程。透過泛型設計與介面抽象，實現了可擴展的多角色登入系統，並結合 Redis 快取機制管理 Refresh Token，提供安全且高效的身份驗證解決方案。

## Key Features
- ✅ 雙 Token 機制：Access Token (短效) + Refresh Token (長效)
- ✅ 多角色支援：Store User 與 Customer User 獨立認證
- ✅ 自動 Token 更新：Middleware 自動偵測過期並刷新
- ✅ 分散式快取：Redis 管理 Refresh Token 狀態
- ✅ 安全性強化：HttpOnly Cookie + CSRF 防護
- ✅ 可擴展架構：新增用戶類型僅需實作兩個介面


## 1. Adapter Pattern (適配器模式)

位置： StoreInfo 、 CustomerInfo、 IClaimAdapter.cs

```csharp
public interface IClaimAdapter<CurrentUser>
{
    IEnumerable<Claim> GetClaims();
    static abstract CurrentUser? FromClaimsPrincipal(ClaimsPrincipal principal);
}
```

```csharp
public class CustomerContext : AppDataContext<CustomerInfo>
{
    public CustomerContext(IHttpContextAccessor httpContextAccessor, IBaseLogger log)
        : base(httpContextAccessor, log) { }
    public override CustomerInfo? CurrentUser
    {
        get
        {
            var user = _httpContextAccessor.HttpContext?.User;
            return CustomerInfo.FromClaimsPrincipal(user);
        }
    }
}

public class CustomerInfo : IClaimAdapter<CustomerInfo>
{
    public IEnumerable<Claim> GetClaims(){}
    public static CustomerInfo? FromClaimsPrincipal(ClaimsPrincipal principal){}
}
```

說明：

- 領域物件 (StoreInfo/CustomerInfo) ←→ IClaimAdapter ←→ JWT Claims System
- 被適配者（Adaptee）：領域物件 `StoreInfo`、`CustomerInfo`（有自己的屬性結構）
  - 實作相同的 `IClaimAdapter<T>` 介面

- 目標介面（Target）：JWT Claims System（使用 `Claim` 集合）
- 適配器（Adapter）：`IClaimAdapter<T>` 介面
  - 將領域物件（StoreInfo/CustomerInfo）與 JWT Claims 系統進行雙向轉換
  - `GetClaims()`: 將物件轉換為 Claims（用於生成 JWT token）
  - `FromClaimsPrincipal()`: 將 ClaimsPrincipal 還原為原始物件（用於解析 JWT token）生成 user context
  - 轉換邏輯緊跟 user info，方便維護轉換邏輯與欄位新刪

## 2. Template Method Pattern (模板方法模式)

位置： AppDataContext.cs

```csharp
public abstract class AppDataContext<TUserInfo>
{
    public abstract TUserInfo? CurrentUser { get; }
    // 提供共用功能
    public string GetLoginIp() { ... }
    public void ClearContext() { ... }
}
```

說明：

- 父類別 `AppDataContext<TUserInfo>` 定義了共用的基礎結構和方法
- 子類別 [StoreContext](vscode-webview://0nlu7ssdt85f5uhh8ljum9dikvvs8gsel4mc6uulua9pmps9lc22/Lib/Infra/Context/StoreContext.cs:8) 和 [CustomerContext](vscode-webview://0nlu7ssdt85f5uhh8ljum9dikvvs8gsel4mc6uulua9pmps9lc22/Lib/Infra/Context/CustomerContext.cs:7) 必須實作抽象屬性 `CurrentUser`
- 共用方法（`GetLoginIp`、`ClearContext`）由父類別提供，避免重複程式碼
- 底層˙ entityService 讀取 CurrentUser 自動更新當前 repository createUser 或 modifyUser，不用再各自服務額外寫

## 3. Middleware Pattern (中介軟體模式)

位置： RefreshTokenMiddleware.cs

```csharp
public class RefreshTokenMiddleware
{
    private readonly RequestDelegate _next;
    public async Task Invoke(HttpContext context, ...) { ... }
}
```

說明：

- 在 ASP.NET Core 管線中攔截請求
- 自動檢查 access token 是否過期，並使用 refresh token 自動更新
- 遵循責任鏈模式，處理完後呼叫 `_next(context)` 傳遞給下一個中介軟體
- 架構層級控制：在 pipeline 配置層級就決定 middleware 是否執行
- 關注點分離：RefreshTokenMiddleware 專注於 token 刷新邏輯
- 不再 Authentication Handler 架構下，需要額外注意跳過匿名路由 AllowAnonymous 或其他自訂規則

## 4. Factory Method Pattern (工廠方法模式)

位置： JwtAuthServices.cs

```csharp
public class JwtAuthServices
{
    public string GenerateAccessToken(IEnumerable<Claim> genClaims, ...) { ... }
    public string GenerateRefreshToken() { ... }
}
```

說明：

- 集中管理 JWT token 的建立邏輯
- 封裝複雜的 token 生成細節（簽章、過期時間、Claims 處理等）
- 各 token 過期時間、重試次數限制等參數，抽離至 appsetting 方便不同環境測試 jwt 效期實際效果

## 總結

- 關注點分離：JWT 處理、使用者上下文、Claims 轉換各司其職
- 開放封閉原則：容易擴充新的使用者類型（只需實作 `IClaimAdapter` 和繼承 `AppDataContext`）
- 依賴反轉：依賴抽象介面而非具體實作
