# HCL Conference Outline

## 主題

AI x Playwright x Domino：打造智慧化自動化測試平台

## 分享主軸

這場分享不是介紹 AI，也不是介紹 Playwright。

真正主軸是：

> 我們如何解決 Domino 專案長期存在的回歸測試問題。

## 核心訊息

公司產品不只是導入測試工具，而是把企業級自動化測試能力內建到 Domino Workflow Platform。

Playwright 是執行者，AI 是分析者，Testing Center 是產品化入口。

## 建議流程：30 到 40 分鐘

### 1. 痛點：5 分鐘

說明 Domino 專案常見情境：

- 系統生命週期長
- 客戶客製多
- 功能修改頻繁
- 回歸測試成本高
- 人工測試耗時
- 測試容易遺漏

要讓聽眾先理解：問題不是「缺一個測試工具」，而是「測試能力沒有成為平台的一部分」。

### 2. Playwright：8 分鐘

介紹 Playwright 在本專案中的角色：

- 自動登入
- 自動建單
- 自動送簽
- 自動簽核
- Screenshot
- Trace
- HTML Report

Demo 一次完整測試流程，讓聽眾看到自動化測試能取代重複人工操作。

### 3. AI 如何加入：10 到 15 分鐘

強調：

```text
Playwright 負責執行。
AI 負責思考。
```

AI 可切入四個方向：

- Test Case Generation
- Script Generation
- Error Analysis
- Regression Recommendation

可以用錯誤分析作為最直覺的展示：

```text
Playwright 原始錯誤：
Timeout 30000ms

AI 分析：
Login 成功，但因權限不足，Submit Button 未出現，因此 Locator 找不到。
```

### 4. 公司產品 Demo：10 分鐘

展示 Testing Center 產品流程：

```text
Testing Center
  -> Run Test
  -> Playwright 執行
  -> AI 分析
  -> 結果回寫平台
```

建議展示內容：

- 流程清單
- Run Test
- 測試狀態
- Pass / Fail 步驟
- Screenshot
- Playwright Report
- AI Summary
- AI 建議

### 5. 未來展望：3 分鐘

可提到：

- Git Commit 後自動測試
- CI/CD 整合
- Release Gate
- AI 自動修正 Locator
- AI Agent 自動維護測試腳本
- 從 Testing Center 擴展為產品標準能力

## Demo 建議

建議準備兩段 Demo：

### 成功案例

展示完整流程測試：

- Login
- 建單
- 送簽
- 簽核
- 結案

結果全部通過，顯示 Testing Center 可產出 report / screenshot / history。

### 失敗案例

刻意讓一個 Locator 或權限條件失敗，展示：

- Playwright 原始錯誤
- Screenshot
- Trace
- AI Summary
- AI 修正建議

失敗案例通常比成功案例更能展現 AI 的價值，因為它把「錯誤訊息」轉成「可理解、可行動的建議」。
