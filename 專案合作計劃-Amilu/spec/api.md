# Amilu API 規格

> 這份寫「前端要呼叫哪些 API、對應哪個畫面、對應哪張資料表」。
> **現況：前端（彥廷）先列的草案，尚未跟後端（淳）對過規格**，所以不寫完整 request/response 欄位（那部分留到對規格時一起定），也不重複寫訂單狀態與金額規則，規則一律連到 [order-lifecycle.md](order-lifecycle.md)。
> 對完規格後，把本文件第 5 節「待確認事項」逐項勾掉、補上真正的請求／回應格式，再回 [README.md](README.md) 把這份狀態改成 ✅。

---

## 0. 怎麼讀這份文件

- 第 2 節依 [features.md](features.md) 的五大模組列出 API，每個模組一張表：`方法與路徑｜用途｜對應畫面｜對應資料表`。
- 路徑與欄位命名對齊 [data-model.md](data-model.md) 的英文表名／snake_case 欄位名（例如 `ORDER`→`/orders`），避免前後端對規格時文字對不上。
- 用詞對照 [README.md](README.md#用詞對照)：甲方／委託人＝`commissioner`，乙方／創作者＝`creator`。
- 第 3 節特別列出「即時計算、不落地存值」的欄位要靠哪些 API 取得（見 [data-model.md 5.1](data-model.md#51-哪些值刻意不存)）。
- 第 5 節是待確認事項，寫成可以直接拿去跟淳開會討論的議程格式。

---

## 1. 共用慣例（後端框架未定案，以下為暫定假設，待確認）

| 項目 | 暫定假設 | 狀態 |
| --- | --- | --- |
| Base path | `/api/v1/` | 待確認（要不要做版本號） |
| 格式 | Request／Response 皆為 JSON | 待確認 |
| 認證方式 | 未定（session cookie／JWT 皆可能） | 待確認，見 [5.1](#51-後端框架與基礎慣例未定案) |
| 錯誤格式 | 未定，暫定 `{ "error": { "code": "...", "message": "..." } }` | 待確認 |
| 分頁 | 未定，列表類 API 先假設 `page`＋`page_size` | 待確認 |
| 時間格式 | ISO 8601（`datetime`／`date` 對齊 FigJam ER 型別） | 可直接採用 |
| 金額單位 | 新台幣整數（元），不處理小數 | 可直接採用（配合 [order-lifecycle.md 三](order-lifecycle.md#三金額與結算公式)無條件捨去規則）|

---

## 2. API 清單（依五大模組）

### 2.1 模組一｜帳號與身份

| 方法與路徑 | 用途 | 對應畫面 | 對應資料表 |
| --- | --- | --- | --- |
| `POST /auth/register` | 帳密註冊（帳號 ID、暱稱、email、密碼） | [1.1 註冊與登入](features.md#11-註冊與登入) | `USER` |
| `POST /auth/login` | 帳密登入 | [1.1](features.md#11-註冊與登入) | `USER` |
| `POST /auth/google` | Google 快捷登入 | [1.1](features.md#11-註冊與登入) | `USER` |
| `POST /auth/google/complete-profile` | Google 補填資料（帳號 ID、暱稱、大頭照、同意條款） | [1.1 Google 補填資料頁](features.md#11-註冊與登入) | `USER` |
| `POST /auth/logout` | 登出 | — | — |
| `POST /phone-verifications` | 發送手機驗證碼 | [1.2 手機驗證](features.md#12-手機驗證) | `USER` |
| `POST /phone-verifications/confirm` | 確認驗證碼 | [1.2](features.md#12-手機驗證) | `USER` |
| `GET /me` | 取得目前使用者（含 `CREATOR_PROFILE.store_approved` 判斷是否創作者） | [1.3 身份](features.md#13-身份) | `USER`、`CREATOR_PROFILE` |

### 2.2 模組二｜探索與瀏覽

| 方法與路徑 | 用途 | 對應畫面 | 對應資料表 |
| --- | --- | --- | --- |
| `GET /works` | 作品牆（一般作品，支援分類／用途／標籤／關鍵字篩選＋分頁） | [2.1 首頁作品牆](features.md#21-首頁作品牆)、[2.2 搜尋與篩選](features.md#22-搜尋與篩選) | `WORK` |
| `GET /commission-items` | 可委託項目列表（同上篩選，依 Amilu 燈自動過濾） | [2.1](features.md#21-首頁作品牆)、[2.2](features.md#22-搜尋與篩選) | `COMMISSION_ITEM` |
| `GET /works/{id}` | 作品詳情（作品展示燈箱） | [2.5 作品展示燈箱](features.md#25-作品展示燈箱) | `WORK`、`WORK_IMAGE` |
| `GET /creators/{creator_id}/commission-items` | 單一創作者的委託商店內容 | [2.6 委託商店燈箱](features.md#26-委託商店燈箱) | `COMMISSION_ITEM` |
| `GET /commission-items/{id}` | 委託項目詳情 | [2.6](features.md#26-委託商店燈箱) | `COMMISSION_ITEM`、`COMMISSION_ITEM_IMAGE` |
| `GET /users/{id}` | 創作者個人頁基本資料 | [2.7 創作者個人頁](features.md#27-創作者個人頁) | `USER`、`CREATOR_PROFILE` |
| `GET /users/{id}/works` | 個人頁「作品」分頁籤 | [2.7](features.md#27-創作者個人頁) | `WORK` |
| `GET /users/{id}/commission-items` | 個人頁「可委託項目」分頁籤 | [2.7](features.md#27-創作者個人頁) | `COMMISSION_ITEM` |
| `POST /follows` | 追蹤 | [2.8 追蹤](features.md#28-追蹤) | `FOLLOW` |
| `DELETE /follows/{following_id}` | 取消追蹤 | [2.8](features.md#28-追蹤) | `FOLLOW` |
| `GET /users/{id}/followers` | 追蹤清單燈箱（followers 頁籤） | [2.8](features.md#28-追蹤) | `FOLLOW` |
| `GET /users/{id}/following` | 追蹤清單燈箱（following 頁籤） | [2.8](features.md#28-追蹤) | `FOLLOW` |
| `GET /categories` | 作品類型查找表 | [0 共用規則](features.md#0-共用規則分類與標籤) | `CATEGORY` |
| `GET /purposes` | 用途查找表 | [0](features.md#0-共用規則分類與標籤) | `PURPOSE` |
| `GET /tags?q=` | 自訂標籤自動完成／查詢（去空白、不分大小寫比對規則見 [5.8](#58-標籤比對去空白英文不分大小寫沿用既有標籤由前端查詢還是後端處理)） | [0](features.md#0-共用規則分類與標籤) | `TAG` |

### 2.3 模組三｜個人中心

| 方法與路徑 | 用途 | 對應畫面 | 對應資料表 |
| --- | --- | --- | --- |
| `GET /me/profile` | 個人資料讀取 | [3.1 個人資料設定](features.md#31-個人資料設定) | `USER` |
| `PATCH /me/profile` | 修改大頭照／暱稱／簡介 | [3.1](features.md#31-個人資料設定) | `USER` |
| `DELETE /me` | 刪除帳號（軟刪除，條件見畫面規則） | [3.1](features.md#31-個人資料設定) | `USER` |
| `POST /works` | 上傳一般作品 | [3.2 上傳一般作品](features.md#32-上傳一般作品) | `WORK`、`WORK_IMAGE`、`WORK_TAG` |
| `POST /commission-items` | 上架可委託項目 | [3.3 上傳可委託項目](features.md#33-上傳可委託項目) | `COMMISSION_ITEM`、`COMMISSION_ITEM_IMAGE` |
| `PATCH /commission-items/{id}` | 編輯可委託項目 | [3.3](features.md#33-上傳可委託項目) | `COMMISSION_ITEM` |
| `DELETE /commission-items/{id}` | 刪除可委託項目 | [3.3](features.md#33-上傳可委託項目) | `COMMISSION_ITEM` |
| `GET /me/creator-onboarding` | 開通創作者商店三步驟目前完成狀態 | [3.4 開通創作者商店](features.md#34-開通創作者商店) | `CREATOR_PROFILE` |
| `POST /me/creator-onboarding/confirm` | 確認開通（三步驟皆完成後） | [3.4](features.md#34-開通創作者商店) | `CREATOR_PROFILE` |
| `GET /me/creator-settings` | 接案設定讀取 | [3.5 接案設定](features.md#35-接案設定) | `CREATOR_PROFILE` |
| `PATCH /me/creator-settings` | 接案設定修改（收款帳戶、訂金比例、修改次數） | [3.5](features.md#35-接案設定) | `CREATOR_PROFILE` |
| `PATCH /me/creator-settings/amilu-light` | Amilu 燈開關切換 | [3.6 Amilu 燈](features.md#36-amilu-燈) | `CREATOR_PROFILE` |
| `GET /orders?role={commissioner\|creator}&phase={ongoing\|history}` | 約稿管理列表（已委託／已接案，進行中／歷史） | [3.7 約稿管理](features.md#37-約稿管理) | `ORDER` |

### 2.4 模組四｜委託流程與協作

| 方法與路徑 | 用途 | 對應畫面 | 對應資料表 |
| --- | --- | --- | --- |
| `POST /orders` | 送出委託申請 | [4.1 委託申請頁](features.md#41-委託申請頁委託人填寫) | `ORDER`、`ORDER_REFERENCE_IMAGE` |
| `POST /orders/{id}/quote` | 創作者報價（金額、起訖日、各階段 deadline） | [4.2 報價](features.md#42-報價創作者填寫) | `ORDER` |
| `POST /orders/{id}/confirm-quote` | 委託人確認報價，進入付款流程 | [4.3 付款頁](features.md#43-付款頁委託人) | `ORDER` |
| `GET /orders/{id}/payment` | 取得付款資訊（虛擬帳號、銀行代碼、期限） | [4.3](features.md#43-付款頁委託人) | `PAYMENT` |
| `POST /orders/{id}/payment/plan` | 選擇付款方案（先付訂金／一次付清） | [4.3](features.md#43-付款頁委託人) | `PAYMENT`、`ORDER.payment_plan` |
| `POST /orders/{id}/payment/notify-paid` | 「通知已付款」 | [4.3](features.md#43-付款頁委託人) | `PAYMENT` |
| `POST /orders/{id}/payment/simulate-paid` | 「模擬入帳」（demo，契約成立） | [4.3](features.md#43-付款頁委託人) | `PAYMENT`、`ORDER` |
| `GET /orders/{id}` | 訂單詳情（契約快照、目前階段、結算用欄位，見[第 3 節](#3-金額結算類特殊-api)） | [4.4 委託討論牆](features.md#44-委託討論牆) | `ORDER` |
| `GET /orders/{id}/stages` | 階段清單與各階段狀態、金額（即時計算） | [4.4](features.md#44-委託討論牆) | `ORDER_STAGE`、`ORDER_STAGE_IMAGE` |
| `POST /orders/{id}/stages/{stage_no}/submissions` | 創作者上傳／重傳本階段稿件（整組覆蓋，見 [5.2](#52-圖片上傳機制未定案)） | [4.4 上傳作品燈箱](features.md#44-委託討論牆) | `ORDER_STAGE_IMAGE`、`ORDER_STAGE.submitted_at` |
| `POST /orders/{id}/stages/{stage_no}/confirm` | 委託人確認該階段審核 | [4.4](features.md#44-委託討論牆) | `ORDER_STAGE` |
| `GET /orders/{id}/topup` | 補款資訊（金額即時計算、期限） | [4.4 補款頁](features.md#44-委託討論牆) | `PAYMENT` |
| `POST /orders/{id}/topup/notify-paid` | 「通知已補款」 | [4.4](features.md#44-委託討論牆) | `PAYMENT` |
| `POST /orders/{id}/topup/simulate-paid` | 「模擬入帳」（demo，視同確認最後階段並結案） | [4.4](features.md#44-委託討論牆) | `PAYMENT`、`ORDER` |
| `GET /orders/{id}/messages` | 聊天室訊息列表 | [4.7 聊天室](features.md#47-聊天室) | `MESSAGE` |
| `POST /orders/{id}/messages` | 傳送訊息（文字／圖片） | [4.7](features.md#47-聊天室) | `MESSAGE` |
| `POST /orders/{id}/cancel` | 取消委託（契約成立前） | [4.8 取消委託燈箱](features.md#48-取消委託燈箱) | `ORDER.cancel_reason` |
| `GET /orders/{id}/termination-preview` | 終止合作結算試算（三個固定欄位，即時計算） | [4.9 終止合作燈箱](features.md#49-終止合作燈箱) | 見[第 3 節](#3-金額結算類特殊-api) |
| `POST /orders/{id}/terminate` | 申請終止合作（二次確認後送出，立即生效） | [4.9](features.md#49-終止合作燈箱) | `ORDER.terminated_by_id`／`terminated_at` |
| `POST /orders/{id}/ratings` | 雙向評價（1～5 星） | [4.5 結案畫面](features.md#45-結案畫面)、[5.1 評價](features.md#51-評價) | `ORDER.rating_for_creator`／`rating_for_commissioner` |

### 2.5 模組五｜通知與評價

| 方法與路徑 | 用途 | 對應畫面 | 對應資料表 |
| --- | --- | --- | --- |
| `GET /notifications` | 通知列表（專案通知／系統公告） | [5.2 通知中心「通知」頁籤](features.md#52-通知中心) | `NOTIFICATION` |
| `POST /notifications/mark-all-read` | 全部標為已讀 | [5.2](features.md#52-通知中心) | `NOTIFICATION` |
| `GET /notifications/unread-count` | 未讀數（header 鈴鐺紅點、頁籤旁數字） | [5.2](features.md#52-通知中心) | `NOTIFICATION` |
| `GET /conversations` | 對話頁籤列表（各案件最新一則訊息預覽） | [5.2 通知中心「對話」頁籤](features.md#52-通知中心) | `MESSAGE` |

---

## 3. 金額／結算類特殊 API

[data-model.md 5.1](data-model.md#51-哪些值刻意不存) 明確列出這些值刻意不存欄位、全部即時計算，所以前端**不能自己算，一定要靠 API 拿到後端算好的數字**：

| 值 | 怎麼拿到 | 對應規則 |
| --- | --- | --- |
| 本階段金額 | `GET /orders/{id}/stages` 回應裡每個階段帶 `amount` 欄位。**這只是顯示用的試算金額，不代表撥款**——撥款只發生在三個時點：第一階段確認（撥訂金）、最後階段確認（撥尾款）、終止合作結算，中間階段確認不觸發撥款（2026/10/15 定案，見 [order-lifecycle.md 三｜付款方案](order-lifecycle.md#三金額與結算公式)）。`POST /orders/{id}/stages/{stage_no}/confirm` 只有在 `stage_no`＝1 或＝5（最後階段）時才會連帶產生 `PAYMENT` 已撥款紀錄，中間階段的 confirm 只改 `ORDER_STAGE` 確認狀態 | [order-lifecycle.md 三｜階段金額拆分、付款方案](order-lifecycle.md#三金額與結算公式) |
| 補款金額 | `GET /orders/{id}/topup` 回應 | `quote_amount` − `PAYMENT` 已託管入款加總，[data-model.md 4.4](data-model.md#44-order-結案方式判斷不另設欄位) |
| 終止合作結算試算（委託人已付／創作者可得／委託人退款或應補付） | `GET /orders/{id}/termination-preview` | [order-lifecycle.md 三｜終止合作結算](order-lifecycle.md#三金額與結算公式) |
| 目前進行到哪一階段 | `GET /orders/{id}/stages` 查哪一筆未確認 | [data-model.md 5.1](data-model.md#51-哪些值刻意不存) |
| 結案方式（已完成／已終止，是否雙輸） | `GET /orders/{id}` 回應需帶 `status`、`completed_at`、`terminated_by_id`、`terminated_at`，前端依此判斷 | [data-model.md 4.4](data-model.md#44-order-結案方式判斷不另設欄位) |
| followers／following 數字 | `GET /users/{id}` 回應帶計數，或 `GET /users/{id}/followers`／`following` 回應帶 `total` | [data-model.md 5.1](data-model.md#51-哪些值刻意不存) |

---

## 4. 跨模組規則

### 4.1 契約快照欄位

`GET /orders/{id}` 要回傳的快照欄位與各自鎖定時間點，詳細規則見 [order-lifecycle.md 五](order-lifecycle.md#五契約快照與同意)：

| 欄位 | 鎖定時間點 | 送出委託申請後／契約成立後回應是否不同 |
| --- | --- | --- |
| `item_title_snapshot`、`deposit_percent_snapshot`、`revision_count_snapshot`、`stage_plan_snapshot` | 甲方**送出委託申請時**（2026/10/15 定案，四個欄位都在這個時間點一起鎖，不是契約成立時，見 [data-model.md 4.1](data-model.md#41-order-契約快照欄位)） | 申請時就鎖定，申請後與契約成立後回應相同 |
| 授權層級 | 送出委託申請時 | 同上 |
| 報價金額、起訖日、各階段 `deadline` | 乙方報價時填寫，甲方確認報價時鎖定 | 報價前為空，確認後鎖定 |
| 其餘契約快照（表單填寫內容、付款方式等，不含上面四個欄位） | 平台確認入帳（契約成立）時 | 成立前後欄位內容不同，待確認是否要用同一個 API 回應區分「草稿版」與「鎖定版」，見 [5.1](#51-後端框架與基礎慣例未定案) |

### 4.2 權限規則（動作 → 允許呼叫者）

同一筆 `ORDER`，甲方（`commissioner_id`）與乙方（`creator_id`）能呼叫的動作不同：

| 動作 | 允許呼叫者 |
| --- | --- |
| `POST /orders`（送出申請） | 甲方 |
| `POST /orders/{id}/quote`（報價） | 乙方 |
| `POST /orders/{id}/confirm-quote`（確認報價） | 甲方 |
| `POST /orders/{id}/stages/{stage_no}/submissions`（上傳稿件） | 乙方 |
| `POST /orders/{id}/stages/{stage_no}/confirm`（確認階段） | 甲方 |
| `POST /orders/{id}/cancel`（取消委託） | 甲方或乙方皆可（契約成立前） |
| `POST /orders/{id}/terminate`（終止合作） | 甲方或乙方皆可（契約成立後） |
| `POST /orders/{id}/ratings`（評價） | 甲方或乙方皆可，雙向各自評對方 |

判斷方式待確認，見 [5.6](#56-同一帳號雙重身份時訂單類-api-怎麼判斷現在是哪一方)。

### 4.3 狀態機對應（API → 狀態轉移）

對照 [order-lifecycle.md 一｜ORDER.status](order-lifecycle.md#一委託狀態機)：

| API | 觸發的狀態轉移 |
| --- | --- |
| `POST /orders` | 建立，進入「待乙方報價」 |
| `POST /orders/{id}/quote` | 「待乙方報價」→「待甲方付款」 |
| `POST /orders/{id}/payment/simulate-paid` | 「待甲方付款」→「製作中」（契約成立） |
| `POST /orders/{id}/stages/{stage_no}/confirm`（最後階段）／`POST /orders/{id}/topup/simulate-paid` | 「製作中」→「已完成」 |
| `POST /orders/{id}/terminate` | 「製作中」→「已終止」 |
| `POST /orders/{id}/cancel` | 「待乙方報價」或「待甲方付款」→「已取消」 |
| 無對應 API（後端排程觸發） | 付款逾期自動取消、逾期強制結算、補款逾期結算，見 [5.4](#54-系統排程觸發的動作確認前端完全不需要呼叫-api) |

---

## 5. 待確認事項

以下每項都是跟淳對規格前要先討論的議程，寫清楚情境、選項與影響範圍，方便直接拿去開會用。

### 5.1 後端框架與基礎慣例未定案

- **情境**：淳還沒決定 Python 用哪個框架（FastAPI／Django REST Framework／Flask），第 1 節的錯誤格式、分頁、驗證方式都只是暫定假設。
- **要討論**：
  1. 錯誤回應格式：統一 `{error:{code, message}}` 這種前端好 parse 的格式，還是用框架預設（例如 DRF 預設格式）？
  2. 分頁：作品牆、約稿管理列表、通知列表這幾個 list API 要不要分頁？用 `page`＋`page_size` 還是 cursor？
  3. 驗證方式：session cookie 還是 JWT／token-based？Vue3 SPA 常見用 token，但要看淳對後端框架的熟悉度。
- **影響範圍**：幾乎所有 list／寫入類 API 的格式，建議第一次對規格會議就先定這三件事，其他 endpoint 才能照同一套慣例寫。

### 5.2 圖片上傳機制未定案

- **情境**：涉及的上傳情境——`USER.avatar_url`（單張）、`WORK_IMAGE`（1～20 張）、`COMMISSION_ITEM_IMAGE`（1～5 張）、`ORDER_STAGE_IMAGE`（每次 1～5 張）、`ORDER_REFERENCE_IMAGE`、聊天室 `MESSAGE.image_url`（單張）。
- **選項**：
  - **A．直傳後端**：前端 `multipart/form-data` 直接丟後端存檔、回網址。簡單，demo 流量小沒差；缺點是後端要處理檔案 I/O。
  - **B．presigned URL**：後端只發簽好的上傳網址，前端直傳雲端（S3-like），上傳完再呼叫後端「確認上傳完成」寫入資料庫。省後端流量，但要多串雲端 SDK，demo 規模可能過度設計。
- **特別要注意的規則**（見 [data-model.md 4.3](data-model.md#43-order_stage_image-上傳規則)）：`ORDER_STAGE_IMAGE` 同一階段重傳是「整組覆蓋」——刪掉該 `stage_id` 底下所有列再重寫、`seq_no` 從 1 重編。不管選 A 或 B，`POST /orders/{id}/stages/{stage_no}/submissions` 都要設計成「整組替換」而不是逐張新增；選 B 的話還要處理舊檔案要不要清掉（孤兒檔案）的問題。
- **影響範圍**：5 種上傳情境的 API 設計，以及重傳覆蓋語意要不要用同一個 endpoint（整組 PUT）還是先刪後 POST 兩步。

### 5.3 模擬金流按鈕要不要獨立 API

- **情境**：「模擬入帳」「通知已付款」「通知已補款」目前都是 demo 用的按鈕，效果是改 `PAYMENT.status`。
- **要討論**：這些要設計成一個「更新付款狀態」API 吃不同 action 參數，還是各自獨立 endpoint（目前第 2.4 節暫定列成獨立 endpoint，如 `POST /orders/{id}/payment/simulate-paid`）？
- **為什麼要先想清楚**：[prd.md](prd.md#mvp-範圍) 明確寫 v1.1 以後要串真實金流（綠界代收轉付），到時「模擬入帳」這個手動觸發的 API 要整個拔掉、換成金流服務的 webhook callback。語意清楚的獨立 endpoint 之後拔掉比較乾淨；混在同一個通用更新 API 裡，之後要拆會比較麻煩。

### 5.4 系統排程觸發的動作與測試用時間控制

> 這一項直接對應 [acceptance.md 第 5 節「還沒決定」#6](acceptance.md#5-待確認事項)：E2E-05、E2E-06、E2E-07 子情境 D 都標了「需時間控制」，acceptance.md 寫明「前端 Playwright clock 擋不住後端計時，需要後端提供測試用的時間控制，這點留給 api.md 決定」——這裡就是要決定的地方，對規格時務必跟淳定案。

- **情境**：以下動作都是後端排程／cron 觸發，不是前端按按鈕（見 [order-lifecycle.md 二｜計時規則](order-lifecycle.md#二計時規則)）：
  - 付款期限 3 天到期未入帳 → 自動取消（E2E-07 子情境 D）
  - 補款期限 7 天到期 → 進入 7 天等待期；等待期滿仍未補齊 → 「最後階段未完成」結算（E2E-05）
  - 整筆委託最終截稿日 +30 天仍未了結 → 系統強制結算（E2E-06）
  - 結算後委託人應補付未付 → 鎖帳號
- **前端完全不呼叫 API 觸發**：前端只需要「被動反應」——收到通知後重新 `GET /orders/{id}`，這點不用討論。要討論的是 Playwright 測試（以及 demo 簡報）要怎麼讓後端的 3／7／30 天提前發生：
  1. **假時間注入**：後端在測試／demo 環境額外接受一個「目前時間」覆寫（例如 header 或測試專用 API 設定 fake now），所有計時都以這個基準算，正式環境關閉。最貼近真實邏輯，但後端要多做一層時間抽象層，實作成本較高。
  2. **可調閾值**：把 3／7／30 天抽成環境變數／設定值，測試環境調小（例如 3 天→30 秒、30 天→5 分鐘），正式環境維持原值。改動小，但要注意「7 天等待期接著另一個 7 天」這種連續計時，調小後測試要等兩段。
  3. **demo-only 手動觸發 endpoint**：跳過「等時間流逝」，直接提供 `POST /orders/{id}/force-settle-debug` 之類的端點，直接把指定 ORDER 推進到「已逾期」「強制結算後」的最終狀態，上線前整段拔掉。實作最快，但驗證的是「結算邏輯對不對」，驗不到「真的等到期限才觸發」這件事本身。
  - **建議方向**：選項 2（可調閾值）＋選項 3（手動觸發）組合——可調閾值讓 E2E-05／E2E-06／E2E-07D 的「到期後發生什麼」有機會在合理測試時間內跑完整個流程；手動觸發則用在只想驗證結算金額、不想等任何時間的單元測試。選項 1 的真實度最高但目前团队規模（demo，兩人）不一定值得投入，先列出來讓淳評估實作成本後決定。
- **影響範圍**：`acceptance.md` E2E-05、E2E-06、E2E-07 子情境 D 這三條 P1／P0 測試能不能寫、怎麼寫；決定後要回 `acceptance.md` 把「需時間控制」改成具體操作方式，並把第 5 節的 #6 從「還沒決定」移到「已定案」。

### 5.5 聊天室與通知更新機制：輪詢頻率，還是要做推播

- **情境**：`MESSAGE`、`NOTIFICATION` 在 [features.md](features.md#47-聊天室) 都明確定義為「非即時」。
- **要討論**：前端用 `setInterval` 輪詢（例如每 10～30 秒打一次 `GET /notifications/unread-count`），還是要上 WebSocket／SSE？
- **傾向**：demo 規模輪詢應該夠用，但頻率要跟後端一起定（太頻繁增加後端負擔，太慢使用者會覺得沒更新）；順便確認「未讀數」這個輕量 API 要不要獨立於完整列表 API 之外，討論牆開著時可能一直輪詢，不該每次都把整個列表撈出來。

### 5.6 同一帳號雙重身份時，訂單類 API 怎麼判斷「現在是哪一方」

- **情境**：約稿管理「已委託」「已接案」兩個頁籤是同一個使用者，但一個是甲方視角一個是乙方視角；報價、確認階段、終止合作這些 API 的權限判斷都要分清楚呼叫者是 `ORDER.commissioner_id` 還是 `creator_id`。
- **要討論**：後端直接用 token 裡的 `user_id` 比對該筆 `ORDER` 的兩個欄位、依結果決定能呼叫哪些動作（例如報價只能乙方呼叫、確認階段只能甲方呼叫），前端不需要額外傳「視角」參數——這樣最不容易出錯，但要跟淳確認後端是否也這麼打算。
- **影響範圍**：[4.2 權限規則](#42-權限規則動作-→-允許呼叫者)整張表的實際判斷邏輯，對規格時逐條跟淳確認。

### 5.7 防自刷（同帳號不能委託自己）由後端擋，前端只是提示

- **情境**：[features.md 1.3](features.md#13-身份)「委託人與創作者是同一個帳號時，擋下送出委託」。
- **要討論**：`POST /orders` 要回什麼錯誤碼／訊息，前端才能顯示對應提示（而不是顯示通用錯誤）。這個檢查必須後端擋（防止前端被跳過），前端頂多提前做 UI 隱藏。

### 5.8 標籤比對（去空白、英文不分大小寫、沿用既有標籤）由前端查詢還是後端處理

- **情境**：上傳作品／委託項目時掛標籤，規則是「比對到既有標籤就沿用，不新增」（見 [features.md 0](features.md#0-共用規則分類與標籤)）。
- **選項**：
  - 前端先呼叫 `GET /tags?q=` 查詢既有標籤，使用者選到既有的就送 `tag_id`，沒選到的新字串才送出讓後端建立。
  - 或者前端一律送文字陣列，後端自己做正規化比對與建立。
- **建議討論方向**：後端處理比較保險（避免兩人同時建立同義字造成競態重複），但前端還是需要 `GET /tags?q=` 做輸入建議，這支查詢 API 要不要算進這次要定案的清單，也請跟淳確認。
