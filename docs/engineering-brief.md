# Engineering Brief: AI Testing Center MVP

## 交辦目標

第一階段請先完成一個可展示、可運作的 Testing Center MVP 閉環。

本階段不是要完成完整 AI 平台，也不是單純寫幾支 Playwright script，而是要證明：

```text
使用者可以在 Domino Workflow Platform 內觸發測試，
後端可以啟動 Playwright 執行指定流程，
測試結果可以回寫 Testing Center 並保留歷史紀錄。
```

## 核心定位

這個專案的定位是：

> 不是把 Playwright 接進 Domino，而是把「測試能力」變成 Domino Workflow Platform 的核心能力。

Playwright 是執行測試的工具，Testing Center 是產品化入口。AI 功能第二階段再加入，第一階段先預留資料欄位與畫面位置。

## MVP 完成畫面

MVP 完成後，Demo 應該可以跑出以下流程：

```text
進入 Testing Center
  -> 看到可測試流程，例如 F085001
  -> 按下 Run Test
  -> 畫面顯示 Running
  -> Backend API 建立 test run
  -> Playwright 執行瀏覽器自動化測試
  -> 測試完成
  -> 畫面顯示 Pass / Fail
  -> 顯示每個 step 的結果
  -> 可開啟 Screenshot
  -> 可開啟 HTML Report
  -> 歷史紀錄保留本次執行結果
```

## 建議 MVP 範圍

### 1. Testing Center UI

建立一個 Testing Center 頁面，第一版不需要複雜功能，但要可以展示產品概念。

至少包含：

- 流程清單
- 單一流程測試入口
- Run Test 按鈕
- 測試執行狀態：Pending / Running / Passed / Failed
- 測試步驟結果
- 最新一次測試摘要
- 測試歷史紀錄
- Screenshot 連結或預覽
- Playwright HTML Report 連結
- AI Summary 欄位，第一版可先顯示空值或固定範例

建議第一版先支援一個 demo flow：

```text
F085001
```

### 2. Backend API

建立一組簡單 API，負責讓 UI 觸發測試與查詢結果。

建議 API：

```text
GET  /api/flows
POST /api/test-runs
GET  /api/test-runs
GET  /api/test-runs/:id
GET  /api/test-runs/:id/artifacts
```

基本責任：

- 回傳可測試流程清單
- 建立 test run
- 啟動 Playwright
- 查詢 test run 狀態
- 查詢 test run 結果
- 回傳 report / screenshot / trace 路徑

### 3. Playwright Runner

建立可由 Backend API 呼叫的 Playwright 執行流程。

至少支援：

- 執行指定 test id
- 產生 Screenshot
- 產生 Trace
- 產生 HTML Report
- 回傳 pass / fail
- 回傳錯誤訊息
- 回傳每個測試步驟狀態

Demo 測試步驟可先定義為：

```text
Login
建單
送簽
簽核
結案
```

### 4. Test Result Storage

第一版儲存方式可以先用 JSON 或 SQLite，不需要一開始就接正式資料庫。

建議 test run 資料欄位：

```text
id
flowId
testName
status
startedAt
finishedAt
durationMs
steps
errorMessage
reportPath
screenshotPath
tracePath
aiSummary
aiSuggestion
createdBy
```

`aiSummary` 與 `aiSuggestion` 第一版可以先保留欄位，不需要真的串 AI。

### 5. Artifacts

Playwright 產生的檔案要能被 UI 開啟或下載。

至少保存：

- HTML Report
- Screenshot
- Trace

若時間足夠，可加上：

- Video
- Console Log

## 第一階段不做的事

為了避免 MVP 發散，第一階段先不做：

- 完整 AI 分析
- AI 自動產生 Playwright script
- Regression Recommendation
- CI/CD 整合
- Git commit 後自動測試
- 多流程完整測試管理
- 複雜權限管理
- 正式版報表設計

這些都可以放到第二階段。

## 建議技術選型

