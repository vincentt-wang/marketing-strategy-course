# Slide System Skill — Interactive Workshop Page
> 這份文件是完整的製作規格。照著這份 spec，任何人都能從零複製出同款風格、動畫、互動邏輯的工作坊簡報頁面。
> Last updated: 2026-04

---

## 0. 核心概念

這個系統的本質是：**一個 HTML 檔案 = 整場工作坊**。

- 用 CSS `scroll-snap` 做出像簡報一樣的翻頁體驗
- 每一「頁」是一個 `<section class="slide">`，對應一個 data 物件
- 所有頁面由 JavaScript 根據 `D[]` 資料陣列動態渲染，不用手寫每一頁
- 互動元件（點擊揭示、計時器、折疊展開）全部用 CSS + JS，無外部依賴
- 單一 `.html` 檔案，可直接部署在 GitHub Pages

---

## 1. 設計語言

### 1.1 視覺風格
| 維度 | 規格 |
|---|---|
| 底色 | 深黑 `#0a0a0f`，不是純黑，是帶一點藍紫的暗色 |
| 格線背景 | 1px 網格線，`rgba(255,255,255,.018)`，給空間感但不搶戲 |
| 光暈 | 每頁右下角 + 左上角各一個大橢圓 radial-gradient，顏色對應 section |
| 動態背景 | `aurora` keyframe，讓光暈位置緩慢浮動（18s loop） |
| 文字 | `#f0f0f5`（主要）/ `rgba(240,240,245,.55)`（次要） |
| 圓角 | 按鈕 `8px`，卡片 `12px`，大容器 `14–16px` |
| 邊框 | `rgba(255,255,255,.08)` 基礎，hover 時加亮 |

### 1.2 色彩系統
```css
:root {
  --bg:     #0a0a0f;
  --bg2:    #111118;
  --ink:    #f0f0f5;
  --mid:    rgba(240,240,245,.55);
  --line:   rgba(255,255,255,.08);

  --orange: #FF6B00;
  --red:    #FF2055;
  --blue:   #3B82F6;
  --purple: #A855F7;
  --green:  #22C55E;
  --yellow: #FBBF24;
  --cyan:   #06B6D4;

  --ease-back:   cubic-bezier(0.34, 1.56, 0.64, 1);  /* 有彈性的 bounce */
  --ease-smooth: cubic-bezier(0.4, 0.0, 0.2, 1);      /* Material 標準 */
  --min-font:    clamp(0.9rem, 1.3vw, 1.05rem);
  --min-label:   clamp(0.78rem, 1.1vw, 0.9rem);
}
html { font-size: 20px; }
```

### 1.3 Section 背景色（每個章節有不同底色）
```css
.slide[data-sec="1"] { background: linear-gradient(165deg, #1c1130, #241535, #2c1628, #1e1020); }
.slide[data-sec="2"] { background: linear-gradient(165deg, #0c1830, #112240, #142848, #0c1828); }
.slide[data-sec="3"] { background: linear-gradient(165deg, #0c201a, #102818, #143020, #0c1e14); }
.slide[data-sec="4"] { background: linear-gradient(165deg, #201808, #281e08, #221c10, #180e08); }
.slide[data-sec="5"] { background: linear-gradient(165deg, #16102a, #1e1438, #221840, #14102a); }
```

### 1.4 Light Themes（淺色頁面）
部分頁面（練習題、轉場）需要換成淺色背景，在 data 物件加 `theme` 屬性：
```
theme: 'light'       → 米白 #f9f8f5
theme: 'warm'        → 奶油白 #fffbf4
theme: 'blue-light'  → 淺藍 #f0f5ff
theme: 'green-light' → 淺綠 #f0faf5
```

---

## 2. 排版系統

