# Amilu Design Spec

> 來源：[Figma `design spec` 頁](https://www.figma.com/design/RkN83Jr1KTUExyKkfOjXyn/Amilu---Web-UI?node-id=480-2)（2026/10/1 建立）。數值以 Figma 的 variable 與 style 為準，這份是給工程師查的文字版。
> 前端開始實作後，token 會變成 CSS 變數檔；到時這份只保留規則與「token 定義在哪個檔案」，色碼表刪掉，避免兩邊各自維護。

**調性**：灰白黑極簡、零阻力瀏覽、以圖片內容為優先。

---

## 1. 色彩

Figma variable collection：「Amilu｜色彩」，共 14 色，分灰階、底色、強調色三組；另有 2 個漸層 paint style。專案只用這些顏色，不要另外寫色碼。

- 名稱後的「-數字」是同色的透明度版本，例如 `accent/orange-30` 是 `#EB5919` 透明度 30%。
- Google 登入按鈕的標誌色是 Google 規定的顏色，不屬於專案色票。

### 1.1 灰階

| Token | 色碼 | 用途 |
| --- | --- | --- |
| `neutral/900` | `#232020` | 主要文字、主按鈕、icon |
| `neutral/900-50` | `#232020`，透明度 50% | 圖片上的移除按鈕、標記、手機版遮罩 |
| `neutral/500` | `#959595` | 次要文字、說明、次要 icon |
| `neutral/300` | `#C2C2C2` | 邊框、分隔線、placeholder |
| `neutral/white` | `#FFFFFF` | 頁面底、深底上的文字 |

### 1.2 底色

| Token | 色碼 | 用途 |
| --- | --- | --- |
| `base/100` | `#F7F7F7` | 淺色區塊底、提示框 |
| `base/200` | `#EBE9E6` | 主背景、卡片底、頁籤 |
| `base/500` | `#B7AEA3` | 次要強調、分隔、標籤 |

### 1.3 強調色

| Token | 色碼 | 用途 |
| --- | --- | --- |
| `accent/orange` | `#EB5919` | 錯誤、警示、必填、未讀紅點 |
| `accent/orange-10` | `#EB5919`，透明度 10% | 狀態標籤底、警語框底 |
| `accent/orange-30` | `#EB5919`，透明度 30% | 選項選取底、錯誤輸入框外框 |
| `accent/green` | `#78B260` | 成功提示、可商用 |
| `accent/green-30` | `#78B260`，透明度 30% | 狀態標籤底、已開通 |
| `accent/blue` | `#80CAEA` | 狀態標籤、資訊提示 |

### 1.4 漸層（paint style）

| 名稱 | 用途 | CSS |
| --- | --- | --- |
| `gradient/rainbow` | 按鈕 | 色標：`#80CAEA 0%, #78B260 20%, #EBBD00 40%, #F0809F 60%, #EB5919 80%` |
| `gradient/side-bg` | 首頁兩側背景 | `linear-gradient(270deg, rgba(235, 233, 230, 0) 0%, #EBE9E6 100%)` |

- `gradient/rainbow` 在 Figma 裡的漸層軸比按鈕高很多，所以按鈕上實際只看得到粉到橘那一段。CSS 角度與長度請對照 Figma 按鈕畫面調整，不要直接用色標從 0% 排到 80%。

---

## 2. 字體

中英文都用 **源泉圓體 GenSenRounded2 TC**。

**源泉圓體字重**：

| 字重 | 用在哪裡 |
| --- | --- |
| R（Regular 400） | 內文 |
| M（Medium 500） | 強調 |
| B（Bold 700） | 標題與按鈕 |
| H（Heavy） | Logo 字 |

---

## 3. 字級

- Figma text style 前綴 `Amilu/`，字級綁定「Amilu｜字級」variable。
- variable 有三個 mode：**Desktop 1440**、**Tablet 768**、**Mobile 375**，對應下方三欄。
- 行高用百分比，跟著字級縮放。

| Style | Desktop | Tablet | Mobile | 行高 | 字重 | 用途 |
| --- | --- | --- | --- | --- | --- | --- |
| `Amilu/Display` | 96px | 64px | 40px | 120% | Bold | 主視覺大標（首頁 Hero） |
| `Amilu/H1` | 56px | 44px | 32px | 130% | Bold | 頁面主標 |
| `Amilu/H2` | 36px | 30px | 24px | 130% | Bold | 區塊標題 |
| `Amilu/H3` | 28px | 24px | 20px | 140% | Bold | 卡片／燈箱標題 |
| `Amilu/Title` | 24px | 22px | 18px | 140% | Bold | 欄位標題、小標 |
| `Amilu/Body-L` | 22px | 20px | 16px | 160% | Regular | 強調內文、說明段落 |
| `Amilu/Body` | 20px | 18px | 16px | 160% | Regular | 一般內文 |
| `Amilu/Caption` | 16px | 14px | 14px | 150% | Regular | 輔助說明、提示文字 |
| `Amilu/Small` | 14px | 12px | 12px | 150% | Medium | 標籤、時間戳、極小字 |
| `Amilu/Button-md` | 22px | 20px | 16px | 100% | Bold | 主要按鈕 |
| `Amilu/Button-sm` | 18px | 16px | 14px | 100% | Bold | 次要／小按鈕 |

---

## 4. Grid 與斷點

| 裝置 | 斷點 | 設計基準寬 | 欄數 | Gutter | Margin |
| --- | --- | --- | --- | --- | --- |
| 桌機 | ≥ 1024 | 1440 | 12 | 24 | 48 |
| 平板 | 768–1023 | 768 | 12 | 16 | 32 |
| 手機 | < 768 | 375 | 12 | 16 | 16 |

- 間距一律用 8 的倍數。
- **作品牆**以元件實作，欄數：桌機 5／平板 3／手機 2。
- Figma grid style 前綴 `Amilu/Grid/`。

