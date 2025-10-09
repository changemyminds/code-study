# 📖 專注閱讀模式功能

優先級：**P2** | 預計時間：**1-2 天**

## 功能概述

為了提升使用者閱讀筆記的體驗,實作一個「專注閱讀模式」功能。使用者可以透過右下角的浮動按鈕或快捷鍵 (F/R) 切換模式,進入專注閱讀時:

- ✅ **左側選單完全隱藏** - 提供更寬廣的閱讀空間
- ✅ **內容區域置中且最佳寬度** - 限制在約 850px,符合最佳閱讀體驗
- ✅ **右側 TOC 懸浮顯示** - 滑鼠移到右側邊緣時滑入,不干擾閱讀
- ✅ **狀態保存** - 使用 localStorage 記住使用者偏好
- ✅ **快捷鍵支援** - 按 F 或 R 鍵快速切換

## 實作狀態

**狀態**: ✅ 已完成並優化 (v2.0 - 2025-10-09)

## 更新日誌

### v2.0 (2025-10-09) - 優化版本
**新增功能:**
- ✅ 移除按鈕文字標籤,改為純圖示圓形按鈕 (更簡潔)
- ✅ 新增 Header 自動隱藏功能 (向下滾動隱藏,向上滾動顯示)
- ✅ 內容寬度從 850px 增加到 1200px (提供更大閱讀空間)
- ✅ 新增滾動方向偵測 composable (`useScrollDirection`)
- ✅ 優化按鈕動畫效果 (激活時旋轉 180 度)

**技術改進:**
- 圓形按鈕設計 (48px × 48px)
- 圖示放大到 28px
- 使用 `requestAnimationFrame` 優化滾動偵測效能
- Header 使用 `transform: translateY()` 實現流暢隱藏動畫

### v1.0 (2025-10-09) - 初始版本
- ✅ 基礎閱讀模式功能
- ✅ 左側選單隱藏
- ✅ 右側 TOC 懸浮
- ✅ 快捷鍵支援 (F/R 鍵)
- ✅ localStorage 狀態保存

## 技術實作

### 1. 閱讀模式 Composable

**檔案**: `composables/useReadingMode.ts`

提供全域的閱讀模式狀態管理:

```typescript
export const useReadingMode = () => {
  // 使用 useState 讓狀態在整個應用中共享
  const isReadingMode = useState<boolean>('reading-mode', () => {
    // 從 localStorage 讀取使用者偏好
    if (process.client) {
      const saved = localStorage.getItem('reading-mode')
      return saved === 'true'
    }
    return false
  })

  // 切換、啟用、停用閱讀模式的函數
  const toggleReadingMode = () => { ... }
  const enableReadingMode = () => { ... }
  const disableReadingMode = () => { ... }

  // 快捷鍵支援 (F 或 R 鍵)
  // ...

  return {
    isReadingMode: readonly(isReadingMode),
    toggleReadingMode,
    enableReadingMode,
    disableReadingMode,
  }
}
```

**特點**:
- 使用 `useState` 確保狀態在整個應用中共享
- 使用 `localStorage` 保存使用者偏好
- 自動設置快捷鍵監聽 (F 或 R 鍵)
- 避免在輸入框中誤觸發快捷鍵

### 2. 滾動方向偵測 Composable (v2.0 新增)

**檔案**: `composables/useScrollDirection.ts`

偵測使用者滾動方向,用於實現 Header 自動隱藏:

```typescript
export const useScrollDirection = () => {
  const scrollDirection = useState<'up' | 'down' | null>('scroll-direction', () => null)
  const isAtTop = useState<boolean>('is-at-top', () => true)

  // 使用 requestAnimationFrame 優化效能
  // 滾動超過 5px 才觸發,避免微小抖動
  // 滾動超過 100px 才開始隱藏 Header

  return {
    scrollDirection: readonly(scrollDirection),
    isAtTop: readonly(isAtTop),
  }
}
```