```css
.hero-txt { font-size: clamp(3.5rem, 10vw, 8rem); font-weight: 800; letter-spacing: -0.035em; }
h1        { font-size: clamp(2.4rem, 6.5vw, 5.5rem); font-weight: 800; letter-spacing: -0.03em; }
h2        { font-size: clamp(1.6rem, 3.6vw, 3rem);   font-weight: 700; }
h3        { font-size: clamp(1rem, 1.8vw, 1.6rem);   font-weight: 600; }
p, li     { font-size: clamp(0.9rem, 1.45vw, 1.15rem); color: var(--mid); }
```

### 漸層文字（gradient text）
```css
.g-orange { background: linear-gradient(135deg, #FF6B00, #FF9A00); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.g-red    { background: linear-gradient(135deg, #FF2055, #FF6080); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.g-blue   { background: linear-gradient(135deg, #3B82F6, #60A5FA); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.g-purple { background: linear-gradient(135deg, #A855F7, #C084FC); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.g-green  { background: linear-gradient(135deg, #22C55E, #4ADE80); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
.g-multi  { background: linear-gradient(135deg, #FF6B00, #FF2055, #A855F7, #3B82F6); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
```

---

## 3. 頁面結構 Boilerplate

```html
<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>工作坊標題</title>
  <style>
    /* 貼入 CSS（見上方色彩 + 排版系統 + 元件樣式）*/
  </style>
</head>
<body>
  <!-- 進度條 -->
  <div id="prog"><div id="prog-fill"></div></div>
  <!-- 麵包屑 -->
  <div id="breadcrumb">Section 1 · 開場</div>
  <!-- 章節指示器 -->
  <div id="sec-indicator">① 第一章</div>
  <!-- 頁碼 -->
  <div id="counter">1 / 50</div>
  <!-- 右側圓點導覽 -->
  <div id="dot-nav"></div>
  <!-- 左側章節軌道 -->
  <div id="sec-rail"></div>
  <!-- 上下頁按鈕 -->
  <button class="nav-btn" id="btn-prev">↑</button>
  <button class="nav-btn" id="btn-next">↓</button>
  <!-- 簡報容器 -->
  <div id="slides-container"></div>
  <!-- 底部章節導覽列 -->
  <div id="sec-nav">
    <button class="sec-btn active" data-sec="1" onclick="jumpToSection(1)">① 第一章</button>
    <button class="sec-btn" data-sec="2" onclick="jumpToSection(2)">② 第二章</button>
  </div>

  <script>
    const D = [ /* 所有 slide 資料物件 */ ];
    /* 貼入完整 renderSlide() + 導覽 JS */
  </script>
</body>
</html>
```

---

## 4. Slide 資料物件格式

每個 slide 是 `D[]` 陣列中的一個物件：

```js
{
  id:    'unique-id',      // 唯一識別碼，用於 xref 跳轉
  sec:   1,                // Section 編號（決定背景色和底部 nav 高亮）
  sub:   '章節名｜小標',   // 麵包屑顯示文字
  type:  'slide-type',     // ← 決定渲染哪種版型（見下方完整清單）
  chip:  ['標籤文字', 'orange'], // 左上角標籤，顏色：orange/red/blue/purple/green/yellow/cyan/white
  title: '標題',
  theme: 'warm',           // 可選，淺色主題
  xrefs: [{id:'s05', label:'創新三元'}], // 可選，右下角跳轉連結
  time:  180,              // 可選，計時器秒數
}
```

---

## 5. 核心版型（Slide Types）完整清單

### 5.1 基礎版型

#### `cover` — 封面
```js
{ type: 'cover', title: '標題<br>第二行', sub2: '副標 / 日期 / 講者' }
```
呈現：超大字、多色漸層、居中

#### `section` — 章節過渡頁
```js
{ type: 'section', chip: ['Section 2 收尾', 'white'], title: '大標<br>第二行', sub2: '下一章預告', theme: 'light' }
```
呈現：居中大字，用於各 section 結尾

