# 測試中台 MVP 規格

## 目的

目前系統每執行一次測試就建立一筆 `TestLog`，會讓「測試案例規格」與「每次執行紀錄」混在一起。

MVP 建議調整為：

```text
測試模組（TestModule）：可重用測試腳本積木
測試案例（TestCase）：主文件，定義要跑哪些測試模組
執行紀錄（TestRun）：測試案例的子文件，每跑一次建立一筆紀錄
```

核心原則：

- 測試案例是穩定規格，不放本次變更說明。
- 執行紀錄是每次執行結果，保存執行時間、狀態、測試產物、錯誤與 AI 分析。
- 測試模組是可重用腳本，例如登入、開啟表單、送簽、簽核。
- 既有表單 / 流程設定已經有表單代號、表單名稱、權責單位、流程名稱，因此測試案例只引用表單代號與流程名稱，不重複維護流程主資料。

## 中文化命名原則

中台畫面與文件盡量使用中文；程式、JSON、API 與 Runner 對接仍可保留英文欄位代號。

建議命名方式：

| 中文名稱 | 內部代號 | 說明 |
|---|---|---|
| 測試模組 | `TestModule` | 可重用腳本積木 |
| 測試案例 | `TestCase` | 主文件，定義測試規格 |
| 執行紀錄 | `TestRun` | 子文件，每跑一次建立一筆 |
| 執行批次 | `TestBatch` | 第二階段用，多筆案例共用一次批次脈絡 |
| 測試產物 | `Artifacts` | Report、Screenshot、Trace、Console Log |
| 步驟結果 | `StepResults` | 每個測試模組的執行結果 |

## 資料模型總覽

```text
既有表單 / 流程設定
  - 表單代號
  - 表單名稱
  - 流程名稱
  - 權責單位
  - 權責管理人員
  - 適用單位

測試模組
  - 模組代號
  - 腳本
  - 必填參數

測試案例主文件
  - 表單代號 / 流程名稱
  - 測試目的
  - 測試模組組合順序
  - 預期結果

執行紀錄子文件
  - 每次執行狀態
  - 步驟結果
  - 報告 / 截圖 / Trace / Console Log
  - AI 摘要 / AI 建議
```

## 測試模組（TestModule）

`test1-login.js`、`test2-openform.js` 這類腳本不建議直接視為完整測試案例。它們比較適合設計成可重用的測試模組。

範例：

```text
common.login
form.open
form.create
workflow.submit
workflow.approve
workflow.close
assert.visible
assert.text
```

### 測試模組欄位

| 畫面欄位 | 內部欄位代號 | 說明 |
|---|---|---|
| 文件類型 | `DocType` | 固定為 `TestModule` |
| 模組代號 | `ModuleId` | 唯一代號，例如 `common.login` |
| 模組名稱 | `ModuleName` | 顯示名稱，例如「登入」 |
| 模組狀態 | `ModuleStatus` | `啟用 / 停用 / 淘汰` |
| 模組類型 | `ModuleType` | `登入 / 開表單 / 流程操作 / 驗證 / 自訂` |
| 用途說明 | `Description` | 模組用途 |
| 腳本來源 | `ScriptSourceType` | `附件 / 檔案路徑 / 內嵌文字` |
| 腳本路徑 | `ScriptPath` | 若腳本放檔案系統，記錄路徑 |
| 腳本附件 | `ScriptAttachment` | 若腳本存在 Domino 文件，放 JS 檔附件 |
| 腳本內容 | `ScriptBody` | 若先用文字欄位存腳本，可放 inline script |
| 必填參數 | `RequiredParams` | 必填參數 JSON，例如 `["formCode", "role"]` |
| 預設參數 | `DefaultParams` | 預設參數 JSON |
| 逾時秒數 | `TimeoutSec` | 單一模組 timeout |
| 維護者 | `Owner` | 模組維護者 |
| 版本 | `Version` | 模組版本，可先手動填 |
| 最近修改時間 | `ModifiedAt` | 最近修改時間 |

### 測試模組腳本建議格式

每個模組匯出固定 `run` function，Runner 會傳入同一個 browser context / page。

```js
export async function run(ctx, params) {
  const { page } = ctx;

  // 測試模組邏輯

  return {
    status: "passed",
    message: "login completed"
  };
}
```

重要：不要每個測試模組自己啟動 browser。Runner 應建立同一個 `browser/context/page`，依序把同一個 `ctx` 傳給各模組，才能保留登入狀態。

