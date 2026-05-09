# 生命數字計算器 — 接手摘要

## 檔案位置
`/mnt/user-data/outputs/mangala.html`(單一 HTML 檔,~1680 行)

## 專案概述
一個「生命數字 / Numerology」計算器,輸入出生日期 + 宇宙年 + 英文姓名,計算並視覺化:
- 命盤(高峰、挑戰、月日年中軸)
- 資訊牌(實歲/生命道路/制約數字 / 流年/目前高峰/目前挑戰)
- 名字字母→數字 + ESPM(表現/靈驅/人格/成熟)
- 業的圖表(1-9 數字統計長條圖)
- 性情數字(頭腦/情緒/行動/直覺)
- 流月(12 格一排)
- 個人年週期(9×2 格,1-9 數字位置固定,年份漂移)

## 命名規則(注意 — 已改名)
- topbar input 第 4 欄叫「**宇宙年**」(原本叫「流年」)
- 流年週期區叫「**個人年 N 週期**」(原本叫「流年 N 週期」)
- 但資訊牌中間「**2026流年**」格子保留「流年」字眼(使用者要求保留)
- 流月區「2026 流月 / 1-8 月仍流年 9 / 9 月起入流年 1」也仍用「流年」(未動)

## 重要邏輯規則

### 計算
- 生命道路 = (年→reduce) + (月→reduce) + (日→reduce) → reduce
- 制約數字 = 月 + 日 → reduce
- 流年(個人年) = (查詢年→reduce) + 月 + 日 → reduce
- 流月 = 流年 + 月 → reduce
- 高峰 P1=月+日, P2=日+年, P3=P1+P2, P4=月+年(都 reduce)
- 挑戰 C1=|月-日|, C2=|日-年|, C3=|C1-C2|, C4=|月-年|
- 名字 ESPM:E=全字母, S=母音, P=子音(含Y), M=E+生命道路
- **業的圖表**:統計 1-9 每個數字在名字中出現次數
- **性情數字**:頭腦=count(1)+count(8) / 情緒=count(2)+count(3)+count(6) / 行動=count(4)+count(5) / 直覺=count(7)+count(9)
- 沒有大師數字(11/22/33),全部 reduce 到單位數

### 實歲與高峰判定(關鍵 — 已改邏輯)
**「實歲」跟著查詢的宇宙年走,而不是固定為今天的歲數**
```js
let age = fy - y;
if (TODAY_M < m || (TODAY_M === m && TODAY_D < d)) age--;
const currentStage = (age <= e1) ? 0 : (age <= e2) ? 1 : (age <= e3) ? 2 : 3;
```
語意:**把今天的月日(TODAY_M/TODAY_D)投影到查詢年那一刻,看人幾歲**
- 例:1994/8/19 出生,fy=2026,今天 5/9 → 31 歲(還沒過 8/19)
- 例:1994/8/19 出生,fy=2027,投影到 2027/5/9 → 32 歲
- 高峰 stage 用同一個 age 判定,所以底色會跟著漂

### 個人年週期(關鍵)
- **1-9 數字位置固定** — 從「該人的個人年 1 那年」當週期起點
- **年份才會漂移** — 查詢年高亮在週期裡的對應位置
- 邏輯:`while (calcPersonalYear(rm, rd, cycleStart) !== 1) cycleStart--`
- 上排:當下週期 9 年(個人年 1→9)
- 下排:前一週期 9 年

### 姓名解析
- **逗號 `,`** = 姓/名分組(轉成空格分隔)
- **空格 + 連字號 `-`** = 同組裡的音節分隔(全部合併)
- 範例:`CHANG CHIEN,KUO-TING` → `["CHANGCHIEN", "KUOTING"]`
- 範例:`CHUNG YU FANG`(無逗號)→ `["CHUNG", "YU", "FANG"]`(三組分開顯示)

## CSS 變數(色票)
```
--ink: #0f1e3a (最深字色)
--ink-soft: #2a3a58
--ink-dim: #6b7a92
--ink-dimmer: #9aa5ba
--accent: #1e4a8a (主藍)
--accent-strong: #143868 (深藍 - 數字)
--accent-soft: #6585b0
--vowel: #b7384c (酒紅 - 母音/最終值/當下標示)
--band-prev: #b8cce4 (流月色帶 - 上一年流年)
--band-next: #1e4a8a (流月色帶 - 當下流年)
```

字型:Cormorant Garamond(英文 italic)、Noto Serif TC(中文)、JetBrains Mono(數字)

## 顏色邏輯
**酒紅(--vowel)用於**:名字母音字母 / ESPM 數字 + 字母(全形)/ 命盤中軸最終值 / 流月當月 / 業圖表 peak bar / 性情數字最大值 / 資訊牌實歲/生命道路/制約數字

**藍色用於**:標題、標籤、結構文字 / 命盤過去未來高峰挑戰 / 資訊牌流年/目前高峰/目前挑戰底色 / 流月當下流年色帶

## 排版結構

### 桌機板(>1000px)
```
標題(Numerology · 生命數字)+ tagline
└── layout grid(maxw 1200px)
    ├── chart(命盤)
    └── right
        ├── topbar(試算列:出生年月日 / 宇宙年 / 姓名 / [計算] / 姓名翻譯)
        ├── name-block(名字置頂 + 兩欄 [資訊牌 | 分隔線 | ESPM])
        ├── karma-traits-row(業圖表 1.7fr | 性情 1fr)
        ├── month-title-row + month-grid(12 欄)
        └── timeline(9×2,row-gap 2px)
```