#### `section-cover` — 章節開場
```js
{ type: 'section-cover', secNum: '②', secTitle: '欲求性', secEn: 'Desirability', sub2: '你在對誰說話' }
```
呈現：大圓數字 + 章節名 + 英文 + 副標，全頁居中

#### `big-quote` — 金句頁
```js
{ type: 'big-quote', chip: ['關鍵問句', 'red'], quote: '你做的東西<br><span class="g-red">有人真的需要嗎</span>', sub2: '補充說明文字', theme: 'warm' }
```
呈現：超大引言字，下方小字說明

#### `dark` — 要點清單頁
```js
{
  type: 'dark',
  chip: ['重點', 'orange'],
  title: '頁面標題',
  bullets: ['要點一', '要點二', '<strong>強調文字</strong>'],
  footer: '底部備注文字'
}
```
呈現：左對齊要點列表，每點前有 ▸ 符號

#### `rules` — 規則說明頁
```js
{
  type: 'rules',
  chip: ['Before We Start', 'white'],
  title: 'Two things.',
  rules: ['第一條規則', '第二條規則'],
  footer: '備注文字'
}
```

---

### 5.2 互動揭示版型

#### `reveal` — 單一點擊揭示
```js
{
  type: 'reveal',
  chip: ['思考題', 'purple'],
  title: '問題標題',
  question: '這是鎖住的問題',
  answer: '<strong>答案在這裡</strong>，點擊後才顯示',
  confidential: true  // 加上 CONFIDENTIAL 印章
}
```
互動：整個卡片點擊 → 遮罩消失 → 答案浮現

#### `multi-reveal` — 多格點擊揭示（2/3/4 欄）
```js
{
  type: 'multi-reveal',
  chip: ['互動練習', 'green'],
  title: '標題',
  cols: 2,  // 2、3 或 4
  cards: [
    { q: '問題一', label: 'LABEL', title: '卡片標題', body: '卡片內容', col: 'orange' },
    { q: '問題二', label: 'LABEL', title: '卡片標題', body: '卡片內容', col: 'blue' },
  ]
}
```

#### `step-reveal` — 逐步展開列表
```js
{
  type: 'step-reveal',
  chip: ['步驟拆解', 'cyan'],
  title: '標題',
  steps: [
    { num: '01', q: '第一步問題', answer: '展開後的答案說明' },
    { num: '02', q: '第二步問題', answer: '展開後的答案說明' },
  ]
}
```
互動：點擊每一步 → 往下展開答案（accordion）

---

### 5.3 視覺框架版型

#### `triad-v2` — 三元素互動卡
```js
{
  type: 'triad-v2',
  chip: ['Innovation Triad', 'blue'],
  title: '創新需要三個條件同時成立',
  orbs: [
    { en: 'Desirability', zh: '欲求性', tagline: '有人真的想要嗎？', col: 'var(--red)', glow: 'rgba(255,32,85,.35)',
      details: [{ tag: 'CORE', txt: '詳細說明' }] },
    { en: 'Feasibility',  zh: '可行性', tagline: '你做得出來嗎？',   col: 'var(--blue)', glow: 'rgba(59,130,246,.35)',
      details: [{ tag: 'CORE', txt: '詳細說明' }] },
    { en: 'Viability',    zh: '存續性', tagline: '能持續存在嗎？',   col: 'var(--green)', glow: 'rgba(34,197,94,.35)',
      details: [{ tag: 'CORE', txt: '詳細說明' }] },
  ]
}
```
互動：點擊任一 orb → 展開詳細說明 panel

#### `venn` — 可點擊文氏圖
```js
{ type: 'venn', chip: ['創新三元', 'blue'], title: '三個條件的交集' }
```
互動：點擊三個圓和交集區 → 右下方出現對應說明

#### `dfv-combinations` — DFV 陷阱組合
```js
{ type: 'dfv-combinations', chip: ['三種陷阱', 'red'], title: '只靠一個，就是陷阱' }
```
互動：點擊各象限 → 出現對應說明