## 測試案例（TestCase）

測試案例是主文件，定義一個可重複執行的測試規格。它不是每次執行紀錄。

例如：

```text
079009-OPEN-001：可攜式資料載具申請書 - 開啟表單測試
  1. common.login
  2. form.open
```

### 測試案例必要欄位

| 畫面欄位 | 內部欄位代號 | 說明 |
|---|---|---|
| 文件類型 | `DocType` | 固定為 `TestCase` |
| 案例代號 | `CaseId` | 測試案例代號，例如 `079009-OPEN-001` |
| 案例名稱 | `CaseName` | 測試案例名稱 |
| 案例狀態 | `CaseStatus` | `草稿 / 啟用 / 淘汰` |
| 表單代號 | `FormCode` | 引用既有表單代號，例如 `079009` |
| 流程名稱 | `FlowName` | 引用既有流程名稱，例如 `079009_Flow` |
| 測試套件 | `TestSuite` | 例如 `079009 自動化測試` |
| 測試目的 | `Purpose` | 這個案例驗證什麼 |
| 測試角色 | `TestRole` | 測試角色 / 帳號群組，例如 `申請人` |
| 前置條件 | `Precondition` | 測試前必須滿足的條件 |
| 預期結果 | `ExpectedResult` | 測試通過時應該看到的結果 |
| 測試步驟 | `StepsJson` | 測試模組組合順序與參數 |
| 分類標籤 | `Tags` | `smoke / regression / permission / open-form` |
| 優先級 | `Priority` | `P1 / P2 / P3` |
| 預設逾時秒數 | `DefaultTimeoutSec` | 預設 timeout |
| 重試次數 | `RetryCount` | 失敗後是否重試 |
| 最近執行時間 | `LastRunAt` | 最近一次執行時間 |
| 最近執行結果 | `LastRunStatus` | 最近一次執行結果 |
| 最近執行紀錄 | `LastRunUNID` | 最近一次執行紀錄 UNID |
| 建立人 / 建立時間 | `CreatedBy` / `CreatedAt` | 建立資訊 |
| 修改人 / 修改時間 | `ModifiedBy` / `ModifiedAt` | 修改資訊 |

### StepsJson 範例

```json
[
  {
    "order": 1,
    "moduleId": "common.login",
    "moduleName": "登入",
    "params": {
      "role": "applicant"
    },
    "expected": "登入成功並進入首頁"
  },
  {
    "order": 2,
    "moduleId": "form.open",
    "moduleName": "開啟表單",
    "params": {
      "formCode": "079009",
      "formName": "可攜式資料載具申請書"
    },
    "expected": "成功開啟 079009 表單"
  }
]
```

### 測試案例不應放的資料

以下資料不要寫死在測試案例：

- 本次變更說明
- 本次版本更新內容
- 某一次 Bug 修復描述
- 某一次執行錯誤
- 某一次截圖 / Trace / 報告

這些都屬於執行紀錄或未來的執行批次。

## 執行紀錄（TestRun）

執行紀錄是測試案例的 response document。每按一次「執行測試」，就建立一筆執行紀錄。

### 執行紀錄必要欄位