若沒有既有限制，建議 MVP 先用：

```text
Node.js
Express
Playwright
JSON 或 SQLite
簡單 Web UI
```

原因是開發速度快，也容易展示：

- API 啟動 Playwright
- 測試產物路徑管理
- HTML Report 直接打開
- Screenshot 直接顯示

若要與現有 Domino Workflow Platform 更緊密整合，可以先把 API contract 做清楚，後續再決定是否改接 Domino 後端或公司既有服務。

## 工程師需要先確認的問題

請先確認以下資訊，避免 Playwright 測試寫完後無法穩定執行。

### 測試環境

- Domino 測試環境網址是什麼？
- 測試環境是否可以由 Playwright 執行機器連線？
- 是否需要 VPN 或內網權限？
- 測試資料是否可以重複建立？

### 帳號與權限

- 測試帳號有哪些？
- 是否需要不同角色帳號，例如申請人、主管、簽核人？
- 測試帳號密碼要如何管理？
- 是否需要支援 OTP、SSO 或特殊登入流程？

### 測試資料

- Demo flow 要使用哪一個流程？
- 建議是否先用 `F085001`？
- 測試前是否需要初始化資料？
- 測試後是否需要清除資料？
- 如果不能清除資料，如何避免下一次測試衝突？

### Playwright 執行環境

- 測試要跑在開發機、測試機，還是 server？
- 是否允許 headless mode？
- 是否需要保留 video？
- HTML Report 要存在哪個位置？
- Screenshot / Trace 要保留多久？

### 系統整合

- 第一版 Testing Center UI 是獨立頁面，還是直接嵌入現有 Domino Workflow Platform？
- Backend API 要獨立服務，還是接現有後端？
- 測試歷史紀錄未來要不要寫回 Domino 資料庫？

## 建議開發順序

### Step 1: Local Playwright Prototype

- 建立 Playwright 專案
- 寫一支 demo test
- 可手動執行
- 可產生 screenshot / trace / HTML report

### Step 2: Backend API Prototype

- 建立 API 專案
- API 可建立 test run
- API 可啟動 Playwright
- API 可保存測試結果

### Step 3: Testing Center UI Prototype

- 建立流程列表
- 建立 Run Test 按鈕
- 顯示 Running / Passed / Failed
- 顯示測試歷史紀錄
- 顯示 report / screenshot

### Step 4: Demo Stabilization

- 準備一個穩定成功案例
- 準備一個可控失敗案例
- 確認 report / screenshot / trace 都能開啟
- 確認連續執行不會互相污染資料

### Step 5: AI Placeholder

- UI 預留 AI Summary / AI Suggestion 區塊
- test run model 預留 AI 欄位
- 先用固定範例資料展示未來方向

## 驗收標準

MVP 驗收時至少要能做到：

- 使用者可以在 Testing Center 看到 `F085001`
- 使用者可以按下 Run Test
- UI 可以顯示測試正在執行
- Backend 可以啟動 Playwright
- Playwright 可以完成一支 demo test
- 測試結果可以回寫 UI
- UI 可以顯示 pass / fail
- UI 可以顯示每個測試步驟結果
- UI 可以開啟 Screenshot
- UI 可以開啟 HTML Report
- 系統可以保留歷史紀錄

## 建議交付物

工程師第一階段建議交付：

- 可執行的本地專案
- Testing Center MVP UI
- Backend API
- Playwright demo test
- 測試結果儲存檔或資料庫
- Screenshot / Report / Trace artifacts
- README 執行說明
- Demo 成功案例
- Demo 失敗案例

## 給工程師的簡短版說明

第一階段請先完成 Testing Center MVP 閉環：在 UI 選擇一個流程並按 Run Test，後端啟動 Playwright 執行 demo test，完成後把 pass/fail、step result、screenshot、HTML report 與歷史紀錄回寫到 UI。AI 功能先預留欄位與畫面區塊，不需要第一版實作。
