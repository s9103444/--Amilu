# Amilu Spec

Amilu 是專為台灣 CG 市場打造的雙向委託平台。這個資料夾是給工程師看的規格文件：規則以這裡為準，圖放在 Figma／FigJam，這裡只放連結。

---

## 文件清單

| 檔案                                     | 回答什麼問題                                           | 狀態      |
| ---------------------------------------- | ------------------------------------------------------ | --------- |
| [prd.md](prd.md)                         | 為什麼做、給誰用、MVP 做到哪、哪些不做                 | ✅        |
| [features.md](features.md)               | 每個功能怎麼運作（依五大模組）                         | ✅        |
| [order-lifecycle.md](order-lifecycle.md) | 訂單狀態怎麼變、誰觸發、各種倒數天數、金額與結算怎麼算 | ✅        |
| [data-model.md](data-model.md)           | 有哪些資料表、欄位、表之間的關係                       | ✅        |
| [design-spec.md](design-spec.md)         | 色彩、字體、字級、斷點與 Grid                          | ✅        |
| [api.md](api.md)                         | 前後端怎麼溝通                                         | ⬜ 草案已列 |
| [acceptance.md](acceptance.md)           | 怎樣算做完（拿來寫 E2E 測試）                          | ✅        |
| [changelog.md](changelog.md)             | 規則改過什麼、什麼時候改的                             | ✅        |

**第一次看建議順序**：prd → features → order-lifecycle → data-model。

---

## 外部連結

| 內容            | 位置                                                                                                  |
| --------------- | ----------------------------------------------------------------------------------------------------- |
| 畫面（ui flow） | [Figma ui flow 頁](https://www.figma.com/design/RkN83Jr1KTUExyKkfOjXyn/Amilu---Web-UI?node-id=352-86) |
| 設計規範        | Figma 同一檔案的 `design spec` 頁                                                                     |
| 資料表關係圖    | [FigJam ER](https://www.figma.com/board/KNj96DaqlW6fQ4NuWFu52b)                                       |
| 使用者流程圖    | `../討論紀錄/Excalidraw/flowChart圖形放置區/`（目前需用 Obsidian 開啟，之後匯出成 SVG）               |

---

## 用詞對照

| 文件用詞     | 意思                                   | 程式／資料表用詞  |
| ------------ | -------------------------------------- | ----------------- |
| 甲方、委託人 | 下單委託的人                           | commissioner      |
| 乙方、創作者 | 接案畫圖的人                           | creator           |
| 一般作品     | 作品集裡的圖文，不能直接下單           | `WORK`            |
| 可委託項目   | 創作者上架、可以下單的項目             | `COMMISSION_ITEM` |
| Amilu 燈     | 創作者目前是否接案（開放中／暫不接案） | —                 |

---

## 團隊分工

| 項目           | 負責人 | 技術                        |
| -------------- | ------ | --------------------------- |
| 後端           | 淳     | Python                      |
| 前端           | 彥廷   | TypeScript／Vue3            |
| 訂單狀態機設計 | 彥廷   | 完成後與後端對一次 API 規格 |
| E2E 測試       | 待定   | Playwright                  |

---

## 修改規則

1. 規則有變動時，直接改對應的 spec 檔，並在 `changelog.md` 記一筆。
2. 畫面改 Figma、資料表改 FigJam 後，同步更新 spec 裡的文字。
3. 同一條規則只寫在一個檔案裡，其他地方用連結指過去。