| 畫面欄位 | 內部欄位代號 | 說明 |
|---|---|---|
| 文件類型 | `DocType` | 固定為 `TestRun` |
| 執行代號 | `RunId` | 每次執行唯一 ID，例如 `RUN-20260810-0001` |
| 主文件 UNID | `ParentUNID` | 對應測試案例主文件 UNID |
| 案例代號 | `ParentCaseId` | 對應 `CaseId` |
| 批次代號 | `BatchId` | 同批回歸測試 ID，MVP 可先保留空值 |
| 案例名稱快照 | `CaseNameSnapshot` | 執行當下的案例名稱 |
| 表單代號快照 | `FormCodeSnapshot` | 執行當下的表單代號 |
| 流程名稱快照 | `FlowNameSnapshot` | 執行當下的流程名稱 |
| 測試步驟快照 | `StepsSnapshotJson` | 執行當下的測試步驟 JSON |
| 執行時間 | `ExecutedAt` | 使用者按下執行的時間 |
| 執行者 | `ExecutedBy` | 執行測試的人 |
| 測試環境 | `Environment` | `DEV / UAT / PROD-like` |
| 執行原因 | `RunReason` | `手動測試 / 回歸測試 / Bug 修復驗證 / 上版前驗證` |
| 本次變更說明 | `ChangeContext` | 本次執行的變更說明，選填 |
| 測試狀態 | `TestStatus` | `排隊中 / 執行中 / 通過 / 失敗 / 錯誤` |
| 開始 / 結束時間 | `StartAt` / `EndAt` | 開始與結束時間 |
| 執行耗時 | `DurationMs` | 執行耗時 |
| 步驟結果 | `StepResultsJson` | 每個測試模組的執行結果 |
| 失敗步驟 | `FailedStep` | 失敗發生在哪一步 |
| 錯誤類型 | `ErrorType` | `locator_timeout / assertion_failed / login_failed / env_error / script_error` |
| 原始錯誤訊息 | `ErrorMessage` | Playwright 原始錯誤 |
| 報告路徑 | `ReportPath` | HTML / markdown report 路徑 |
| 截圖路徑 | `ScreenshotPath` | 失敗截圖路徑 |
| Trace 路徑 | `TracePath` | Playwright trace.zip 路徑 |
| Console Log 路徑 | `ConsoleLogPath` | console log 路徑 |
| 原始結果 JSON | `ResultJsonPath` | runner 原始 JSON 結果 |
| AI 分析狀態 | `AIStatus` | `不需分析 / 待分析 / 已完成 / 分析失敗` |
| AI 摘要 | `AISummary` | AI 分析摘要 |
| AI 可能原因 | `AILikelyCause` | 可能原因 |
| AI 建議 | `AISuggestion` | 修正建議 |
| AI 信心程度 | `AIConfidence` | `低 / 中 / 高` |

### StepResultsJson 範例

```json
[
  {
    "order": 1,
    "moduleId": "common.login",
    "moduleName": "登入",
    "status": "passed",
    "durationMs": 1098,
    "message": "登入成功"
  },
  {
    "order": 2,
    "moduleId": "form.open",
    "moduleName": "開啟表單",
    "status": "passed",
    "durationMs": 1257,
    "message": "成功開啟 079009"
  }
]
```

### 為什麼執行紀錄要保存快照

測試案例未來可能會修改步驟或模組參數。若舊的執行紀錄只引用最新測試案例，歷史紀錄會失真。

因此執行紀錄建立時，應保存當下的：

- `CaseNameSnapshot`
- `FormCodeSnapshot`
- `FlowNameSnapshot`
- `StepsSnapshotJson`

這樣未來看歷史紀錄時，可以知道當時到底跑了哪些步驟。

## 執行批次（TestBatch）：第二階段

MVP 可以先不做完整執行批次表單，但執行紀錄要先保留 `BatchId`。

未來批次回歸測試流程：

```text
建立執行批次
  -> 選擇測試套件或多筆測試案例
  -> 填一次測試環境 / 執行原因 / 本次變更說明
  -> 產生多筆執行紀錄
  -> 每筆執行紀錄共用同一個 BatchId
```

這樣本次變更說明只填一次，不需要每個測試案例都填。

## Runner 執行合約

使用者在測試中台按下「執行測試」後，後端應產生一份測試執行 JSON 給 Runner。

### 輸入：測試執行 JSON

```json
{
  "runId": "RUN-20260810-0001",
  "caseId": "079009-OPEN-001",
  "caseName": "可攜式資料載具申請書 - 開啟表單測試",
  "environment": "UAT",
  "outputDir": "D:/react/Eform2/079009/results/20260810191222",
  "steps": [
    {
      "order": 1,
      "moduleId": "common.login",
      "scriptPath": "test-modules/common.login.js",
      "params": {
        "role": "applicant"
      }
    },
    {
      "order": 2,
      "moduleId": "form.open",
      "scriptPath": "test-modules/form.open.js",
      "params": {
        "formCode": "079009"
      }
    }
  ]
}
```

### Runner 命令

```text
node runner.js --case-file <case-execution.json> --run-id <run-id> --output-dir <output-dir>
```

若目前是 Python runner，也維持同樣概念：

```text
python runner.py --case-file <case-execution.json> --run-id <run-id> --output-dir <output-dir>
```

### 輸出：result.json

Runner 完成後要輸出固定 `result.json`，不要只靠 console 文字解析。