**特點**:
- 使用 `requestAnimationFrame` 優化效能
- 設置 5px 的抖動容忍度
- 滾動超過 100px 才觸發 Header 隱藏
- 在頂部時始終顯示 Header

### 3. 閱讀模式切換按鈕

**檔案**: `components/ReadingModeToggle.vue`

固定在右下角的浮動按鈕:

**特點 (v2.0 優化)**:
- **純圖示設計**: 移除文字標籤,僅顯示圖示
- **圓形按鈕**: 48px × 48px 圓形設計
- **圖示放大**: 從 20px 增加到 28px (更明顯)
- **旋轉動畫**: 激活時旋轉 180 度
- **圖示切換**: 書本 📖 (未激活) / 眼睛關閉 👁️‍🗨️ (激活)
- **hover 效果**: 向上浮起 4px + 縮放 1.05 倍
- **半透明背景**: backdrop-filter 模糊效果

**樣式變化 (v1.0 → v2.0)**:
```css
// v1.0 - 長方形按鈕帶文字
width: auto, height: auto
icon: 20px

// v2.0 - 圓形純圖示按鈕
width: 48px, height: 48px
borderRadius: full (圓形)
icon: 28px
transform: rotate(180deg) when active
```

### 4. Header 自動隱藏 (v2.0 新增)

**檔案**: `components/AppHeader.vue`

Header 根據滾動方向自動隱藏/顯示:

```typescript
const shouldHideHeader = computed(() => {
  // 只有在閱讀模式下才隱藏
  if (!isReadingMode.value) return false

  // 在頂部時始終顯示
  if (isAtTop.value) return false

  // 向下滾動時隱藏
  return scrollDirection.value === 'down'
})
```

**動畫效果**:
```css
header {
  transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);

  &.header-hidden {
    transform: translateY(-100%);
  }
}
```

**特點**:
- 使用 GPU 加速的 `transform` 實現流暢動畫
- 在頁面頂部時始終顯示
- 僅在閱讀模式下啟用
- 向上滾動立即顯示

### 5. DocsPageLayout 組件修改

**檔案**: `components/DocsPageLayout.vue`

修改佈局以支援閱讀模式:

#### 修改內容

1. **引入 composable**:
```typescript
const { isReadingMode } = useReadingMode()
```

2. **添加 CSS 類別**:
```vue
<Container
  :class="{
    // ...其他類別
    'reading-mode': isReadingMode,
  }"
>
```

3. **添加切換按鈕**:
```vue
<!-- Reading Mode Toggle -->
<ReadingModeToggle />
```

#### 樣式調整

**左側選單** (`.aside-nav`):
```css
.reading-mode && {
  transform: translateX(-100%);  // 向左移出畫面
  opacity: 0;                     // 淡出
  pointer-events: none;           // 禁用互動
}
```

**內容區域** (`.page-body`) - v2.0 更新:
```css
.reading-mode && {
  max-width: 1200px;  // v2.0: 從 850px 增加到 1200px
  mx: auto;           // 置中顯示
  px: 3rem;           // 增加左右內距

  // 圖片/表格/程式碼可以完整顯示
  img, pre, table {
    max-width: 100%;
  }
}
```

**右側 TOC** (`.toc`):
```css
.reading-mode && {
  position: fixed;                          // 固定位置
  right: 0;                                 // 靠右
  transform: translateX(calc(100% - 20px)); // 大部分隱藏,僅露出 20px
  opacity: 0.6;                             // 半透明
  z-index: 40;                              // 高層級

  &:hover {
    transform: translateX(0);  // hover 時完全顯示
    opacity: 1;                // 完全不透明
  }
}
```

## 使用方式

### 切換閱讀模式

1. **點擊按鈕**: 點擊右下角的浮動按鈕
2. **快捷鍵**: 按 `F` 或 `R` 鍵

### 查看 TOC

在閱讀模式下:
1. 將滑鼠移到畫面右側邊緣
2. TOC 會從右側滑入顯示
3. 移開滑鼠後會自動隱藏

## 設計考量

