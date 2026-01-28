---
title: Test Tool
date: 2025-12-22T08:00:00+00:00
menu:
  sidebar:
    name: Test Tool
    identifier: Test
    parent: Tool
    weight: 30
tags:
- Test
- Tool

---

# Test Tool

Listed Tool

- StressTesting 壓力測試: JMeter、Nbomber
- End-to-End Test E2E 測試: Automa

## StressTesting

### JMeter

//Todo

### Nbomber

//Todo

## End-to-End Test

### Automa

//Todo



## 補充

### All Types of Test 

文章提及工具: 壓力測試、E2E 測試

#### 功能性測試

- 單元測試 (Unit Test) - 測試最小可測試單元
- 整合測試 (Integration Test) - 測試模組間的介面與互動
- 系統測試 (System Test) - 測試完整系統功能
- 驗收測試 (Acceptance Test) - 驗證系統是否符合需求
- 冒煙測試 (Smoke Test) - 快速驗證主要功能是否正常
- 回歸測試 (Regression Test) - 確保新改動不影響既有功能

#### 非功能性測試

- 效能測試 (Performance Test) - 測試系統效能表現
  - 負載測試 (Load Test) - 測試預期負載下的表現
  - 🟢 壓力測試 (Stress Test) - 測試極限負載: `JMeter`、`Nbomber`
  - 尖峰測試 (Spike Test) - 測試突發流量
  - 耐久測試 (Endurance/Soak Test) - 測試長時間運行穩定性
- 安全測試 (Security Test) - 測試系統安全性與漏洞
- 相容性測試 (Compatibility Test) - 測試不同環境、瀏覽器、裝置的相容性
- 可用性測試 (Usability Test) - 測試使用者體驗

#### UI 相關測試

- 🟢 E2E 測試 (End-to-End Test) - 模擬使用者完整操作流程: `Automa`
- UI 測試 - 測試介面元件與互動
- 視覺回歸測試 (Visual Regression Test) - 比對畫面截圖差異

#### 其他測試類型

- 探索性測試 (Exploratory Test) - 手動探索系統行為
- 混沌工程測試 (Chaos Engineering) - 主動注入故障測試韌性
- A/B 測試 - 比較不同版本效果
- 災難復原測試 (Disaster Recovery Test) - 測試系統復原能力