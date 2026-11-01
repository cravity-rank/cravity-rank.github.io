# CRAVITY 本命曲排名：交接說明

> 給 Claude Code 接手用。最後更新：2026-11-02（已上線、ID 已寫死、改版為封面介面）

## 專案目標

一個單頁網站，讓使用者用「二選一」比較 CRAVITY 的歌曲，排出完整名次，最後以專輯封面九宮格顯示前 9 名。每首歌都要能試聽，並發布在 GitHub Pages。

## 目前狀態

| 項目 | 狀態 |
|---|---|
| 二選一排序遊戲（105 首） | ✅ 完成，邏輯已用模擬測試過 |
| 結果：前 9 名封面九宮格，10–105 名可展開 | ✅ 完成 |
| 每首歌 30 秒試聽＋Apple Music 連結 | ✅ 105 首 ID 已寫死並在線上實測 |
| GitHub repo＋Pages | ✅ （網址見 repo 設定） |

## 檔案

```
index.html   整個網站（HTML + CSS + JS 都在同一檔，無外部相依，除了 Google Fonts）
README.md    repo 說明
.nojekyll    讓 GitHub Pages 直接提供檔案
HANDOFF.md   本文件
```

## 更新 Apple Music 資料
iTunes 查詢 API 可直接用 curl 取得：
```bash
curl "https://itunes.apple.com/lookup?id=<專輯ID>&entity=song&country=tw"
```
每首歌一律連到 `a` 欄位所寫的**原版專輯**（不用單曲或改版專輯）。新增歌曲時補上 `alb`、`i`、`art`、`pv` 四個欄位。

## 程式架構（index.html 內的 `<script>`）

### 歌曲資料 `SONGS`
每筆：`{ t: 歌名, a: 專輯, y: 年份, alb: Apple 專輯 ID, i: Apple 曲目 ID, art: 封面網址（{s} 換尺寸）, pv: 試聽網址 }`，載入後自動加上 `id`（陣列索引）。全部 105 首都已寫死，不再即時配對。

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
- 直接用 `pv` 播放；播放失敗時（試聽網址失效）用 JSONP 呼叫 `itunes.apple.com/lookup?id=<i>` 重取一次。
- 單一 `Audio` 物件，同時只播一首；選歌、上一步、重來時會停止。`sync()` 依 `data-play` 屬性切換播放狀態。

### 其他
- 鍵盤 ← → 可選歌。
- 支援淺色／深色主題（CSS 變數，`prefers-color-scheme` 與 `data-theme`）。
- 對決時兩首歌永遠橫向並排（封面在上，資訊與試聽在下）；點封面也能選。
- 視覺：米白紙感底、Noto Serif TC＋Cormorant Garamond＋IBM Plex Mono。

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