#### `octalysis` — 八角化驅動力
```js
{
  type: 'octalysis',
  chip: ['Octalysis 八角化', 'purple'],
  title: '8 個讓人停不下來的驅動力'
}
```
互動：點擊每個驅動力按鈕 → accordion 展開描述 + 案例

---

### 5.4 練習 + 計時器版型

#### `exercise-tpo-v2` — TPO 練習（含計時器）
```js
{ type: 'exercise-tpo-v2', chip: ['TPO 練習', 'purple'], title: 'Time · Place · Occasion', time: 180, theme: 'blue-light' }
```

#### `fermi-practice` / `fermi-game-v2` — 費米估算
```js
{
  type: 'fermi-game-v2',
  chip: ['費米估算', 'cyan', 'lg'],
  title: '全台灣一天賣出幾杯手搖飲',
  subtitle: '先猜一個數字，再一起推導'
}
```

#### `timer` — 純計時器頁
```js
{ type: 'timer', chip: ['計時', 'orange'], title: '開始計時', time: 300 }
```

通用計時器 HTML 結構：
```html
<div class="timer-wrap">
  <div class="timer-display" id="td-{id}">03:00</div>
  <div style="display:flex;gap:1rem">
    <button class="timer-btn" id="tb-{id}" onclick="toggleTimer('{id}', 180)">開始</button>
    <button class="timer-reset-btn" onclick="resetTimer('{id}', 180)">重設</button>
  </div>
</div>
```

---

### 5.5 內容展示版型

#### `vitamin-painkiller` — 維他命 vs 止痛藥
```js
{ type: 'vitamin-painkiller', chip: ['痛點測試', 'orange'], title: '維他命 vs 止痛藥' }
```

#### `hormozi-value` — Hormozi 價值公式
```js
{ type: 'hormozi-value', chip: ['價值公式', 'orange'], title: 'Alex Hormozi 的價值公式' }
```

#### `mece-intro-v2` — MECE 介紹
```js
{ type: 'mece-intro-v2', chip: ['MECE', 'yellow'], title: 'MECE：相互獨立，完全窮盡' }
```

#### `prototype-types` — 原型類型比較
```js
{ type: 'prototype-types', chip: ['原型類型', 'purple'], title: 'Prototype / MVP / MVC / POC 的差別' }
```

#### `pricing-full` — 定價策略
```js
{ type: 'pricing-full', chip: ['Pricing is Product', 'yellow'], title: '定價即產品' }
```

#### `pitch-combined` — Pitch 四要素
```js
{ type: 'pitch-combined', chip: ['Pitch 四要素', 'red'], title: 'Hook → Pain → Solution → CTA' }
```

#### `gtm-interactive-v2` — GTM 互動式
```js
{ type: 'gtm-interactive-v2', chip: ['Go-To-Market', 'blue'], title: '讓對的人看見對的訊息' }
```

#### `roadmap` — 路線圖
```js
{ type: 'roadmap', chip: ['下午路線圖', 'white'], title: '接下來兩小時' }
```

#### `resources` — 資源頁
```js
{ type: 'resources', chip: ['帶走這些', 'white'], title: '帶走這些' }
```

#### `close-carousel` — 結尾頁（含名言輪播）
```js
{ type: 'close-carousel', chip: ['Q&A', 'white'], cta: '掃描 QR Code 加 Vincent LinkedIn' }
```

---

## 6. Chip 標籤系統

```js
chip: ['標籤文字', '顏色', '大小(可選)']
// 顏色：orange / red / blue / purple / green / yellow / cyan / white
// 大小：不填 = 小, 'lg' = 大
```

