# CRAVITY 本命曲排名：交接說明

> 給 Claude Code 接手用。最後更新：2026-11-02

## 專案目標

一個單頁網站，讓使用者用「二選一」比較 CRAVITY 的歌曲，排出完整名次，最後顯示前 10 名。每首歌都要能試聽，並發布在 GitHub Pages。

## 目前狀態

| 項目 | 狀態 |
|---|---|
| 二選一排序遊戲（105 首） | ✅ 完成，邏輯已用模擬測試過 |
| 顯示前 10 名，11–105 名可展開 | ✅ 完成 |
| 每首歌 30 秒試聽＋Apple Music 連結 | ⚠️ 已寫好，**尚未在真實網站上測試**（開發環境連不到 Apple） |
| 推送到 GitHub、開啟 Pages | ❌ 未完成：repo 還不存在 |

## 檔案

```
index.html   整個網站（HTML + CSS + JS 都在同一檔，無外部相依，除了 Google Fonts）
README.md    repo 說明
.nojekyll    讓 GitHub Pages 直接提供檔案
HANDOFF.md   本文件
```

## 待辦（依序）

1. **建立 repo 並推送**
   ```bash
   gh repo create <owner>/cravity-ranking --public --source=. --push
   ```
   （或在 github.com/new 手動建立後 `git remote add origin … && git push -u origin main`）
2. **開啟 GitHub Pages**（main 分支、根目錄）
   ```bash
   gh api repos/<owner>/cravity-ranking/pages -X POST \
     -f "source[branch]=main" -f "source[path]=/"
   ```
   網址：`（網址見 repo 設定）`
3. **實測試聽配對**：打開網站，在瀏覽器 console 執行
   ```js
   SONGS.filter(s => !MAP[s.id]).map(s => s.t)
   ```
   列出沒配到 Apple Music 的歌，再看下方「配對規則」調整，或把確定的曲目 ID 直接寫進 `SONGS`（`alb` + `i` 欄位）。
4. **（建議）把配對結果固定下來**：用 iTunes API 跑一次，把所有歌的 `trackId`／`collectionId` 寫死進 `SONGS`，網站就不必每次即時查詢，也比較穩。
   ```bash
   curl "https://itunes.apple.com/lookup?id=1507226584&entity=song&limit=200&country=tw"
   ```

## 程式架構（index.html 內的 `<script>`）

### 歌曲資料 `SONGS`
每筆：`{ t: 歌名, a: 專輯, y: 年份, alb?: Apple 專輯 ID, i?: Apple 曲目 ID }`，載入後自動加上 `id`（陣列索引）。
- 26 首已有手動確認過的 `alb`/`i`（來自 Apple Music 網頁）。
- 其餘 79 首靠執行期查詢。

### 排序：互動式合併排序（bottom-up）
- 狀態 `S = { queue, L, R, M, done, hist, final }`
  - `queue`：待合併的已排序組；一開始是每首歌各自一組（洗牌過）
  - `L`、`R`：正在合併的兩組；每場比 `L[0]` vs `R[0]`
  - `M`：本次合併已排好的部分
  - `final`：完成後的完整排名（索引 0 = 第 1 名）
- 105 首最多 608 場，模擬實測約 587 場。
- `remainingMax()` 計算剩餘最多場數（進度條用）。
- `hist` 保留最近 40 步供「上一步」。
- 進度存在 `localStorage`，key：`cravity-rank-all-v1`。**改動 `SONGS` 內容或順序時要換 key**，否則舊進度的 id 會對錯歌。

### Apple Music 試聽
- `resolveAll()`：頁面載入後執行
  1. 先讀 `localStorage` 快取（key `cravity-am-map-v1`，7 天有效）
  2. 用 JSONP 呼叫 `itunes.apple.com/lookup?id=1507226584&entity=song&limit=200&country=tw`
  3. `assign()` 依歌名配對，寫入 `MAP[id] = { url, preview }`
  4. 沒配到的歌用 `searchOne()` 逐首搜尋 `itunes.apple.com/search`