```json
{
  "runId": "RUN-20260810-0001",
  "status": "passed",
  "startedAt": "2026-08-10T19:12:22+08:00",
  "finishedAt": "2026-08-10T19:12:25+08:00",
  "durationMs": 2350,
  "steps": [
    {
      "order": 1,
      "moduleId": "common.login",
      "moduleName": "登入",
      "status": "passed",
      "durationMs": 1098
    },
    {
      "order": 2,
      "moduleId": "form.open",
      "moduleName": "開啟表單",
      "status": "passed",
      "durationMs": 1257
    }
  ],
  "artifacts": {
    "reportPath": "D:/react/Eform2/079009/results/20260810191222/report.md",
    "screenshotPath": "",
    "tracePath": "D:/react/Eform2/079009/results/20260810191222/trace.zip",
    "consoleLogPath": "D:/react/Eform2/079009/results/20260810191222/console.log"
  },
  "error": null
}
```

失敗時：

```json
{
  "runId": "RUN-20260810-0002",
  "status": "failed",
  "failedStep": "form.open",
  "error": {
    "type": "locator_timeout",
    "message": "Timeout 30000ms waiting for locator"
  },
  "artifacts": {
    "reportPath": ".../report.md",
    "screenshotPath": ".../failure.png",
    "tracePath": ".../trace.zip",
    "consoleLogPath": ".../console.log"
  }
}
```

## 導覽與 View 設計

建議左側導覽先設計成以下結構。

```text
自動化測試
  儀表板

  測試案例
    依表單
    依測試套件
    啟用中
    草稿
    已淘汰

  測試模組
    所有模組
    依類型
    已停用

  執行紀錄
    最新執行紀錄
    失敗紀錄
    依測試案例
    依執行批次

  AI 分析
    待分析
    分析歷史

  設定
    測試環境
    測試帳號
    Runner 設定
```

### 儀表板

用途：中台首頁，快速看整體狀態。

建議顯示：

- 今日執行次數
- 最近一次執行結果
- 最近失敗案例
- 啟用中測試案例數量
- 失敗紀錄數量
- 常用「執行測試」入口

### 測試案例 / 依表單

資料來源：`DocType = TestCase`

欄位：

```text
表單代號
流程名稱
案例代號
案例名稱
測試套件
案例狀態
最近執行結果
最近執行時間
```

### 測試案例 / 依測試套件

資料來源：`DocType = TestCase`

第一欄以 `TestSuite` 分類，方便一次看某套回歸測試有哪些案例。

欄位：

```text
測試套件
案例代號
案例名稱
表單代號
優先級
最近執行結果
最近執行時間
```

### 測試模組 / 所有模組

資料來源：`DocType = TestModule`

欄位：

```text
模組代號
模組名稱
模組類型
模組狀態
腳本來源
逾時秒數
最近修改時間
```

### 執行紀錄 / 最新執行紀錄

資料來源：`DocType = TestRun`

取代目前平鋪的 `TestLogView`。

欄位：

```text
執行時間
案例代號
案例名稱
測試狀態
執行耗時
報告路徑
AI 摘要
```

### 執行紀錄 / 依測試案例

資料來源：`DocType = TestRun`

第一欄以 `ParentCaseId` 或 `CaseNameSnapshot` 分類，讓使用者可以打開一個 TestCase 後看歷史紀錄。

欄位：

```text
案例代號
執行時間
測試狀態
執行原因
執行耗時
失敗步驟
報告路徑
```

### 執行紀錄 / 失敗紀錄

資料來源：`DocType = TestRun AND TestStatus in ("FAILED", "ERROR")`

欄位：

```text
執行時間
案例代號
案例名稱
失敗步驟
錯誤類型
原始錯誤訊息
AI 分析狀態
AI 摘要
```

### AI 分析 / 待分析

資料來源：

```text
DocType = TestRun
TestStatus in ("FAILED", "ERROR")
AIStatus in ("PENDING", "FAILED")
```

用途：第二階段 AI 分析工作佇列。

## 設定資料設計

設定區用來保存「測試執行前需要參考的外部條件」，不是測試案例本身。

可以這樣理解：

```text
測試案例：要測什麼
測試步驟：怎麼測
測試環境：去哪裡測
測試帳號：用誰測
Runner 設定：用哪個程式、在哪裡跑、產物放哪裡
```

核心原則：

- 測試案例不要寫死網址、帳密、執行路徑。
- `StepsJson` 不存密碼、API key、token。
- 不同環境、不同帳號、不同 Runner 的差異，應透過設定帶入。
- AI 分析時可引用設定的顯示名稱與角色，但不得取得密碼明文。

### 測試環境

用途：定義 DEV、UAT、正式前測試環境等連線資訊。執行測試時，使用者選擇一個測試環境，Runner 依此取得系統網址、登入網址與測試產物儲存位置。