### 手機板(<600px) — 已大改
DOM 用 `display: contents` + `order` 重排:
1. 試算列(topbar)— **改用 CSS Grid 鎖兩排**
   - 第一排:出生年 / 出生月 / 出生日 / 宇宙年 / 姓名翻譯 ↗(grid-cols `1fr 1fr 1fr 1fr auto`)
   - 第二排:姓名 input(grid-col 1/5 跨 4 欄)+ 計算按鈕(grid-col 5)
   - `.action-stack` 用 `display: contents` 把翻譯連結跟按鈕拆開到 grid 不同位置
   - 手機板隱藏 `.field label .hint`(桌機用 width:0 飄字的把戲在手機會溢出)
2. 命盤(chart)
3. 名字區(name-block 改單欄堆疊,分隔線隱藏)
4. 業圖表 + 性情(改上下堆疊;trait-row grid 改 `auto auto 1fr` — label/小數字靠左,大數字貼右)
5. 流月標題
6. **流月格子改 6×2(原本 4×3)**
7. 流年標題
8. 流年週期(9 欄)

## Fit-to-screen 縮放(新功能)
桌機板 JS 動態縮放,讓內容一頁裝得下:
```js
ratio = min(viewport_h / content_h, viewport_w / content_w, 1)
ratio = max(ratio, 0.7)  // 下限保護
wrap.style.transform = `scale(${ratio})`
wrap.style.transformOrigin = 'top center'
document.body.style.height = (naturalH * ratio) + 'px'
```
- 小筆電(1280-1366):縮 0.79~0.83
- 大螢幕(1920+):不縮、1:1
- 手機板 (≤600px):**完全不套用**,維持直式捲動
- 視窗 resize / 計算後都重算

## 初始空白 + 缺值容錯(新功能)
- 5 個 input 預設值都清空
- 首次載入仍用 1994/8/19/TODAY_Y/CHUNG YUFANG 渲染版面骨架,然後加 `placeholder-mode` class 把所有結果視覺蓋成 ✦
- `run()` 缺值時不洗版面,只加回 `placeholder-mode` class — 使用者已輸入的其他欄位會保留
- **計算後 input 不會被清空**(`controlsBar(vals)` 接收當前值,render 後保留輸入)

## 主要決定點(避免重複討論)
- ✅ 個人年週期 1-9 順序固定,年份漂移
- ✅ 姓名 CSV 用 grid 強制對齊(70px 24px auto 1fr)
- ✅ Ｅ Ｓ Ｐ Ｍ 用全形字保持寬度一致
- ✅ ESPM 中字母酒紅,中文名(表現/靈驅/人格/成熟)灰色,工作/親密/社交/36歲後淺藍 55%
- ✅ 業的圖表 + 性情數字並排,traits-vert 用 flex:1 + justify-content:center
- ✅ bar-chart 用 `flex: 1; min-height: 90px; padding: 18px 6px 0` 撐滿並上方留白
- ✅ 桌機流月 12 欄一排、手機 6×2,當月酒紅框 + 淡紅底 + 紅字
- ✅ 計算按鈕 height 28px(配合 input)
- ✅ 計算按鈕 = toggle 取消(直接重新計算,不切回空白)
- ✅ Placeholder mode:所有結果類元素變透明 ✦,流月色帶/今月底色/流年週期當下底色都拿掉
- ✅ 實歲跟著查詢年走(`age = fy - y` + 月日修正)
- ✅ 高峰判定用查詢年歲數,跟流年同步漂移
- ✅ 桌機 fit-to-screen 自動縮放,手機不套用
- ✅ 初始 input 空白,placeholder mode 起手
- ✅ 計算後 input 保留輸入(controlsBar 接 vals)

## Placeholder mode CSS 重點
所有「填入後才會出現的數字/底色」要在 `body.placeholder-mode` 下變透明 + 顯示 ✦,包含:
- ki-cell .num
- slot .val / .reduction / .age / .yr
- name-item .nval / .nformula
- letter-stack .ch / .num
- month-cell .pm-num
- tl-cell .v
- mini-title .mt-py / .mt-range
- bar (peak 紅色拿掉, height: 1px, count visibility: hidden)
- trait-row .tval (透明 ✦)
- 各種底色:band-curr / band-prev / today / tl-cell.now → transparent

## 標題
- 主標題:Numerology(英文 italic, clamp 28-40px) + 生 命 數 字(中文, 20px, 字距 0.4em),flex baseline align,gap 18px
- tagline:此**生命數字計算器**為小玩製作 · ⊡ yufangzhong(IG SVG 圖示 13×13px)

## 已知小細節(可考慮處理)
- placeholder 模式下,「2026流年」「2026 流月 / 1-8 月仍流年 9 / 9 月起入流年 1」這幾處的 2026 字眼還會顯示(因內部用 TODAY_Y 算骨架,placeholder mode CSS 沒蓋這幾處)。要不要在 placeholder 模式下也藏掉,看設計取捨。
- 業圖表底下 1-9 的 count 數字格式:0 顯示為「.0」,跟非零值「3」「2」「1」格式不一致,看起來有點怪。可能是 placeholder mode CSS 殘留或 `.bar.zero` 樣式問題。

## 如何接手
1. 開啟 `/mnt/user-data/outputs/mangala.html` 看現況
2. 桌機:寬螢幕看(自動 fit-to-screen)
3. 手機:縮小視窗到 <600px 看單欄重排
4. 一進入是 placeholder 模式 + input 空白
5. 填入年/月/日/宇宙年/姓名,按計算,看完整結果
6. 試切換不同宇宙年(2027、2030)驗證高峰底色跟著漂

## 範例輸入(測試用)
- 出生:1994/8/19
- 宇宙年:2026
- 姓名:CHUNG YUFANG
- 預期 2026:31 歲、第一高峰、流年 1
- 預期 2027:32 歲、第二高峰、流年 2
