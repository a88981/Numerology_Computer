# Relation.html · 流年關係組合計算器 · Handoff

## 專案概要

`relation.html` — 單一 HTML 檔的關係流年計算器,跟 `mangala.html` 同一個系列(同設計語言:冷藍 + 酒紅 / Cormorant Garamond + Noto Serif TC + JetBrains Mono / 無 emoji)。

**核心功能**:輸入兩人個人流年(或生日),計算當年的關係週期能量主題。

**檔案位置**:`/mnt/user-data/outputs/relation.html`(目前 1638 行 ≈ 95KB)

---

## 三種模式

預設順序(從左到右): `[ 自由試算 ][ 單一年 ][ 年份範圍 ]`,**預設 active 是「自由試算」**。

### 1. 自由試算(manual)— 預設模式
- 不需要生日,直接點 1-9 數字選擇雙方個人流年
- 隱藏生日輸入區,在原位置顯示 1-9 圓鈕
- 選完雙方數字 → 自動計算並顯示完整結果(summary-bar 不含年份字眼)
- 隱藏「計算」按鈕跟查詢年 input

### 2. 單一年(single)
- 雙方生日(年/月/日) + 一個查詢年
- 按「計算」→ 顯示 summary-bar(2026 自己流年 / 2026 對方流年 / 關係年) + 個人流年 + 關係年主題 + 組合動態

### 3. 年份範圍(range)
- 雙方生日 + 起始年 → 結束年
- 顯示**橫排 chip 網格**(每個年份一格 chip)
- chip 內容:年份(灰小字) → 雙方個人流年公式 `A+B`(藍+紅) → 關係年(酒紅大字) → 短主題
- 點任一 chip → 下方統一展開區顯示完整結果(summary-bar + 個人流年 + 關係年主題 + 組合動態)
- 再點同一 chip → 收合

---

## 主要結構元素

### HTML 結構
```
.input-section
├── .birthday-block#birthdayBlock
│   ├── .bb-mode-birthday  (一般模式:生日輸入)
│   │   ├── .person-row [自己 / 年 / 月 / 日]
│   │   └── .person-row [對方 / 年 / 月 / 日]
│   └── .bb-mode-manual    (自由試算模式:1-9 按鈕,取代生日)
│       ├── .person-row.mp-person-row [自己 / 1-9 buttons]
│       └── .person-row.mp-person-row [對方 / 1-9 buttons]
└── .mode-row
    ├── .mode-options (segmented tabs:自由試算/單一年/年份範圍)
    ├── .year-inputs#yearInputs (查詢年 / 起始→結束 / 自由試算空)
    └── button#calc (自由試算時隱藏)

.result-section#result
└── [動態渲染]
```

### 渲染函式
- `renderPersonalPair(py1, py2)` — 個人流年雙欄(✦ 個人流年 eyebrow)
- `renderCard(num)` — 關係年主題卡(✦ 關係年 eyebrow + 完整 desc + 三組標籤:核心關鍵字/適合做/需注意)
- `renderCombo(py1, py2, rel)` — 組合動態(✦ X×Y → 關係年Z 的組合動態 eyebrow)
- `renderSingle(...)` — 組合所有 → summary-bar + 三區塊
- `renderRange(...)` — 生成 chip 網格 + 空 detail 容器
- `bindRowToggle()` — chip 點選展開邏輯

---

## 三組資料

### personalData(9 個)
每個個人流年數字的詳細描述。
- `{ en, title, desc }`
- 例:`1: { en: 'Sowing', title: '播種 · 新週期啟動', desc: '...' }`
- 其他:Awaiting(等待)、Expression(表達)、Building(建設)、Freedom(自由)、Devotion(責任)、Reflection(內省)、Achievement(成就)、Release(放下)

### themeData(9 個)
每個關係年的主題描述。
- `{ en, title, desc, key[3], good[3], warn[3] }`
- 例:`1: { en: 'The New Beginning', title: '新開始 · 重新出發', desc: '...', key: [...], good: [...], warn: [...] }`

