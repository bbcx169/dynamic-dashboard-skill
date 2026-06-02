---
name: dynamic-dashboard
description: 建立與 Google 試算表（Google Sheets）動態數據同步的 Apps Script 儀表板。當說「建立動態儀表板」「設定動態數據儀表板」或「動態儀表板」時載入此技能。
---

# 動態數據儀表板建置與自動化部署技能

當使用者想要針對 Google 試算表（Google Sheets）建立動態同步儀表板時，請遵循以下完整步驟與排錯指南：

## 1. 資料來源評估與讀取 (公開 vs 私人)

在開始前，先評估試算表權限並選擇最優讀取路徑：

### 🔓 途徑 A：試算表為「公開」 (知道連結的任何人均可檢視)
- **優點**：最簡捷，繞過所有 Google 帳號登入與 NotebookLM 快取卡死問題。
- **步驟**：AI 助理在背景直接使用網路請求從以下 URL 直讀並解析 CSV 數據：
  `https://docs.google.com/spreadsheets/d/<試算表ID>/export?format=csv`

### 🔒 途徑 B：試算表為「非公開」 (限特定帳號存取)
- **優點**：高安全性，保護內部隱私數據不外洩。
- **步驟**：透過本機已登入的 `nlm` (NotebookLM CLI) 工具安全地匯入專屬筆記本：
  ```powershell
  # 建立專屬筆記本
  nlm create notebook "專案數據筆記本"
  # 將私人試算表安全地加入該筆記本作為上下文
  nlm add drive <筆記本ID> <試算表文件ID> --type sheets --wait
  ```
  *(注意：若 nlm 指令報錯或卡死，請參考下方常見問題排除。)*

## 2. 產出動態前後端程式碼 (直接以試算表數據與結構生成)
- 依據途徑 A（下載並讀取 CSV）或途徑 B（透過 `nlm content` 獲取內容）直讀該試算表的原始數據與表頭欄位。
- AI 助理無須經過繁瑣的對答，直接依據讀取到的試算表真實資料結構，在本地 `tmp/` 目錄下生成高度匹配且完全動態連動的 3 個核心檔案：
  - `Code.gs`：負責伺服器端 `doGet()` 路由與 Spreadsheet 資料解析（包含數值清洗，去除 %、轉換整數）。
  - `index.html`：基於 Chart.js 提供極致 Vanilla CSS 暗色系視覺效果的前端頁面，支援類別篩選與排序切換功能。
  - `appsscript.json`：設定 Apps Script 專案屬性。



## 3. 連接與強制推送 (clasp)
- 請使用者點選試算表的 `擴充功能` -> `Apps Script`，複製專案設定中的 **指令碼 ID (Script ID)**。
- 本地建立 `.clasp.json`：
  ```json
  {
    "scriptId": "<指令碼 ID>"
  }
  ```
- 執行強制推送（首次推送必須加上 `-f` 以覆蓋預設 placeholder）：
  ```powershell
  npx.cmd clasp push -f
  ```

## 4. 部署網頁應用程式
- 指引使用者在 Apps Script 網頁後台：
  1. 點選 `部署` -> `新增部署` -> 選擇 `網頁應用程式`。
  2. 設定執行身份為「我」，誰可以存取為「所有人」。
  3. 部署並完成 Google 帳號授權，取得網頁應用程式網址。

---

## 🛠️ 排錯指南 (常見問題)

| 問題 | 解法 |
|---|---|
| nlm 指令找不到 (未在 PATH 中) | 將 `C:\Users\<你>\AppData\Roaming\Python\Python314\Scripts` 加入 Windows 環境變數 PATH 中。 |
| NotebookLM 登入快取卡死 / 驗證過期 | 執行 `nlm login profile delete default -y` 刪除舊 Profile，再重新執行 `nlm login` 登入。 |
| GitHub CLI 提示 Token 無效 | 執行 `$env:GITHUB_TOKEN = $null` 清除沙盒環境自動注入的 dummy token，回歸使用本機 Keyring。 |
| clasp push 顯示 Skipping push | 執行 `npx.cmd clasp push -f` 加上 `--force` 參數以強制覆蓋雲端預設程式碼。 |