### 最佳閱讀寬度

根據研究,最佳的閱讀寬度為:
- 每行 **50-75 個字元** (約 600-900px)
- 我們選擇 **850px** 作為最大寬度
- 這樣可以減少眼球移動,提升閱讀舒適度

### 為什麼不完全隱藏 TOC?

1. **快速導航**: 使用者可能隨時需要跳轉章節
2. **視覺提示**: 20px 的露出提醒使用者 TOC 的存在
3. **平滑體驗**: hover 顯示比點擊切換更流暢

### 狀態保存

使用 `localStorage` 保存使用者偏好:
- 使用者切換模式後,下次訪問會自動套用
- 跨頁面保持狀態 (切換章節時不會重置)

## 響應式設計

### 桌面版 (≥1024px)
- 完整功能
- 左側選單隱藏
- 內容置中限寬
- TOC 懸浮顯示

### 平板/手機版 (<1024px)
- 按鈕文字隱藏 (僅顯示圖示)
- TOC 維持原本的展開/收合邏輯
- 左側選單本來就是隱藏的 (移動版預設行為)

## 瀏覽器支援

- ✅ Chrome/Edge (最新版)
- ✅ Firefox (最新版)
- ✅ Safari (最新版)
- ✅ 支援 `localStorage` 的所有現代瀏覽器

## 效能考量

1. **CSS Transitions**: 使用 GPU 加速的 `transform` 和 `opacity`
2. **事件委託**: 快捷鍵事件僅綁定一次在 window
3. **狀態共享**: 使用 `useState` 避免多個組件重複邏輯
4. **延遲執行**: 僅在客戶端執行 localStorage 操作

## 未來優化方向

### 可選功能

1. **Header 自動隱藏**
   - 向下滾動時隱藏 Header
   - 向上滾動時顯示
   - 提供更大的閱讀空間

2. **字體大小調整**
   - 在閱讀模式下提供字體大小調整
   - A- / A / A+ 按鈕

3. **主題切換**
   - 閱讀模式專用的配色 (如米色背景)
   - 夜間閱讀模式

4. **更多快捷鍵**
   - `Esc` 退出閱讀模式
   - `←` / `→` 切換上/下一章節

### 效能優化

1. **動畫優化**
   - 使用 `will-change` 提示瀏覽器
   - 減少 reflow/repaint

2. **惰性加載**
   - 閱讀模式下延遲加載 TOC

## 測試檢查清單

- [x] 點擊按鈕可切換閱讀模式
- [x] 按 F 鍵可切換閱讀模式
- [x] 按 R 鍵可切換閱讀模式
- [x] 在輸入框中按 F/R 不會觸發切換
- [x] 左側選單在閱讀模式下隱藏
- [x] 內容區域在閱讀模式下置中限寬
- [x] TOC 在閱讀模式下變為懸浮模式
- [x] hover TOC 區域時完全顯示
- [x] 狀態保存到 localStorage
- [x] 重新載入頁面後狀態保持
- [x] 切換頁面後狀態保持
- [x] 響應式設計在手機上正常運作
- [x] 深色模式下樣式正常
- [x] 過渡動畫流暢

## 相關檔案

- [composables/useReadingMode.ts](../composables/useReadingMode.ts) - 狀態管理
- [components/ReadingModeToggle.vue](../components/ReadingModeToggle.vue) - 切換按鈕
- [components/DocsPageLayout.vue](../components/DocsPageLayout.vue) - 佈局組件

## 參考資料

- [Web Typography: The Elements of Typographic Style](http://webtypography.net/2.1.2) - 最佳行寬
- [Material Design: Reading](https://material.io/design/communication/writing.html) - 閱讀體驗設計
- [Nuxt 3 State Management](https://nuxt.com/docs/getting-started/state-management) - useState 用法
- [Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API) - localStorage 使用

---

**完成日期**: 2025-10-09
**實作者**: Claude Code
**優先級**: 🎯 中高
**影響範圍**: 使用者體驗 + 閱讀舒適度
