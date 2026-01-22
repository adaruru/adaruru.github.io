---
title: Store
date: 2026-01-22T00:00:00+00:00
menu:
  sidebar:
    name: Store
    identifier: Store
    weight: 20
    parent: frontend
hero: posts/frontend/attach/store/store.png
tags:
- frontend
- store
- state
- localStorage
---

# Store 狀態管理

### Store 

全局狀態管理（state management）

核心職責：

- 管理記憶體 state (在單次 session 中統一管理記憶體中的資料結構)
- 可觀察/可修改的接口，供外部 get/set store porperties

相對應的技術工具

- React 生態：Redux、`Zustand`、MobX、Recoil、Jotai、Effector
- Vue 生態：Vuex、`Pinia`
- Angular 生態：NgRx、Akita

Persistence：可選功能，用於跨 reload / 瀏覽器存活，沒有 Persistence 也能跨 component / module 共用 state，如服務是靜態的 SPA，就不需要考慮 Persistence 

### Store 用途

| 能力                 | 說明                                | 範例                                                         |
| -------------------- | ----------------------------------- | ------------------------------------------------------------ |
| 全局單例管理 state   | 在多個 component 間共享同一份 state | 多個 component 都能讀 `store.count`，更新一處其他地方立即反應 |
| Action orchestration | 集中定義行為（副作用）              | `store.increment()` 自動更新 state，而所有使用者都看到最新值 |
| Computed / getters   | 派生 state                          | `store.doubleCount = store.count  2`                         |
| Hooks / 監聽器       | 訂閱 state 變化                     | `store.$subscribe((mutation, state) => { console.log(state.count) })` |
| 組織邏輯             | 把相關 state 與操作放在一起         | 比較清楚的模組邏輯，避免散落 function                        |

### Store Persistence

state 只存在於瀏覽器的記憶體 ( RAM )，生命週期受限於頁面執行環境( 刷新頁面、關閉分頁或跨頁，記憶體會被清空 )。

web browser 有那些 Storage 可以提供持久化>> f12 >>Application 就可以看到，有可能會因為不同的瀏覽器支援不同儲存方式

| Feature                      | Pinia w/ plugin                                              | Zustand persist                                              |
| ---------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| localStorage                 | ✅                                                            | ✅（預設）([zustand.docs.pmnd.rs](https://zustand.docs.pmnd.rs/integrations/persisting-store-data?utm_source=chatgpt.com)) |
| sessionStorage               | ✅                                                            | ✅([zustand.docs.pmnd.rs](https://zustand.docs.pmnd.rs/integrations/persisting-store-data?utm_source=chatgpt.com)) |
| cookie                       | ⚠️（plugin 特定實作）([Claire's Notes](https://clairechang.tw/2023/08/16/nuxt3/nuxt-v3-state-management-persistedstate/?utm_source=chatgpt.com)) | ❌（需自訂 storage）                                          |
| IndexedDB                    | ⚠️（需 plugin/自訂）([Stack Overflow](https://stackoverflow.com/questions/74777210/storing-in-indexeddb-using-pinia?utm_source=chatgpt.com)) | ✅（透過 createJSONStorage 自訂）([zustand.docs.pmnd.rs](https://zustand.docs.pmnd.rs/integrations/persisting-store-data?utm_source=chatgpt.com)) |
| Async Storage（RN / custom） | ⚠️（需 plugin）                                               | ✅（支援 async storage）([zustand.docs.pmnd.rs](https://zustand.docs.pmnd.rs/integrations/persisting-store-data?utm_source=chatgpt.com)) |
| 自定義 storage               | ✅                                                            | ✅                                                            |

### Action 是否該進 Store 判斷標準

「技術上真正需要放進 store 的 action」與「只是風格或偏好」。

只有當「行為需要活得比 function call 或 component 還久」時，action 才值得進 store。

僅列出有可驗證條件的情境( action 放進 store 是合理且必要的工程決策 )，避免架構過度設計。

#### 長期存活的 callbacks（Long-lived Callbacks）

當 callback 的生命週期 超過單次 function 呼叫，並且會在未來反覆觸發、更新 state 時，action 必須放在 store。

##### 範例

- Event listeners: EventOn 更新 state
- WebSocket handlers: 持續監聽伺服器訊息，收到資料後更新 store。
- setInterval / setTimeout: 定期或延遲執行的 callback，更新或輪詢狀態。
- background polling: 持續向伺服器請求資料，並更新 state。

##### 說明

只要 callback 活得比呼叫它的 function 久，就該進 store

- callback 需要持續 `set()` state
- 不能依賴 component render 或外部 function scope

#### setup / cleanup 生命週期配對

任何需要「初始化 + 清理」的行為，都應在同一語意邊界內定義。

##### 範例

- subscribe / unsubscribe
- addEventListener / removeEventListener
- start / stop polling

##### 說明

- action 天然適合作為 lifecycle 邊界
- 避免 setup 與 cleanup 分散在不同模組

只要有 cleanup，就要考慮把 setup/cleanup 綁在 store action

#### Store Action 之間形成流程（Action Orchestration）

當 action 不是單點 setter，而是形成流程或狀態轉移序列。

##### 範例

```ts
startMigration: () => {
  get().refreshStatus();
  get().lockUI();
}
```

##### 說明

行為需要全域唯一性時，store 是最穩定的控制點，若 action 會呼叫其他 action 形成流程，應進 store

- 流程集中，語意清楚
- 避免外部 orchestration 四散

#### Singleton 行為控制

某些行為在整個 application 中 只能同時存在一個實例。

##### 範例

- 任何全局同步任務，通常全局只能跑一次: migration、background sync、session refresh

  具有副作用昂貴、不可重入的特性。例如 migration 若同時執行多次可能造成資料衝突或重複處理。

- Token 管理 / Refresh:  refreshToken() / isRefreshing flag / background refresh task

##### 說明

防止重複，所以獨立管理在 store

- 檢查 `isRunning`

  用於全局控制，同時可以避免多個 component 或 request 重複觸發同一任務。

- 防止重入（re-entrancy）

  重入保護是 Singleton 核心要點，適用於所有全局唯一行為，包括 migration、background sync 及 token refresh。

#### Store 內閉包的私有變數管理（非 State）

有些資料不該進 state（不需要 re-render），但 actions 需要共用。

##### 範例

```ts
let timerId: number | null = null;
let abortController: AbortController | null = null;
```

##### 說明

需要 action 共用、但不該進 state 的資料 → store closure

- 利用 module scope / closure
- 避免污染 state

#### 多入口 + 高副作用成本

同一行為需要從多個入口觸發，且副作用成本高。

##### 範例

- background task
- test
- debug / console

##### 說明

（必要）入口多 + 行為不可隨便重跑，才值得進 store

- 單一行為定義
- 防止流程分歧

#### 跨 component lifecycle 的行為一致性（特定架構）

行為必須獨立於 component 是否存在。

##### 範例

- Member.tsx 與 Purchase.tsx 都需要取得/修改，會員點數/會員等級

##### 說明

行為不能依賴 component lifecycle 時 → 放 store

- 架構 micro-frontend
- 行為 lazy-loaded modules
- component 可能 unmount，但行為需繼續

#### Rollback / Transaction 語意集中 (非必須，但有價值)

將 snapshot → mutate → rollback 放在同一 action 內。

##### 說明

為了防呆與可維護性，可選擇放 store

- 技術上非必須
- 提升可讀性
- 降低被誤用風險