建議欄位：

| 畫面欄位 | 內部欄位代號 | 說明 |
|---|---|---|
| 文件類型 | `DocType` | 固定為 `TestEnvironment` |
| 環境代號 | `EnvironmentId` | 例如 `DEV`、`UAT`、`PRODLIKE` |
| 環境名稱 | `EnvironmentName` | 例如 `UAT 測試環境` |
| 環境類型 | `EnvironmentType` | `DEV / UAT / PROD-like` |
| 系統網址 | `BaseUrl` | 測試系統首頁或根網址 |
| 登入網址 | `LoginUrl` | 登入頁網址 |
| Domino Server | `DominoServer` | 例如 `Server01/Org`，選填 |
| 資料庫路徑 | `DatabasePath` | 例如 `Eform2/eform.nsf`，選填 |
| 報告根目錄 | `ReportRootPath` | 測試報告在檔案系統的根目錄 |
| 報告 URL 前綴 | `ReportUrlPrefix` | UI 開啟報告時使用的 URL 前綴 |
| 是否啟用 | `IsActive` | `是 / 否` |
| 備註 | `Remark` | 例如需 VPN、內網限定 |

範例：

```text
環境代號：UAT
環境名稱：UAT 測試環境
系統網址：https://uat.example.com
登入網址：https://uat.example.com/login
報告根目錄：D:\react\Eform2\reports
報告 URL 前綴：http://server/reports
是否啟用：是
```

MVP 最小欄位：

```text
環境代號
環境名稱
系統網址
登入網址
報告根目錄
是否啟用
```

### 測試帳號

用途：定義測試角色對應的帳號。測試案例與 `StepsJson` 只引用帳號代號，不直接保存帳密。

登入步驟建議這樣寫：

```json
{
  "order": 1,
  "moduleId": "common.login",
  "params": {
    "accountKey": "engineer"
  },
  "expected": "工程師測試帳號登入成功"
}
```

Runner 執行時再用 `accountKey = engineer` 去「測試帳號」設定找登入帳號與密碼來源。

建議欄位：

| 畫面欄位 | 內部欄位代號 | 說明 |
|---|---|---|
| 文件類型 | `DocType` | 固定為 `TestAccount` |
| 帳號代號 | `AccountKey` | 例如 `engineer`、`applicant`、`approver` |
| 顯示名稱 | `AccountName` | 例如 `工程師測試帳號` |
| 適用環境 | `EnvironmentId` | 對應測試環境代號 |
| 測試角色 | `TestRole` | 例如 `工程師 / 申請人 / 主管 / 系統管理員` |
| 登入帳號 | `Username` | 實際登入帳號 |
| 密碼來源 | `PasswordSource` | `ENV / Secret Manager / Domino 加密欄位` |
| 密碼 Key | `PasswordKey` | 例如 `TEST_PASS_ENGINEER` |
| 是否啟用 | `IsActive` | `是 / 否` |
| 備註 | `Remark` | 例如可開啟哪些表單或具備哪些權限 |

範例：

```text
帳號代號：engineer
顯示名稱：工程師測試帳號
適用環境：UAT
測試角色：工程師
登入帳號：test_engineer01
密碼來源：ENV
密碼 Key：TEST_PASS_ENGINEER
是否啟用：是
```

MVP 最小欄位：

```text
帳號代號
測試角色
登入帳號
密碼來源
密碼 Key
適用環境
是否啟用
```

注意事項：

- 不要把密碼放在 `StepsJson`。
- 不要把密碼放在 `TestRun.StepsSnapshotJson`。
- 不要把密碼放進 AI Analysis Bundle。
- 若暫時使用 Domino 欄位保存密碼，需使用加密欄位或公司既有安全機制。

### Runner 設定

用途：定義測試程式怎麼被啟動、在哪個工作目錄執行、測試模組從哪裡讀取、產物要保存到哪裡。

建議欄位：

