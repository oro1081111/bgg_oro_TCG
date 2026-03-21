# Pocket Monster TCG — Future Plan / 上線規劃

> 目的：整理「目前狀態」、「上傳 GitHub 讓其他人遊玩需要補什麼」、以及「Firebase 是否必要」等後續工作方向。

---

## 0. 現況快速結論

### 抽卡是否有「每日更新」？
**目前沒有真的每日更新機制。**

目前卡包數量 `packs` 是存在：
- `localStorage.playerParams.packs`

抽卡時會 `packs--`，但程式碼沒有任何「日期判斷 / 每日重置」邏輯（找不到 `new Date()` / `Date()` 之類的操作）。

因此：
- 你今天抽完 = `packs=0`
- 明天打開仍然是 `packs=0`（除非手動清 localStorage 或改資料）

---

## 1. 發佈到 GitHub 讓其他人遊玩（最小可行）

### 方案 A：GitHub Pages（推薦：純靜態頁）
若你希望「不用後端」、「打開網址就能玩」，**GitHub Pages 是最簡單的**。

**前提**：遊戲需能純前端運作（目前你的 `index.html + cards.js + assets` 可以）。

建議步驟：
1. 確認專案入口檔
   - `pocket-monster-tcg/index.html`
   - `pocket-monster-tcg/cards.js`（卡片資料目前由它提供）
2. 在 GitHub Pages 指定發布來源
   - 選擇 `main` 分支 + `/docs` 或 root，或使用 GitHub Actions build 到 `gh-pages` 分支
3. 若用 **repo 子路徑**（例如 `https://user.github.io/bgg_oro_analysis/`）
   - 所有相對路徑要能正確載入（例如 `cards.js`、圖片網址等）
   - 目前 `index.html` 用 `script src="cards.js"` 屬於相對路徑，一般可行

補強建議：
- 加一個「一鍵重置資料」按鈕（清 localStorage），方便玩家卡住時自救。
- 將所有 `alert()` 統一改成同一套 modal（目前我們已經新增 `system-modal`，但其它 alert 還沒全改）。

### 方案 B：Vite build 後部署（如果要走 React 版本）
你的資料夾同時存在 Vite/React 架構（`src/`, `vite.config.ts`, `package.json`），但目前實際在玩的像是**純靜態 `index.html`**。

若要用 Vite/React 部署到 GitHub Pages，通常需要：
- `npm install`
- `npm run build` 產出 `dist/`
- 設定 Vite `base`（repo 子路徑時必須），例如：
  - `base: '/<repo>/'`
- 用 GitHub Actions 把 `dist/` 發佈到 Pages

> 註：若要走方案 B，需要先釐清「目前真正在用的是哪一套 UI/程式碼」（靜態版 or React 版），避免兩套並存造成維護成本。

---

## 2. Firebase 是否必要？

### 不用 Firebase 也可以（純單機 / 純前端）
如果你的目標是：
- 玩家進網址就玩
- 收藏、牌庫、勝率等都存在玩家自己的瀏覽器

那 **完全可以不用 Firebase**。目前你已經用 localStorage 做到：
- 收藏 collectedIds
- 牌庫 decks
- 最愛 favorites
- 爆擊加成 cardCritRates
- 玩家資訊 playerParams

缺點：
- 換手機/換瀏覽器資料就沒了
- 清快取資料就沒了
- 無法做全球排行、多人連線、跨裝置同步

### 什麼情況「需要 Firebase / 後端」
當你想要以下功能時，Firebase（或任何後端）才會變得有價值：
1. **登入與雲端存檔**（Google / Email 登入，跨裝置同步）
2. **排行榜、戰績統計**（全玩家可比較）
3. **多人連線對戰**
4. **防作弊**（抽卡、道具、資源由伺服器判定）
5. **每日重置/活動**（由伺服器提供「今天是第幾天/有哪些活動」，避免玩家改本機時間）

結論：
- **想先快速給別人玩：不用 Firebase**
- **想走長期營運：Firebase/後端可再加**

---

## 3. 每日卡包更新（Roadmap）

### 版本 1：純前端每日重置（不需要 Firebase）
作法：
- 在 localStorage 記錄 `lastPackResetDate`（例如 `YYYY-MM-DD`）
- 每次 `init()` 時取出，跟「今天」比較
- 如果不是同一天：把 `packs` 重設回 8（或指定值）

優點：簡單、零成本。

缺點：玩家可以透過改系統時間作弊。

### 版本 2：伺服器/雲端每日重置（可用 Firebase）
作法：
- 以 server timestamp 為準
- 每天產生一個每日狀態（活動、卡包、任務）

優點：較不易作弊、可做活動。
缺點：需要後端（Firebase / Cloudflare Workers / 自架 API）。

---

## 4. 上線前建議補的「基本體驗」

### (A) UI / UX
- [ ] 把剩餘的 `alert()` 也改成同風格的 `system-modal`
- [ ] 加入「設定」頁：
  - 重置存檔（清 localStorage）
  - 音效/震動（如果未來要）
- [ ] 抽卡頁面：明確顯示「每日重置規則」（目前尚未實作）

### (B) 內容與平衡
- [ ] 抽卡機率/稀有度機制（目前抽卡是全卡池均勻抽）
- [ ] 進階模式配方固定（目前有配方概念）
- [ ] 爆擊成長曲線調整（目前：0→10%，之後乘 1.1，上限 50%）

### (C) 工程/維運
- [ ] 決定：只保留「靜態版」或只保留「React 版」其中一套
- [ ] 建立 GitHub Actions（如果走 Vite build 部署）
- [ ] 加入版本號與變更紀錄（CHANGELOG）

---

## 5. 下一步我建議你先決定的 2 個問題

1. 你要以哪個版本為主？
   - 靜態 `index.html`（最快發佈、最少依賴）
   - React/Vite（可擴充、較乾淨的模組化，但需要建置與部署流程）

2. 每日卡包重置：你能接受玩家改時間作弊嗎？
   - 可接受 → 純前端每日重置就好
   - 不可接受 → 要後端（Firebase 只是其中一種選擇）

