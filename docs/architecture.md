# Architecture

## 整體流程

```text
使用者
  |
  v
Workflow Platform
  |
  v
Testing Center
  |
  +-- Run Test
  |
  v
Backend API
  |
  v
Playwright
  |
  +-- Browser Automation
  +-- Screenshot
  +-- Trace
  +-- Video
  +-- HTML Report
  |
  v
AI Analysis
  |
  +-- 分析錯誤原因
  +-- 提供修正建議
  +-- 產生摘要
  |
  v
回寫 Testing Center
```

## 主要模組

### Workflow Platform

公司現有 Domino Workflow Platform。使用者在產品內操作 Testing Center，而不是跳到外部測試工具。

### Testing Center

Testing Center 是使用者與測試能力互動的入口。它負責：

- 顯示可測試流程
- 啟動指定測試
- 顯示測試狀態
- 顯示執行結果
- 顯示報表、截圖、Trace
- 保存歷史紀錄
- 顯示 AI 分析摘要

### Backend API

Backend API 是 Domino Workflow Platform 與 Playwright 執行環境之間的橋接層。它負責：

- 接收測試執行請求
- 建立測試任務
- 啟動 Playwright
- 收集測試結果
- 儲存測試產物
- 回傳測試狀態
- 將 AI 分析結果寫回 Testing Center

### Playwright

Playwright 負責實際執行瀏覽器自動化測試。它負責：

- 自動登入
- 建單
- 送簽
- 簽核
- 驗證畫面與資料
- 擷取 Screenshot
- 產生 Trace
- 產生 Video
- 產生 HTML Report

### AI Analysis

AI 不取代 Playwright，而是基於 Playwright 的執行結果進行分析。它可以使用：

- Screenshot
- Trace
- Console Log
- Error Message
- Test Metadata
- Workflow Metadata

AI 的輸出包含：

- 錯誤原因摘要
- 可能的根因
- 修正建議
- 受影響流程推測
- Regression Test 推薦

## 測試結果範例

```text
F085001
----------------
[ Run Test ]

執行結果

OK Login
OK 建單
OK 送簽
OK 簽核
FAIL 結案

錯誤分析：
Locator 找不到 Submit Button。

AI 建議：
Button ID 可能已改名，建議更新 Locator 或檢查目前使用者是否具備結案權限。
```
