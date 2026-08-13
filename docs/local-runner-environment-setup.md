# 本機 Runner 環境設定

## 背景

Testing Center 的正式架構建議以「中台觸發、Runner 統一執行」為主，讓測試環境、測試產物與執行紀錄集中管理。

但在導入初期，也可能會遇到另一種做法：每位測試或開發人員在自己的電腦安裝 Node.js、Playwright 與瀏覽器，透過本機 Runner 執行測試。這種模式可以降低初期伺服器建置成本，但要特別處理環境一致性。

本文件記錄「如果採用本機 Runner，如何讓每台電腦的測試環境盡量一致」。

## 適用情境

本機 Runner 適合：

- POC 或 MVP 初期驗證
- 工程師需要開啟 headed browser 除錯
- 測試案例尚在開發、需要快速調整腳本
- 中央 Runner Server 尚未建置完成

本機 Runner 不適合：

- 多人長期正式回歸測試
- 需要穩定產生正式測試報告
- 需要統一控管測試環境與測試產物
- 需要作為 release gate 或 CI/CD 的依據

因此建議定位為：

```text
本機 Runner：開發、除錯、POC
中央 Runner：正式回歸測試、報告保存、CI/CD
```

## 前提：確認公司網路是否允許下載 Playwright 瀏覽器

`npx playwright install` 會連到 Microsoft CDN 下載瀏覽器 binary。Chromium 約 100-300MB，實際大小會依版本而異。

這是本機 Runner 能否順利導入的第一個阻塞點：

- 若公司網路允許下載，可直接採用下方設定流程。
- 若公司網路或 proxy 擋住下載，需要改由 IT 提供內部 mirror 或標準安裝包。

這件事要先確認，否則每位使用者自行安裝時會很容易失敗。

## Tier 1：先做低成本一致化

### 1. 鎖定版本

Playwright 的 npm 套件版本與瀏覽器 binary 版本需要配對，建議：

- `package.json` 的 `@playwright/test` 不使用 `^`，直接鎖定固定版本。
- 使用 `package-lock.json`。
- 規定安裝時使用 `npm ci`，不要使用 `npm install`。
- 加入 `.nvmrc` 或其他 Node.js 版本管理設定，讓團隊 Node.js 版本一致。

範例：

```text
@playwright/test: 1.54.2
Node.js: 22
```

實際版本依專案當下驗證成功的版本為準。

### 2. 只安裝需要的瀏覽器

如果專案只使用 Chromium，就不要安裝 Firefox 與 WebKit：

```bash
npx playwright install chromium --with-deps
```

不要直接執行不帶參數的：

```bash
npx playwright install
```

否則會下載多個瀏覽器，增加安裝時間與網路負擔。

### 3. 建立一鍵設定腳本

不要只靠文件要求每個人手動設定。建議提供一支 `scripts/setup-env.ps1`：

```powershell
# scripts/setup-env.ps1（規劃）
node -v
npm ci
npx playwright install chromium --with-deps

if (-not (Test-Path .env)) {
    Copy-Item .env.example .env
    Write-Host "請填寫 .env 內的測試帳號設定"
}

npm run test:smoke
```

最後一步 `npm run test:smoke` 很重要，用來確認：

- Node.js 與 npm 套件安裝正確
- Playwright 瀏覽器可啟動
- 測試網站可連線
- 基本登入或開頁流程可執行

這樣可以在正式測試前先排除環境問題。

## Tier 2：需要更高一致性時再加入 Docker

如果團隊變大，或開始出現「同一支測試在不同電腦結果不一致」的問題，可以考慮加入 Docker。

Playwright 官方提供 container image，可把 Node.js、Playwright、瀏覽器與系統相依套件固定在同一個映像檔中。

建議用法：

```text
日常開發 / 除錯：使用本機 Runner，方便開 headed browser 看畫面
標準驗證 / CI：使用 Docker Runner，確保環境一致
```

不建議一開始就強制全部使用 Docker，因為 Domino Web / XPages 類型的問題常需要實際看瀏覽器畫面，例如 partial refresh、iframe dialog、editor 元件等。若完全在 container 內除錯，會增加額外門檻。

## 建議導入順序

```text
1. 確認公司網路能否下載 Playwright 瀏覽器
2. 鎖定 Node.js、Playwright 與 package-lock.json
3. 建立 setup-env.ps1 一鍵設定腳本
4. 建立 smoke test 作為本機環境自我檢查
5. 等正式回歸測試需要穩定化後，再建中央 Runner 或 Docker Runner
```

## 結論

本機 Runner 可以作為 MVP 初期與工程師開發測試腳本時的過渡方案，但不應視為長期正式架構。

Testing Center 若要成為產品能力，正式測試仍建議集中到中央 Runner，才能穩定保存測試紀錄、Report、Screenshot、Trace、Console Log 與 AI 分析結果。
