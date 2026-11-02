# CRAVITY 本命曲排名：交接說明

> 給 Claude Code 接手用。最後更新：2026-11-02（前 30 名淘汰賽、範圍篩選、4:3 結果圖）

## 專案目標

一個單頁網站，讓使用者用「二選一」比較 CRAVITY 的歌曲，可先選專輯與主打／收錄曲範圍，只排出前 30 名，可輸出 4:3 結果圖。每首歌都要能試聽，並發布在 GitHub Pages。

## 目前狀態

| 項目 | 狀態 |
|---|---|
| 二選一排序遊戲（105 首） | ✅ 完成，邏輯已用模擬測試過 |
| 前 30 名淘汰賽、範圍篩選、4:3 結果圖 | ✅ 完成 |
| 每首歌 30 秒試聽＋Apple Music 連結 | ✅ 105 首 ID 已寫死並在線上實測 |
| GitHub repo＋Pages | ✅ （網址見 repo 設定） |

## 檔案

```
index.html   整個網站（HTML + CSS + JS 同一檔；外部只有 Google Fonts 與 Apple 封面／試聽）
img/*.webp   藍色貼紙裝飾（來源：IG貼文素材，已裁邊壓縮）
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
每筆：`{ t: 歌名, a: 專輯, y: 年份, alb: Apple 專輯 ID, i: Apple 曲目 ID, art: 封面網址（{s} 換尺寸）, pv: 試聽網址, tt?: 1＝主打歌 }`，載入後自動加上 `id`（陣列索引）。全部 105 首都已寫死，不再即時配對。

### 排序：淘汰賽樹，只取前 30 名
- 狀態 `S = { K, order, res, est }`：`order` 是洗牌後的對戰表，`res` 是每場結果 `[a, b, 勝者]`，`est` 是預估總場數。
- `solve(S)` 每次從頭用 `res` 重算：打一輪淘汰賽得到第 1 名，拿掉後重算，冠軍路徑以外的場次沿用舊結果；遇到沒比過的組合就回傳 `{need:[a,b]}`，排滿 K 名回傳 `{final}`。
- 105 首約 266 場（全排序約 587 場）；上一步＝`res.pop()`。
- 進度與篩選存在 `localStorage`，key：`cravity-rank-v2`。**改動 `SONGS` 順序時要換 key**。
- 主打歌（`tt`）：各專輯／單曲主打，含 Cheese（SUN SEEKER 先行曲）；〈Boogie Woogie〉與 OST〈Ready Set Go〉目前算收錄曲。

### 4:3 結果圖
`exportImage()` 用 canvas 畫 2000×1500 PNG：左側粉藍區塊放第 1–3 名，右側 3×9 放第 4–30 名。封面從 mzstatic 以 `crossOrigin="anonymous"` 載入（有 CORS），字型先用 `document.fonts.load` 載好。

### Apple Music 試聽
- 直接用 `pv` 播放；播放失敗時（試聽網址失效）用 JSONP 呼叫 `itunes.apple.com/lookup?id=<i>` 重取一次。
- 單一 `Audio` 物件，同時只播一首；選歌、上一步、重來時會停止。`sync()` 依 `data-play` 屬性切換播放狀態。

### 其他
- 鍵盤 ← → 可選歌。
- 支援淺色／深色主題（CSS 變數，`prefers-color-scheme` 與 `data-theme`）。
- 對決時兩首歌永遠橫向並排，只顯示專輯資訊；「選這首」靠卡片底部對齊。點封面也能選。
- 視覺參考 Figma「Alliatus – Mastermind Landing Page」，黃色改藍色：淺灰底、白卡、粉藍色塊、Playfair Display＋Noto Serif TC 標題、黑色膠囊按鈕、星芒。

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
