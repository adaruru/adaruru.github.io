---
title: JWT
date: 2025-11-11T00:00:00+00:00
menu:
  sidebar:
    name: JWT
    identifier: JWT
    weight: 20
    parent: backend
tags:
- backend
- jwt
- Stateless
---

# JWT

JWT 架構與實務筆記

## 概念整理

### JWT 基礎概念

#### JWT 組成（Header、Payload、Signature）

##### Header 

Header 描述 **這顆 Token 用什麼演算法簽的，以及它是什麼型別**。

常見 alg

| alg   | 說明                      |
| ----- | ------------------------- |
| HS256 | HMAC + SHA256（對稱金鑰） |
| RS256 | RSA + SHA256（非對稱）    |
| ES256 | ECDSA + SHA256（非對稱）  |

##### Payload

Payload 是真正要傳遞的身分與權限資料，它是 JWT 的核心。

Payload 分三類 Claim：

1. Registered Claims（標準欄位）
2. Public Claims（公開欄位）
3. Private Claims（自定義欄位）: 自由取名

標準欄位

| 欄位 | 意義                     |
| ---- | ------------------------ |
| iss  | Token 發行者             |
| sub  | 主體（通常是 userId）    |
| aud  | 接收者                   |
| exp  | 過期時間（必須）         |
| nbf  | 何時才有效               |
| iat  | 簽發時間                 |
| jti  | Token ID（可用於黑名單） |

Public Claims（公開欄位） 

必須避免與標準欄位衝突。

例如：

```
{
  "email": "a@b.com",
  "company": "OpenAI"
}
```

Private Claims（自定義欄位）

你系統自己用的：

```
{
  "userId": 8123,
  "role": "admin",
  "tenantId": "corp-99",
  "permissions": ["order.read", "order.write"]
}
```

##### Signature

它保證了三件事

1. 完整性:  Header / Payload 沒被改
2. 真實性:  Token 是你系統簽的
3. 不可否認: 只有你有 private key

#### 加密與簽章差異

加密: 不讓你看
簽章: 讓你看到，但你不能亂改，改過就不信任你

| 項目         | 加密 Encryption           | 簽章 Signature                              |
| ------------ | ------------------------- | ------------------------------------------- |
| 目的         | 隱藏內容                  | 證明內容沒被改、來源正確                    |
| 解密者       | 擁有密鑰的人              | 任何人都可驗證                              |
| 保護的是     | 機密性（Confidentiality） | 完整性（Integrity）與真實性（Authenticity） |
| 是否可讀     | ❌ 不可讀                  | ✅ 可讀                                      |
| 是否可被修改 | 可以被改但會解不開        | 一改就驗不過                                |
| JWT 用途     | JWE                       | JWS                                         |

JWS/JWE

| 標準                      | 用途                          |
| ------------------------- | ----------------------------- |
| JWS (JSON Web Signature)  | 簽章（99% 的 JWT SSO 用這個） |
| JWE (JSON Web Encryption) | 加密                          |

#### Token 格式與傳遞方式

1. Restful Header Authorization

   Authorization: Bearer <access_token>

2. Domain Cookie（HttpOnly + Secure）

   1. refresh_token
   2. 禁止 localStorage
   3. 禁止 sessionStorage
   4. 禁止 cookie（非 HttpOnly）

### Access Token vs Refresh Token

- 各自用途與生命週期
- 為何需要 Refresh Token
- 過期重簽策略

### Access Token 儲存

localStorage 存在前端，後端只會驗證，不保留資料

### Refresh Token 儲存

#### 傳統單體架構（Monolithic Web App）

單體部署、後端渲染、以 Cookie 為主的舊式架構

- 通常將 Token 儲存在 Cookie 中

  無法被 JS 存取，避免 XSS

- 防 CSRF

  配合 SameSite=Strict 或 Lax, 跨網域情境下要正確設定 `domain`、`secure`、CORS headers，否則 Cookie 不會附帶

  - withCredentials: true
  - Access-Control-Allow-Credentials: true
  - Set-Cookie` 必須 `SameSite=None; Secure
  - Server 端需正確設置 `CORS`

#### 前後端分離架構（Frontend-Backend Decoupled）

SPA/Frontend 與 API Backend 分離的模式，符合現代主流實作

透過 Header 傳遞 AccessToken 與 RefreshToken。

前端儲存於 JS-controlled storage（如 memory 或 secure localStorage），並進行加密保護。

```http
GET /api/resource
Authorization: Bearer <access_token>
X-Refresh-Token: <refresh_token>   <-- 若 Access Token 過期，自動帶入 refresh token
```

##### 前後端分離架構不用 Cookie 的原因

1. CSRF 風險

   即便設了 `SameSite=Strict`，若誤設或遺漏，攻擊者仍可透過社交工程誘導使用者觸發非預期的 API 呼叫。

2. 可控性低

   Cookie 會自動附加至所有請求，前端難以明確控制是否送出 Refresh Token。