```css
.chip { font-size: .62rem; font-weight: 800; letter-spacing: .14em; text-transform: uppercase;
        padding: .28em .85em; border-radius: 999px; border: 1px solid; }
.chip.chip-lg { font-size: .95rem; padding: .32em 1.1em; }
.chip-orange { color: var(--orange); border-color: rgba(255,107,0,.4); background: rgba(255,107,0,.1); }
/* 以此類推 red / blue / purple / green / yellow / cyan / white */
```

---

## 7. 動畫系統

### 7.1 頁面進場
每個 slide 進入視窗時觸發 `.is-visible`，其內元素逐一 fade-up：
```css
.slide-content > *:nth-child(1) { animation-delay: .04s; }
.slide-content > *:nth-child(2) { animation-delay: .10s; }
/* 每個子元素延遲 0.06s */
@keyframes fade-up {
  from { opacity: 0; transform: translateY(22px) scale(.98); }
  to   { opacity: 1; transform: none; }
}
```

### 7.2 背景光暈動畫（Aurora）
```css
@keyframes aurora {
  0%, 100% { background-position: 0% 50%; }
  50%       { background-position: 100% 50%; }
}
/* 在 blob() function 中使用：18s ease-in-out infinite */
```

### 7.3 互動元件過渡
```css
/* 卡片揭示 */
.card-mask { transition: opacity .4s ease, transform .4s var(--ease-back); }
.unlock-card.unlocked .card-mask { opacity: 0; transform: scale(.88); }

/* Accordion 展開 */
.step-back { transition: max-height .42s var(--ease-smooth), opacity .32s ease; }

/* 鎖定 bounce */
@keyframes lock-pulse {
  0%, 100% { transform: scale(1); }
  50%       { transform: scale(1.1); }
}
```

### 7.4 Section Rail 動畫
```css
@keyframes rburst { 0%{transform:scale(1)} 38%{transform:scale(1.45)} 68%{transform:scale(.92)} 100%{transform:scale(1)} }
@keyframes rring  { from{transform:scale(1);opacity:.75} to{transform:scale(2.6);opacity:0} }
```

---

## 8. 導覽系統

### 8.1 Scroll Snap
```css
#slides-container {
  height: 100vh;
  overflow-y: scroll;
  scroll-snap-type: y mandatory;
  scroll-behavior: smooth;
}
.slide { scroll-snap-align: start; height: 100vh; }
```

### 8.2 鍵盤快捷鍵（自動支援）
- `↓` / `Space` / `PageDown` → 下一頁
- `↑` / `PageUp` → 上一頁
- `Home` → 第一頁
- `End` → 最後一頁
- 數字鍵 `1–5` → 跳至對應 Section

### 8.3 底部 Section Nav
```js
const secStarts = { 1: 0, 2: 22, 3: 41, 4: 49, 5: 67 };
// 每個 section 的起始 slide index
```

### 8.4 跨頁面跳轉（xref）
```js
xrefs: [{ id: 's05', label: '創新三元' }]
// 在 slide 右下角產生跳轉按鈕
// 點擊後 scrollToSlide(targetIndex)
```

---

## 9. 互動 JS 模式

### 9.1 點擊揭示（unlock-card）
```js
onclick="this.classList.toggle('unlocked')"
// .unlocked → .card-mask opacity:0, .card-answer visibility:visible
```

### 9.2 計時器
```js
function toggleTimer(id, total) { /* 開始/暫停 */ }
function resetTimer(id, total)   { /* 重設 */ }
// HTML: id="td-{slideId}" (顯示), id="tb-{slideId}" (按鈕)
```

### 9.3 Accordion（step-item）
```js
function toggleStep(id) { document.getElementById(id).classList.toggle('open'); }
// .open → .step-back max-height:180px, opacity:1
```

### 9.4 Octalysis 展開
```js
function ocToggle(grpId, idx) { /* 展開/收合對應 drive */ }
```

