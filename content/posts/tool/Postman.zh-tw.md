---
title: "Postman"
date: 2026-01-25T00:00:00+00:00
menu:
  sidebar:
    name: Postman
    identifier: Postman
    parent: Tool
    weight: 10
hero: images/posts/postman.png
tags:
- Postman
- Api
- Tool
---

# Postman 使用筆記

## Postman 功能

登入[^Postman]，撰寫測試

### 安裝 Postman Desktop

不要使用線上版，會有資料同步卡頓的問題

### 登入 Postman

### Workspaces

建立或選擇 Workspaces

![image-20250415103252077](/posts/tool/attach/Postman/image-20250415103252077.png)

### 測試命名 

Controller/Action+ 如有其他必要說明

![image-20250415110006880](/posts/tool/attach/Postman/image-20250415110006880.png)

### 環境變數

可以自訂各種會因環境變化必須自訂的參數

![image-20250415110450016](/posts/tool/attach/Postman/image-20250415110450016.png)

#### 環境變數抽換參數

比如新增環境變數 baseUrl，測試 api baseUrl 不要寫死在測試範例，使用環境變數已切換實際測試連線

![image-20250415110554020](/posts/tool/attach/Postman/image-20250415110554020.png)

![image-20250415110401597](/posts/tool/attach/Postman/image-20250415110401597.png)

### Mock Server

前端開發超前後端進度時

想要模擬溝通後端 api 使用 [^postmanServer]

1. 新增測試資料

   ![image-20250415112539998](/posts/tool/attach/Postman/image-20250415112539998.png)

2. 撰寫測試資料

   ![image-20250415112605486](/posts/tool/attach/Postman/image-20250415112605486.png)

3. 查看 mock url

   ![image-20250415112820542](/posts/tool/attach/Postman/image-20250415112820542.png)

4. 前端開發使用測試 Api

   https://64680c7d-60de-4ffe-abb7-462f3958d8df.mock.pstmn.io/Car/GetCar

## 開發相關

### Nuxt API 開發

每開發一支 api 就要登入 Postman 新增一個測試資料
[^Postman]: 點選這裡看說明

api 連線 mock : postman mock

```typescript
nitro: {
    devProxy: {
      '/mockStoreApi': {
        target: "https://64680c7d-60de-4ffe-abb7-462f3958d8df.mock.pstmn.io",
        changeOrigin: true,
        secure: false,
      },
```

#### api 連線 mock : postman mock

1. 模擬資料建立

   [^postmanServer]: 點選這裡看說明

2. api 執行 url 參數多一個 ismock true

```typescript
// dev 或 uat 測試
return ClientUseFetch(Url(controller, 'LineAuth'), {
    method: 'GET',
    query: data
  })
// dev postman 測試
return ClientUseFetch(Url(controller, 'LineAuth',true), {
    method: 'GET',
    query: data
  })
```

#### composables\apiUtils.ts 控制是否走 postman 取測資

```typescript
function Url(controller, endpoint, isMock = false) {
  const { public: $config } = useRuntimeConfig();
  let base = $config.mainApi;
  if (isMock) {
    base = $config.mockApi;
  }
  const url = `/${base}/${controller}/${endpoint}`;
  return url;
}
```