3. 跨網域問題繁雜

   若使用多網域架構，Cookie 的 `domain`、`secure`、CORS 配置會變得非常繁瑣。

4. 多網域架構不易維護

   每個子網域都需獨立處理 Cookie 相容性與安全策略。

##### Secure Token 建議（Memory vs Encrypted LocalStorage）

前後端分離架構 Secure Token 儲存建議

1. Memory 
2. Encrypted LocalStorage）

###### Secure Token in Memory (優選)

- 無持久性，關閉頁籤即失效，但安全性較高。
- 搭配 Refresh Token 自動續期機制，可達成無縫體驗，避免用戶體驗下降。

###### Secure Token in Encrypted LocalStorage (次選)

- 可持久儲存登入狀態，適合 SPA。
- 暴露於 XSS 攻擊風險，需加強輸入過濾與前端加密保護。

## 實作應用

### 實作設計準則

1. JWT 只放「授權判斷所需的最小資料」，絕不放 PII 或可變動資料（例如餘額、點數）

   - 放什麼辨識項目: userId、role、scope、tenant

   - 不放機敏資料: 密碼、金額、動態資料

2. `sub` + `exp` 是不可缺的最小組合

3. Token 壽命: 短效( 15 min /30 min ) Access 搭配 長效 Refresh  ( 7 days )

4. 撤銷: 用 jti + blacklist 或短效 Token

### JWT 驗證流程實作（ASP.NET Core）

- JwtBearer 驗證組態與 TokenValidationParameters 設定
- 自定 Events 行為（如 Token 過期處理、自動觸發 Refresh）
- 搭配 Middleware 達成透明更新 Token

### Refresh Token 實作細節

- Redis 快取存放 Refresh Token，利於效能與集中管理
- 使用精簡 Profile 模型或直接解析 Claims 驗證 RefreshToken 合法性
- 驗證時應同時檢查 CreatedDate 是否過期、ReplacedToken 是否已被替換

#### Refresh Token 儲存選型（Relational vs NoSQL）

討論的重點

1. 是否有高效能要求
2. 是否有 Audit 要求

**建議結論：**

| 考量面向                  | 傾向使用 | 補充說明                                                     |
| ------------------------- | -------- | ------------------------------------------------------------ |
| 即時效能與彈性擴展需求    | Redis    | 支援 TTL、自動過期<br />Revoke 控制<br />適合分散式、<br />高併發架構<br />高效能、低延遲 |
| 一致性與審計/歷程記錄需求 | SQL      | Audit 追蹤、資料完整性要求、強一致性要求                     |
| 結合安全與擴展性最佳化    | 混合使用 | Redis 存放快速驗證用 Token 資訊<br />SQL 保留歷史與長期稽核資料 |

### Token 安全策略

- 防止 Token 竊取：強制加密儲存、限定 IP、裝置識別
- 強制登出：支援 Revoke 機制與 RefreshToken 替換記錄
- Sliding Expiry 雖可延長存活期，但需配合 Refresh Token 管理避免濫用

### JWT Base SSO Server

#### concepts

單點登入（SSO）是一種統一驗證流程，允許使用者登入一次，即可存取多個系統( 多個驗證消費者 Service Providers）。

##### 單點登出1-SSO 主導、子系統被動同步

Front-channel（iframe + postMessage）

1. 子系統的 JavaScript 透過 postMessage 監聽，並在監聽到的時候子系統被動登出，立即清 token + 導回登入
2. SSO Server 在 Session 消失時廣播: parent.postMessage("SSO_LOGOUT", "*");

##### 單點登出2-後端強制通知

1. 使用者登出

2. SSO Server 廣播，Server 呼叫所有子系統(Api)

3. 子系統後端 驗證 SSO 簽章，刪除所有 session

4. 依賴子系統session store 不適用「純 JWT 無狀態」系統

##### 單點登出3-OAuth2 / OpenID Connect

Token-Centric SLO

登出時

```
SSO Server
→ 刪除 Refresh Token
→ 刪除 SSO Session
```

#### 安全建議

1. Key 管理架構

   子系統只保存 public key 不接觸 private key，且不能持有可簽 Token 的能力

   ```
   SSO Server
    ├── Private Key (簽 JWT)
    └── JWKS Endpoint (public keys)
             ↓
         All Apps
   ```

2. 強制欄位檢查

3. Ldap 混合使用

4. Zero Trust 子系統: 永遠不信 client，只信 SSO 的 signature，這確保了身分驗證的信任邊界（Trust Boundary）與跨系統的一致性。

## 延伸討論

### 分散式架構下的 Token 管理：Stateless 優於 Centralized，除非有即時撤銷需求

### 與 OAuth2 整合：可透過 JWT 實作簡化版的授權流程

### 多角色支援與 Tenant 隔離：可透過 Claims 動態識別使用者身分與資料區隔