| 畫面欄位 | 內部欄位代號 | 說明 |
|---|---|---|
| 文件類型 | `DocType` | 固定為 `RunnerConfig` |
| Runner 代號 | `RunnerId` | 例如 `local-node-runner` |
| Runner 名稱 | `RunnerName` | 例如 `本機 Node.js Runner` |
| Runner 類型 | `RunnerType` | `Node.js / Python` |
| 執行命令 | `Command` | 例如 `node runner.js` |
| 工作目錄 | `WorkingDirectory` | Runner 執行時的工作目錄 |
| 測試模組目錄 | `ModuleRootPath` | 測試模組檔案所在目錄 |
| 輸出根目錄 | `OutputRootPath` | 測試結果與產物輸出根目錄 |
| 瀏覽器 | `BrowserName` | `chromium / firefox / webkit` |
| Headless | `Headless` | `是 / 否` |
| 預設逾時秒數 | `DefaultTimeoutSec` | 預設 timeout |
| 重試次數 | `RetryCount` | 預設 retry |
| Trace 模式 | `TraceMode` | `失敗保留 / 每次保留 / 不保留` |
| 截圖模式 | `ScreenshotMode` | `失敗截圖 / 每步截圖 / 不截圖` |
| 錄影模式 | `VideoMode` | `開 / 關` |
| 產物保留天數 | `RetentionDays` | 測試產物保留多久 |
| 是否啟用 | `IsActive` | `是 / 否` |

範例：

```text
Runner 代號：local-node-runner
Runner 名稱：本機 Node.js Runner
Runner 類型：Node.js
執行命令：node runner.js
工作目錄：D:\react\Eform2
測試模組目錄：D:\react\Eform2\test-modules
輸出根目錄：D:\react\Eform2\results
瀏覽器：chromium
Headless：是
Trace 模式：失敗保留
截圖模式：失敗截圖
是否啟用：是
```

MVP 最小欄位：

```text
Runner 代號
執行命令
工作目錄
測試模組目錄
輸出根目錄
Headless
Trace 模式
截圖模式
是否啟用
```

### 設定資料與測試執行的關係

執行單一測試案例時：

```text
使用者選擇 TestCase
  -> 選擇測試環境
  -> TestCase.StepsJson 指定 accountKey / formCode 等參數
  -> Runner 讀取測試環境、測試帳號、Runner 設定
  -> 執行測試模組
  -> 產生 TestRun
```

執行測試集合時：

```text
使用者選擇 TestSuite
  -> 選擇測試環境
  -> 建立 TestBatch
  -> 針對集合內每個 TestCase 建立 TestRun
  -> 每筆 TestRun 共用同一個 BatchId
```

給工程師的重點：

> 設定區是讓 TestCase 不要寫死環境、帳密與執行路徑；TestCase 只描述測試規格，實際去哪跑、用誰跑、怎麼跑，都從設定帶入。

## 表單按鈕建議

### 測試案例主文件按鈕

```text
執行測試
複製案例
停用案例
查看最近執行
查看執行歷史
```

### 執行紀錄子文件按鈕

```text
開啟報告
開啟截圖
開啟 Trace
執行 AI 分析
重新執行此案例
```

### 測試模組文件按鈕

```text
驗證腳本
複製模組
停用模組
```

## MVP 實作順序

### 步驟 1：先拆資料模型

- 建立「測試模組」表單
- 建立「測試案例」表單
- 建立「執行紀錄」response 表單
- 現有 `TestLogView` 改成「最新執行紀錄」

### 步驟 2：把現有腳本改成測試模組

- `test1-login.js` -> `common.login`
- `test2-openform.js` -> `form.open`
- 每個測試模組支援 `run(ctx, params)`
- Runner 共用同一個 browser/page 執行測試模組

### 步驟 3：測試案例組合測試模組

- `TestCase.StepsJson` 定義要跑哪些測試模組
- 第一版先做 `common.login + form.open`
- 按「執行測試」後建立執行紀錄

### 步驟 4：Runner 輸出固定 result.json

- 不只產生 `report.md`
- 必須產生 `result.json`
- Domino 後端讀 `result.json` 回寫執行紀錄

### 步驟 5：補測試產物與 AI 欄位

- 保存報告 / 截圖 / Trace / Console Log
- 失敗時先保留 AI 欄位
- 第二階段再串 AI 分析

## MVP 驗收標準

- 可以新增一個測試模組：`common.login`
- 可以新增一個測試模組：`form.open`
- 可以新增一個測試案例：`079009-OPEN-001`
- 測試案例可以設定步驟：`common.login` -> `form.open`
- 按下「執行測試」會建立一筆執行紀錄子文件
- Runner 會依序執行兩個測試模組
- 執行紀錄會寫回 `PASSED / FAILED`
- 執行紀錄會保存每一步驟結果
- 執行紀錄可開啟報告
- 測試案例主文件會更新 `LastRunAt` 與 `LastRunStatus`