### 9.5 Venn / DFV / Funnel 焦點
```js
function vennFocus(vid, key)   { /* 'des' | 'fea' | 'via' | 'all' */ }
function dfvFocus(vid, key)    { /* 'des' | 'fea' | 'via' */ }
function funnelFocus(fid, key) { /* 'p' | 'i' | 'h' */ }
```

---

## 10. Blob 背景函式

每個 slide 內容區通常有一個抽象的雙色光暈背景：
```js
function blob(c1, c2) {
  return `<div style="position:absolute;inset:0;pointer-events:none;z-index:0;
    background: radial-gradient(ellipse 80% 70% at 20% 20%, ${c1}, transparent 55%),
                radial-gradient(ellipse 70% 80% at 80% 80%, ${c2}, transparent 55%);
    animation: aurora 18s ease-in-out infinite;">
  </div>`;
}
// 使用範例：
h += blob('rgba(255,107,0,.08)', 'rgba(168,85,247,.06)');
```

顏色對應建議：
| Section | blob c1 | blob c2 |
|---|---|---|
| 策略思維 | `rgba(255,32,85,.08)` | `rgba(168,85,247,.07)` |
| 欲求性 | `rgba(59,130,246,.08)` | `rgba(255,32,85,.06)` |
| 可行性 | `rgba(34,197,94,.08)` | `rgba(6,182,212,.06)` |
| 存續性 | `rgba(251,191,36,.09)` | `rgba(168,85,247,.06)` |
| 成長思維 | `rgba(168,85,247,.1)` | `rgba(255,107,0,.07)` |

---

## 11. 快速製作新 Slide 的流程

1. **決定版型** — 參考 Section 5 清單，選最接近你需求的 `type`
2. **加進 D[] 陣列** — 照格式填 `id`, `sec`, `sub`, `type`, `chip`, `title`
3. **設定 theme** — 若是練習頁或轉場，加 `theme: 'warm'` 或 `'light'`
4. **加 xrefs** — 若需要跨頁跳轉，加 `xrefs: [{id:'...', label:'...'}]`
5. **更新 secStarts** — 若調整了頁面順序，同步更新每個 section 的起始 index

---

## 12. 製作新版本的最小 Checklist

- [ ] `D[]` 陣列中所有 `id` 唯一
- [ ] `sec` 值對應存在的 section（1–5）
- [ ] `secStarts` 的 index 與實際 D[] 位置一致
- [ ] 底部 `sec-nav` 的按鈕數量和文字與實際 section 一致
- [ ] 有計時器的 slide 都有 `time` 欄位（秒數）
- [ ] Light theme slide 都有加 `theme` 欄位
- [ ] 每個 section 至少有一個 `section-cover` + 一個 `section`（收尾）

---

## 13. 常用 CSS 元件片段

### Bar（橙紅漸層分隔線）
```css
.bar { width: 2.8rem; height: 3px; background: linear-gradient(90deg, var(--orange), var(--red)); border-radius: 2px; margin: .6rem 0 1rem; }
```

### Tip Box（左邊框提示框）
```css
.tip-box { padding: .8rem 1.2rem; border-left: 3px solid var(--orange); border-radius: 0 8px 8px 0; background: rgba(255,107,0,.08); font-size: var(--min-font); }
```

### Deck List（▸ 要點列表）
```css
.deck-list li { padding-left: 1.5rem; position: relative; }
.deck-list li::before { content: '▸'; position: absolute; left: 0; color: var(--orange); font-weight: 800; }
```

### Xref Link（跨頁跳轉按鈕）
```css
.xref-link { font-size: .58rem; font-weight: 700; padding: .2em .65em; border-radius: 5px;
             border: 1px solid rgba(255,107,0,.35); color: rgba(255,170,80,.9);
             background: rgba(255,107,0,.07); cursor: pointer; }
```

---

> **核心原則**：每個 slide 的目的只有一個。不要在一頁裡放超過一個核心訊息。版型選擇服務訊息，不是反過來。
