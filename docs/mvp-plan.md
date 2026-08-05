# MVP Plan

## MVP 目標

第一階段目標是完成一個可展示、可運作的 Testing Center 閉環：

```text
Testing Center UI
  -> Backend API
  -> Playwright Test Runner
  -> Test Result
  -> Report / Screenshot
  -> History
```

AI 功能可先保留介面與資料結構，第二階段再逐步加入。

## MVP 範圍

### 1. Testing Center UI

建立 Testing Center 頁面，至少包含：

- 流程清單
- 單一流程測試入口
- Run Test 按鈕
- 測試執行狀態
- 測試結果列表
- Report 連結
- Screenshot 顯示
- 歷史紀錄

### 2. Backend API

建立測試執行 API，至少包含：

- 建立測試任務
- 查詢測試任務狀態
- 查詢測試結果
- 查詢測試歷史
- 取得 Report / Screenshot 路徑

### 3. Playwright Runner

建立可由 Backend API 啟動的 Playwright 測試執行流程，至少支援：

- 執行指定 test id
- 產生 HTML Report
- 產生 Screenshot
- 保存 Trace
- 回傳 pass / fail 狀態
- 回傳錯誤訊息

### 4. Result Storage

建立測試紀錄資料模型，至少保存：

- Test Run ID
- Flow ID
- Test Name
- Status
- Started At
- Finished At
- Duration
- Error Message
- Report Path
- Screenshot Path
- Trace Path

### 5. Demo Test Case

先選一個代表性流程作為 MVP Demo，例如：

```text
F085001
```

測試步驟可包含：

- Login
- 建單
- 送簽
- 簽核
- 結案

## 第二階段 AI 功能

AI 功能建議從低風險、高展示價值的場景開始：

### 1. Error Analysis

Playwright Fail 後，將以下資訊交給 AI 分析：

- Error Message
- Screenshot
- Trace Summary
- Console Log
- Test Step
- Flow Metadata

AI 回傳：

- 錯誤摘要
- 可能原因
- 修正建議
- 是否可能為權限問題
- 是否可能為 Locator 失效

### 2. Test Case Generation

根據表單、流程或規格文件產生：

- Test Case
- 測試步驟
- 驗證項目
- 測試資料需求

### 3. Script Generation

AI 協助產生或更新 Playwright Script，降低測試開發成本。

### 4. Regression Recommendation

當某個流程或表單被修改時，AI 推薦應執行哪些測試，避免全部重跑。

## 建議里程碑

### Milestone 1: Local Prototype

- 建立專案骨架
- 建立 Playwright demo test
- 可手動執行測試
- 可產生 report / screenshot

### Milestone 2: Backend Integration

- 建立 API
- API 可啟動 Playwright
- API 可讀取測試結果
- 測試紀錄可保存

### Milestone 3: Testing Center UI

- 建立流程測試頁
- UI 可觸發測試
- UI 可顯示狀態與結果
- UI 可開啟 report / screenshot

### Milestone 4: AI Analysis Prototype

- 將失敗資訊送入 AI
- 顯示 AI Summary
- 顯示 AI 建議

### Milestone 5: Conference Demo

- 準備穩定 Demo 流程
- 準備成功與失敗案例
- 準備 AI 分析展示
- 準備簡報敘事