- **配對規則**
  - `norm()`：轉小寫、NFKC、去掉括號內容和符號
  - `keys()`：一首歌產生多個比對鍵，例如 `낯섦 (Stay)` → `낯섦`、`stay`
  - `BAD` 正規式排除伴奏、日文版、remix 等版本
  - 同名多筆時取最早發行的（＝原曲）
  - 有手動 `i` 的歌優先用曲目 ID 對
- 找不到時退回：手動 ID 連結 → Apple Music 搜尋連結。
- 播放：單一 `Audio` 物件，同時只播一首；選歌、上一步、重來時會停止。

### 其他
- 鍵盤 ← → 可選歌。
- 支援淺色／深色主題（CSS 變數，`prefers-color-scheme` 與 `data-theme`）。
- 手機寬度 720px 以下卡片改成上下排列。

## 收錄範圍（105 首）

收錄 CRAVITY 全員演唱的所有韓文、日文原創曲。**不收**：已收錄歌曲的日文版／伴奏版、個人 OST（Woobin〈hope you stay〉、Hyeongjun〈Starlight〉）、跨團合作曲〈Blue & Black〉、Road to Kingdom 翻唱。

| 年 | 發行 | 首數 |
|---|---|---|
| 2020 | HIDEOUT : REMEMBER WHO WE ARE - SEASON1. | 7 |
| 2020 | HIDEOUT : THE NEW DAY WE STEP INTO - SEASON2. | 7 |
| 2021 | HIDEOUT : BE OUR VOICE - SEASON3. | 7 |
| 2021 | The Awakening : Written In the Stars | 8 |
| 2022 | LIBERTY : IN OUR COSMOS | 8 |
| 2022 | VIVID（單曲） | 1 |
| 2022 | NEW WAVE | 6 |
| 2023 | MASTER : PIECE | 6 |
| 2023 | Groovy -Japanese ver.-（B 面 I Can't Fight The Feeling） | 1 |
| 2023 | 驅魔麵館 2 OST（Ready Set Go） | 1 |
| 2023 | SUN SEEKER | 6 |
| 2023 | Dilly Dally（日本 EP，原創 2 首） | 2 |
| 2024 | EVERSHINE | 6 |
| 2024 | SHOW OFF（日本單曲） | 2 |
| 2024 | FIND THE ORBIT | 3 |
| 2025 | Jelly Bean（日本 EP，原創 3 首） | 3 |
| 2025 | Dare to Crave | 12 |
| 2025 | Dare to Crave : Epilogue（新歌 3 首） | 3 |
| 2026 | BLAST OUT（日本單曲） | 3 |
| 2026 | ReDeFINE | 6 |
| 2026 | sonorous | 7 |

### 待確認的資料
- 〈좋아하나봐〉英文名：Wikipedia 寫 *Maybe Baby*，kpopofficial 寫 *My Baby*。目前用 Maybe Baby。
- 〈VIVID〉年份：Color Coded Lyrics 標 2020，kpopofficial 標 2022-05-31。目前用 2022。
- ReDeFINE 第 5 首：新聞稿兩種寫法（Love Me As You Do／Love Me Like You Do）。目前用 Apple Music 上的 Love Me Like You Do。

## 資料來源
- Color Coded Lyrics CRAVITY 索引（主要曲目來源）：https://colorcodedlyrics.com/2020/05/cravity-keulaebiti-lyrics-index
- Wikipedia Cravity：https://en.wikipedia.org/wiki/Cravity
- J-Lyric 日文歌曲列表：https://j-lyric.net/artist/a0655e7/
- sonorous 曲目：https://kpopofficial.com/album/cravity-sonorous/
- Dare to Crave : Epilogue 曲目：https://www.allkpop.com/article/2025/10/cravity-to-return-to-the-music-scene-with-their-2nd-full-length-album-dare-to-crave-epilogue
- BLAST OUT 曲目：https://www.asmart.jp/p_90052942
- Apple Music 藝人頁（Artist ID 1507226584）：https://music.apple.com/artist/1507226584