### comboData(45 個動態函式)
兩個個人流年配對的組合動態,**根據關係年 r 變化內容**。
- key: `"1-1"` ~ `"9-9"`(小數字-大數字)
- value: `r => string` — 接收關係年 r,回傳對應描述
- 每個 key 內部使用三元運算子針對 4-8 個特定 r 寫不同描述,其他用 default
- `getCombo(a, b, rel)` 呼叫對應函式
- 模板字面值:`` `...${themeData[r].title}...` `` — 描述裡會插入關係年的中文主題

---

## CSS 變數(設計 token)

```css
--accent: #1e4a8a        /* 主藍 */
--accent-strong: #163c70 /* 深藍 */
--accent-soft: #aab6c8   /* 淺藍 */
--accent-bg: rgba(...)   /* 藍底 */
--vowel: #8b3a4a         /* 酒紅 */
--vowel-soft: rgba(...)  /* 淺酒紅底 */
--ink: #1a1f2e
--ink-soft: ...
--ink-dim: ...
--ink-dimmer: ...
--line / --line-soft     /* 邊框/分隔線 */
--serif-en: 'Cormorant Garamond'  /* 英文 italic */
--serif-tc: 'Noto Serif TC'        /* 中文 */
--mono: 'JetBrains Mono'           /* 數字 */
```

## 關鍵 CSS 模式

### Segmented control(mode tabs)
- `.mode-options`:容器,細邊框
- `.mode-tab`:每個 tab(label),`border-right` 1px 分隔
- `.mode-tab.active`:藍底 `--accent` + 白字
- radio input `opacity: 0; position: absolute; pointer-events: none`
- JS 用 change event 切換 active class

### Person-row grid 對齊技巧 ⚠️
**一般 person-row**:`grid-template-columns: 60px minmax(0, 1fr) minmax(0, 1fr) minmax(0, 1fr)` — label + 年/月/日

**mp-person-row(1-9 按鈕)**:
```css
.mp-person-row {
  grid-template-columns: 60px minmax(0, 1fr) !important;
  min-height: 56px;          /* 跟一般 person-row 高度齊 */
  align-items: center !important;
}
.mp-btns-inline {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;  /* 撐滿整個 row 寬,左右對齊 */
  width: 100%;
}
```
⚠️ **`justify-content: space-between` 很關鍵** — 不然 9 個按鈕只擠在左邊,右邊大片空白,跟生日輸入欄(年/月/日撐滿三欄)視覺不一致。

### Year-chip(範圍模式)
```css
.year-chips {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(96px, 1fr));
  gap: 8px;
}
```
chip 內 4 行垂直堆疊:`chip-year` / `chip-formula(自己藍+對方紅)` / `chip-num` / `chip-title`

### tags 不換行 ⚠️
```css
.tags { display: flex; flex-wrap: nowrap; gap: 4px; }
.tag {
  font-size: 11px;
  padding: 2px 7px;
  white-space: nowrap;
  flex-shrink: 0;
}
```
原本三標籤太寬會換行,改 nowrap + 縮 padding 後可一排塞下三個 4 字標籤(實測 row 寬 217px,三標籤 total 174-196px)。

### 隱藏 number input spinner
```css
.field input[type="number"]::-webkit-outer-spin-button,
.field input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
.field input[type="number"] {
  -moz-appearance: textfield;
  appearance: textfield;
}
```
這是 number input 預設會有上下箭頭 spinner,在桌機/手機都會穿出 input 底線造成視覺超出框。隱藏掉。

### Field input 寬度 ⚠️
```css
.field input {
  width: 100%;
  box-sizing: border-box;
  min-width: 0;
  /* ... */
}
```
input 預設 width 約 213px,會撐爆 grid `1fr` cell。要設 `100%` + `box-sizing: border-box` + `min-width: 0`,grid 用 `minmax(0, 1fr)`。

---

## JS 重要邏輯

