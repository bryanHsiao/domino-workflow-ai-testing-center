# AI Testing Center

AI Testing Center 是一個內建於 Domino Workflow Platform 的智慧化自動化測試平台。

這個專案的核心定位不是把 Playwright 接進 Domino，而是把「測試能力」變成 Domino Workflow Platform 的核心能力。

## 專案背景

公司主要開發平台為 HCL Domino，前端採用 Web Application。系統完成開發後，仍需要投入大量人工進行回歸測試。

每次功能修改、版本更新或客戶客製，都需要再次確認：

- 表單是否正常
- 流程是否正常
- 權限是否正常
- 簽核是否正常

人工測試成本高、耗時，也容易遺漏。因此，本專案希望建立一套整合於公司產品內的 Testing Center，讓自動化測試成為產品能力的一部分。

## 專案目標

本專案不是單純導入 Playwright，而是打造：

```text
Domino Workflow Platform
+ Testing Center
+ Backend API
+ Playwright
+ AI Analysis
```

形成一個可執行、可分析、可追蹤、可產品化的測試平台。

## 核心概念

Playwright 負責「執行」：

- Browser Automation
- Screenshot
- Trace
- Video
- HTML Report

AI 負責「思考」：

- 產生 Test Case
- 協助產生 Playwright Script
- 分析測試失敗原因
- 提供修正建議
- 推薦應執行的 Regression Tests

## 第一階段 MVP

第一階段先完成最小可用版本：

- 建立 Testing Center UI
- 串接 Backend API
- 啟動 Playwright
- 執行指定測試
- 回傳測試結果
- 顯示 HTML Report
- 顯示 Screenshot
- 儲存歷史紀錄

AI 功能可於第二階段逐步加入，先以錯誤分析與測試案例生成為優先。

## 文件

- [測試中台 MVP 規格](docs/testing-center-spec.md)
- [本機 Runner 環境設定](docs/local-runner-environment-setup.md)
