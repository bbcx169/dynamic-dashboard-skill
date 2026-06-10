---
name: dynamic-dashboard
description: 建立與 Google 試算表動態同步的 Apps Script 儀表板。當提到「建立動態儀表板」時載入。
---

# 動態數據儀表板建置技能

## 1. 數據存取途徑
* **公開試算表**：背景直讀並解析 CSV：`https://docs.google.com/spreadsheets/d/<ID>/export?format=csv`
* **非公開試算表**：使用 `nlm` (NotebookLM CLI) 安全讀取。

## 2. 6 大核心項目互動規劃 (生成前暫停並提問)
列出基於資料特徵的具體建議（每項 3 個選項），請用戶確認：
1. **KPI 指標與邏輯** (KPI Metrics)
2. **圖表類型與 X/Y 軸** (Chart Selection)
3. **篩選與分類維度** (Filters & Dimensions)
4. **清洗與排除規則** (Data Cleaning: 如排除總計、空值設 0)
5. **配色與排版** (Theme & Layout)
6. **決策目標** (Business Goal)

## 3. 生成前後端程式碼 (於 `tmp/` 目錄)
* `Code.gs`：`doGet()` 路由及資料清洗。
* `index.html`：前端 Chart.js、篩選器與 KPI 介面。
* `appsscript.json`：專案設定。

## 4. 部署與推播 (clasp)
* 寫入 `.clasp.json`，並執行 `npx.cmd clasp push -f` 強制推播。
* 指引用戶在後台將「新增部署」設為「網頁應用程式」，權限為「所有人」。

## 5. 前端 UI/UX 與佈局最佳實踐
* **圖表定高防溢出 (CSS)**：防止 Chart.js 在視窗重繪時寫死行內高度導致相鄰彈性容器縮水。
  ```css
  .chart-container-wrapper { position: relative; width: 100%; height: 100%; }
  .chart-container-wrapper canvas { position: absolute; top:0; left:0; width:100% !important; height:100% !important; }
  ```
  *且 Chart.js options 須設 `responsive: true` 與 `maintainAspectRatio: false`。*
* **狀態記憶機制 (JS)**：重繪 DOM 時（如年份切換），讀取全域變數（如 `let isEnforcementExpanded;`）維持展開狀態。
* **控制標籤長度限制**：動態標籤字數最簡化（如 `全部年度`），並使用 CSS `text-overflow: ellipsis` 預防長文字變形。

## 🛠️ 排錯指南
* `nlm` 找不到 ➔ 將 Python Scripts 加到 PATH。
* GitHub Token 錯誤 ➔ 執行 `$env:GITHUB_TOKEN = $null`。
* clasp push 失敗 ➔ 使用 `-f` 強制覆蓋。