### updateModeUI()
切換三模式時更新 UI:
```js
if (mode === 'single') {
  // 顯示查詢年 input + 計算按鈕
  // birthday-block 移除 manual-mode class
} else if (mode === 'range') {
  // 顯示 起始年→結束年
} else if (mode === 'manual') {
  // 清空 yearInputs、加 no-fy(隱藏計算按鈕)
  // birthday-block 加 manual-mode class(切換 inner 顯示)
}
```
頁面載入時呼叫一次 `updateModeUI()`,確保預設 manual 模式 UI 正確套用。

### initManualPick (IIFE)
- 生成 1-9 按鈕並綁定 click
- 選完雙方 → 寫到 `#result` 內容包含:summary-bar(無年份字眼:「自己流年 / 對方流年 / 關係年」) + 個人流年 + 關係年主題 + 組合動態
- 監聽 mode change:離開 manual → 重置 empty state「填入雙方生日與年份,按計算」;進入 manual 已選 → 重新渲染;進入 manual 未選 → empty state「選擇兩人的個人流年數字」

### run()
單一年/年份範圍模式的「計算」按鈕觸發:
- 自由試算模式 early return(按鈕已被隱藏)
- 驗證生日輸入
- single → renderSingle / range → renderRange + bindRowToggle

---

## 重要設計決定

1. **mode 順序**:`[自由試算][單一年][年份範圍]` — 從輕量(不需生日)到完整(多年),預設自由試算讓新使用者落地就能玩
2. **「範圍」改「年份範圍」**:避免使用者誤解
3. **生日區跟 1-9 按鈕共用 person-row 結構**:切換模式時視覺位置不變(自己/對方 label 不動,右邊內容互換),避免內容跳動
4. **自由試算 summary-bar 不含年份字眼**:單一年用「2026 自己流年」,自由試算用「自己流年」
5. **三個 section 都有 ✦ eyebrow**:`✦ 個人流年` / `✦ 關係年` / `✦ 1×8 → 關係年9 的組合動態`
6. **chip 公式上色**:自己 = 藍 `--accent-strong`,對方 = 酒紅 `--vowel`,加總 = 灰,最終 = 酒紅實底白字(只在 sum 跟 reduced 不同時顯示「→ N」)
7. **mangala 一樣的設計語言**:冷藍 + 酒紅,無 emoji,無漸層,1px 細邊框,segmented controls,Cormorant Garamond italic eyebrow

---

## 已修過的坑 / 學到的教訓

1. **grid `1fr` + input 預設 width = 撐爆容器** → 用 `minmax(0, 1fr)` + input `width: 100% + box-sizing: border-box + min-width: 0`
2. **number input spinner 視覺溢出** → CSS 隱藏 spinner
3. **flex-wrap: wrap 標籤換行不好看** → 用 `nowrap + white-space: nowrap + flex-shrink: 0` 強制單行,字級略收
4. **切換 mode 時容器高度不一致** → mp-person-row 加 `min-height: 56px`(讓 1-9 按鈕區跟生日 input+label 等高)
5. **1-9 按鈕內容區「視覺上」比生日輸入窄** → `justify-content: space-between` + `width: 100%` 撐滿寬度
6. **預設模式從 single 改 manual 時** → 要在頁面載入時呼叫一次 `updateModeUI()` 讓 manual-mode class 套上
7. **result-section 預設 empty state 文字** → 改成「選擇兩人的個人流年數字」配合預設自由試算

---

## 已知限制 / 可能改進

- 範圍模式跨度超過 ~15 年時 chip 數量多,可考慮加滾動
- 自由試算沒有歷史紀錄/比較功能
- 沒有「分享」或「截圖」功能(mangala 有 html2canvas 截圖,relation 還沒做)
- 配色目前固定為冷藍+酒紅,沒有深色模式切換

---

## 相關檔案

- `/mnt/user-data/outputs/relation.html` — 本檔(1638 行)
- `/mnt/user-data/outputs/mangala.html` — 命盤計算器(姊妹檔,同設計語言)
- `/mnt/user-data/outputs/mangala_handoff.md` — mangala 的 handoff 文件
