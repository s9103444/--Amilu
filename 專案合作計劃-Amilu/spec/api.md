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

| 項目      | 暫定假設                                                                                                                                    | 狀態                                                                                          |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| 後端框架  | **FastAPI**（理由見 [5.1](#51-後端框架與基礎慣例未定案)）                                                                                   | 已定案                                                                                        |
| Base path | `/api/v1/`                                                                                                                                  | 待確認（要不要做版本號）                                                                      |
| 格式      | Request／Response 皆為 JSON；**上傳圖片的 API 例外，為 `multipart/form-data`**（見 [5.2](#52-圖片上傳機制未定案)，已定案用方案 A 直傳後端） | 圖片上傳已定案，其餘待確認                                                                    |
| 認證方式  | 未定（session cookie／JWT 皆可能，建議搭配 FastAPI 用 `OAuth2PasswordBearer`＋JWT）                                                         | 待確認，見 [5.1](#51-後端框架與基礎慣例未定案)                                                |
| 錯誤格式  | 未定，暫定 `{ "error": { "code": "...", "message": "..." } }`                                                                               | 待確認                                                                                        |
| 分頁      | 未定，列表類 API 先假設 `page`＋`page_size`                                                                                                 | 待確認                                                                                        |
| 時間格式  | ISO 8601（`datetime`／`date` 對齊 FigJam ER 型別）                                                                                          | 可直接採用                                                                                    |
| 金額單位  | 新台幣整數（元），不處理小數                                                                                                                | 可直接採用（配合 [order-lifecycle.md 三](order-lifecycle.md#三金額與結算公式)無條件捨去規則） |

---

## 2. API 清單（依五大模組）

### 2.1 模組一｜帳號與身份

| 方法與路徑                           | 用途                                                                 | 對應畫面                                           | 對應資料表                |
| ------------------------------------ | -------------------------------------------------------------------- | -------------------------------------------------- | ------------------------- |
| `POST /auth/register`                | 帳密註冊（帳號 ID、暱稱、email、密碼）                               | [1.1 註冊與登入](features.md#11-註冊與登入)        | `USER`                    |
| `POST /auth/login`                   | 帳密登入                                                             | [1.1](features.md#11-註冊與登入)                   | `USER`                    |
| `POST /auth/google`                  | Google 快捷登入                                                      | [1.1](features.md#11-註冊與登入)                   | `USER`                    |
| `POST /auth/google/complete-profile` | Google 補填資料（帳號 ID、暱稱、大頭照、同意條款）                   | [1.1 Google 補填資料頁](features.md#11-註冊與登入) | `USER`                    |
| `POST /auth/logout`                  | 登出                                                                 | —                                                  | —                         |
| `POST /phone-verifications`          | 發送手機驗證碼                                                       | [1.2 手機驗證](features.md#12-手機驗證)            | `USER`                    |
| `POST /phone-verifications/confirm`  | 確認驗證碼                                                           | [1.2](features.md#12-手機驗證)                     | `USER`                    |
| `GET /me`                            | 取得目前使用者（含 `CREATOR_PROFILE.store_approved` 判斷是否創作者） | [1.3 身份](features.md#13-身份)                    | `USER`、`CREATOR_PROFILE` |

### 2.2 模組二｜探索與瀏覽

| 方法與路徑                                    | 用途                                                                       | 對應畫面                                                                                 | 對應資料表                                 |
| --------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------------ |
| `GET /works`                                  | 作品牆（一般作品，支援分類／用途／標籤／關鍵字篩選＋分頁）                 | [2.1 首頁作品牆](features.md#21-首頁作品牆)、[2.2 搜尋與篩選](features.md#22-搜尋與篩選) | `WORK`                                     |
| `GET /commission-items`                       | 可委託項目列表（同上篩選，依 Amilu 燈自動過濾）                            | [2.1](features.md#21-首頁作品牆)、[2.2](features.md#22-搜尋與篩選)                       | `COMMISSION_ITEM`                          |
| `GET /works/{id}`                             | 作品詳情（作品展示燈箱）                                                   | [2.5 作品展示燈箱](features.md#25-作品展示燈箱)                                          | `WORK`、`WORK_IMAGE`                       |
| `GET /creators/{creator_id}/commission-items` | 單一創作者的委託商店內容                                                   | [2.6 委託商店燈箱](features.md#26-委託商店燈箱)                                          | `COMMISSION_ITEM`                          |
| `GET /commission-items/{id}`                  | 委託項目詳情                                                               | [2.6](features.md#26-委託商店燈箱)                                                       | `COMMISSION_ITEM`、`COMMISSION_ITEM_IMAGE` |
| `GET /users/{id}`                             | 創作者個人頁基本資料                                                       | [2.7 創作者個人頁](features.md#27-創作者個人頁)                                          | `USER`、`CREATOR_PROFILE`                  |
| `GET /users/{id}/works`                       | 個人頁「作品」分頁籤                                                       | [2.7](features.md#27-創作者個人頁)                                                       | `WORK`                                     |
| `GET /users/{id}/commission-items`            | 個人頁「可委託項目」分頁籤                                                 | [2.7](features.md#27-創作者個人頁)                                                       | `COMMISSION_ITEM`                          |
| `POST /follows`                               | 追蹤                                                                       | [2.8 追蹤](features.md#28-追蹤)                                                          | `FOLLOW`                                   |
| `DELETE /follows/{following_id}`              | 取消追蹤                                                                   | [2.8](features.md#28-追蹤)                                                               | `FOLLOW`                                   |
| `GET /users/{id}/followers`                   | 追蹤清單燈箱（followers 頁籤）                                             | [2.8](features.md#28-追蹤)                                                               | `FOLLOW`                                   |
| `GET /users/{id}/following`                   | 追蹤清單燈箱（following 頁籤）                                             | [2.8](features.md#28-追蹤)                                                               | `FOLLOW`                                   |
| `GET /categories`                             | 作品類型查找表                                                             | [0 共用規則](features.md#0-共用規則分類與標籤)                                           | `CATEGORY`                                 |
| `GET /purposes`                               | 用途查找表                                                                 | [0](features.md#0-共用規則分類與標籤)                                                    | `PURPOSE`                                  |
| `GET /tags?q=`                                | 自訂標籤自動完成建議（輸入時提示既有標籤，**非必要欄位**，不影響送出格式） | [0](features.md#0-共用規則分類與標籤)                                                    | `TAG`                                      |

> 標籤比對已定案（見 [5.8](#58-標籤比對去空白英文不分大小寫沿用既有標籤由前端查詢還是後端處理)）：`POST /works`、`POST /commission-items` 的標籤欄位直接送**文字陣列**（不是 `tag_id`），去空白、英文不分大小寫、沿用既有標籤或新建的比對邏輯全部由後端處理。

### 2.3 模組三｜個人中心

| 方法與路徑                                                          | 用途                                         | 對應畫面                                            | 對應資料表                                 |
| ------------------------------------------------------------------- | -------------------------------------------- | --------------------------------------------------- | ------------------------------------------ |
| `GET /me/profile`                                                   | 個人資料讀取                                 | [3.1 個人資料設定](features.md#31-個人資料設定)     | `USER`                                     |
| `PATCH /me/profile`                                                 | 修改大頭照／暱稱／簡介                       | [3.1](features.md#31-個人資料設定)                  | `USER`                                     |
| `DELETE /me`                                                        | 刪除帳號（軟刪除，條件見畫面規則）           | [3.1](features.md#31-個人資料設定)                  | `USER`                                     |
| `POST /works`                                                       | 上傳一般作品                                 | [3.2 上傳一般作品](features.md#32-上傳一般作品)     | `WORK`、`WORK_IMAGE`、`WORK_TAG`           |
| `POST /commission-items`                                            | 上架可委託項目                               | [3.3 上傳可委託項目](features.md#33-上傳可委託項目) | `COMMISSION_ITEM`、`COMMISSION_ITEM_IMAGE` |
| `PATCH /commission-items/{id}`                                      | 編輯可委託項目                               | [3.3](features.md#33-上傳可委託項目)                | `COMMISSION_ITEM`                          |
| `DELETE /commission-items/{id}`                                     | 刪除可委託項目（真刪；已有案件讀快照，`ORDER.item_id` 設為空） | [3.3](features.md#33-上傳可委託項目)                | `COMMISSION_ITEM`                          |
| `GET /me/creator-onboarding`                                        | 開通創作者商店三步驟目前完成狀態             | [3.4 開通創作者商店](features.md#34-開通創作者商店) | `CREATOR_PROFILE`                          |
| `POST /me/creator-onboarding/confirm`                               | 確認開通（三步驟皆完成後）                   | [3.4](features.md#34-開通創作者商店)                | `CREATOR_PROFILE`                          |
| `GET /me/creator-settings`                                          | 接案設定讀取                                 | [3.5 接案設定](features.md#35-接案設定)             | `CREATOR_PROFILE`                          |
| `PATCH /me/creator-settings`                                        | 接案設定修改（收款帳戶、訂金比例、修改次數） | [3.5](features.md#35-接案設定)                      | `CREATOR_PROFILE`                          |
| `PATCH /me/creator-settings/amilu-light`                            | Amilu 燈開關切換                             | [3.6 Amilu 燈](features.md#36-amilu-燈)             | `CREATOR_PROFILE`                          |
| `GET /orders?role={commissioner\|creator}&phase={ongoing\|history}` | 約稿管理列表（已委託／已接案，進行中／歷史） | [3.7 約稿管理](features.md#37-約稿管理)             | `ORDER`                                    |

### 2.4 模組四｜委託流程與協作

| 方法與路徑                                                                                                                             | 用途                                                                                                                                            | 對應畫面                                                                 | 對應資料表                                            |
| -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ----------------------------------------------------- |
| `POST /orders`                                                                                                                         | 送出委託申請                                                                                                                                    | [4.1 委託申請頁](features.md#41-委託申請頁委託人填寫)                    | `ORDER`、`ORDER_REFERENCE_IMAGE`                      |
| `POST /orders/{id}/quote`                                                                                                              | 創作者報價（金額、起訖日、各階段 deadline）                                                                                                     | [4.2 報價](features.md#42-報價創作者填寫)                                | `ORDER`                                               |
| `POST /orders/{id}/confirm-quote`                                                                                                      | 委託人確認報價，進入付款流程                                                                                                                    | [4.3 付款頁](features.md#43-付款頁委託人)                                | `ORDER`                                               |
| `GET /orders/{id}/payment`                                                                                                             | 取得付款資訊（虛擬帳號、銀行代碼、期限）                                                                                                        | [4.3](features.md#43-付款頁委託人)                                       | `PAYMENT`                                             |
| `POST /orders/{id}/payment/plan`                                                                                                       | 選擇付款方案（先付訂金／一次付清）                                                                                                              | [4.3](features.md#43-付款頁委託人)                                       | `PAYMENT`、`ORDER.payment_plan`                       |
| `POST /orders/{id}/payment/actions`<br>body `{ "action": "notify_paid" \| "simulate_paid" }`                                           | 「通知已付款」／「模擬入帳」（demo，契約成立）合併成一個 action 參數的 API（已定案，見 [5.3](#53-模擬金流按鈕要不要獨立-api)）                  | [4.3](features.md#43-付款頁委託人)                                       | `PAYMENT`、`ORDER`                                    |
| `GET /orders/{id}`                                                                                                                     | 訂單詳情（契約快照、目前階段、結算用欄位，見[第 3 節](#3-金額結算類特殊-api)）                                                                  | [4.4 委託討論牆](features.md#44-委託討論牆)                              | `ORDER`                                               |
| `GET /orders/{id}/stages`                                                                                                              | 階段清單與各階段狀態、金額（即時計算）                                                                                                          | [4.4](features.md#44-委託討論牆)                                         | `ORDER_STAGE`、`ORDER_STAGE_IMAGE`                    |
| `POST /orders/{id}/stages/{stage_no}/submissions`（`multipart/form-data`，1～5 張）                                                    | 創作者上傳／重傳本階段稿件；重傳時整個請求帶齊這次全部圖片，後端整組覆蓋（已定案，見 [5.2](#52-圖片上傳機制未定案)）                            | [4.4 上傳作品燈箱](features.md#44-委託討論牆)                            | `ORDER_STAGE_IMAGE`、`ORDER_STAGE.submitted_at`       |
| `POST /orders/{id}/stages/{stage_no}/confirm`                                                                                          | 委託人確認該階段審核（階段 1～4）；`stage_no`＝5 時為「確認收稿」，只有一次付清案件可呼叫，提前定稿                                             | [4.4](features.md#44-委託討論牆)                                         | `ORDER_STAGE`                                         |
| `GET /orders/{id}/topup`                                                                                                               | 補款資訊（金額即時計算、期限）                                                                                                                  | [4.4 補款頁](features.md#44-委託討論牆)                                  | `PAYMENT`                                             |
| `POST /orders/{id}/topup/actions`<br>body `{ "action": "notify_paid" \| "simulate_paid" }`                                             | 「通知已補款」／「模擬入帳」（demo，補款入帳即定稿並結案），合併成一個 action 參數的 API（已定案，見 [5.3](#53-模擬金流按鈕要不要獨立-api)）    | [4.4](features.md#44-委託討論牆)                                         | `PAYMENT`、`ORDER`                                    |
| `GET /orders/{id}/settlement-due`                                                                                                      | 終態案件的「應補付」金額與匯款資訊（即時計算；無應補付時回空）                                                                                  | [4.5 結案畫面](features.md#45-結案畫面)                                  | `PAYMENT`、`ORDER`                                    |
| `POST /orders/{id}/settlement-due/actions`<br>body `{ "action": "notify_paid" \| "simulate_paid" }`                                    | 「通知已補付」／「模擬入帳」（demo，補付入帳後解鎖委託人帳號），同 [5.3](#53-模擬金流按鈕要不要獨立-api) 用 action 參數合併                     | [4.5](features.md#45-結案畫面)                                           | `PAYMENT`、`USER.account_status`                      |
| `POST /orders/{id}/debug/actions`<br>body `{ "action": "expire_payment" \| "expire_topup" \| "expire_topup_grace" \| "force_settle" }` | **demo-only**：手動把案件推進到逾期／強制結算後的狀態，測試與簡報用，上線前整組拔掉（已定案，見 [5.4](#54-系統排程觸發的動作與測試用時間控制)） | [acceptance.md E2E-05／06／07D](acceptance.md#1-主線情境)                | `PAYMENT`、`ORDER`                                    |
| `GET /orders/{id}/messages`                                                                                                            | 聊天室訊息列表                                                                                                                                  | [4.7 聊天室](features.md#47-聊天室)                                      | `MESSAGE`                                             |
| `POST /orders/{id}/messages`                                                                                                           | 傳送訊息（文字／圖片）；終態案件唯讀，不可呼叫                                                                                                  | [4.7](features.md#47-聊天室)                                             | `MESSAGE`                                             |
| `POST /orders/{id}/cancel`                                                                                                             | 取消委託（契約成立前）                                                                                                                          | [4.8 取消委託燈箱](features.md#48-取消委託燈箱)                          | `ORDER.cancel_reason`                                 |
| `GET /orders/{id}/termination-preview`                                                                                                 | 終止合作結算試算（三個固定欄位，即時計算）                                                                                                      | [4.9 終止合作燈箱](features.md#49-終止合作燈箱)                          | 見[第 3 節](#3-金額結算類特殊-api)                    |
| `POST /orders/{id}/terminate`                                                                                                          | 申請終止合作（二次確認後送出，立即生效）                                                                                                        | [4.9](features.md#49-終止合作燈箱)                                       | `ORDER.terminated_by_id`／`terminated_at`             |
| `POST /orders/{id}/ratings`                                                                                                            | 雙向評價（1～5 星）                                                                                                                             | [4.5 結案畫面](features.md#45-結案畫面)、[5.1 評價](features.md#51-評價) | `ORDER.rating_for_creator`／`rating_for_commissioner` |

### 2.5 模組五｜通知與評價

| 方法與路徑                          | 用途                                   | 對應畫面                                            | 對應資料表     |
| ----------------------------------- | -------------------------------------- | --------------------------------------------------- | -------------- |
| `GET /notifications`                | 通知列表（專案通知／系統公告）         | [5.2 通知中心「通知」頁籤](features.md#52-通知中心) | `NOTIFICATION` |
| `POST /notifications/mark-all-read` | 全部標為已讀                           | [5.2](features.md#52-通知中心)                      | `NOTIFICATION` |
| `GET /notifications/unread-count`   | 未讀數（header 鈴鐺紅點、頁籤旁數字）  | [5.2](features.md#52-通知中心)                      | `NOTIFICATION` |
| `GET /conversations`                | 對話頁籤列表（各案件最新一則訊息預覽） | [5.2 通知中心「對話」頁籤](features.md#52-通知中心) | `MESSAGE`      |

---

## 3. 金額／結算類特殊 API

[data-model.md 5.1](data-model.md#51-哪些值刻意不存) 明確列出這些值刻意不存欄位、全部即時計算，所以前端**不能自己算，一定要靠 API 拿到後端算好的數字**：

| 值                                                             | 怎麼拿到                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 對應規則                                                                                                      |
| -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| 本階段金額                                                     | `GET /orders/{id}/stages` 回應裡每個階段帶 `amount` 欄位。**這只是顯示用的試算金額，不代表撥款**——撥款只發生在三個時點：第一階段確認（撥訂金）、完稿定稿（撥尾款）、終止合作結算，中間階段確認不觸發撥款（2026/10/15 定案，見 [order-lifecycle.md 三｜付款方案](order-lifecycle.md#三金額與結算公式)）。`POST /orders/{id}/stages/{stage_no}/confirm` 只有在 `stage_no`＝1（撥訂金）或＝5（確認收稿，撥尾款）時才會連帶產生 `PAYMENT` 已撥款紀錄，中間階段的 confirm 只改 `ORDER_STAGE` 確認狀態；一次付清案件的完稿窗口期滿，由後端排程自動定稿並撥款 | [order-lifecycle.md 三｜階段金額拆分、付款方案](order-lifecycle.md#三金額與結算公式)                          |
| 補款金額                                                       | `GET /orders/{id}/topup` 回應                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | `quote_amount` − `PAYMENT` 已託管入款加總，[data-model.md 4.4](data-model.md#44-order-結案方式判斷不另設欄位) |
| 終止合作結算試算（委託人已付／創作者可得／委託人退款或應補付） | `GET /orders/{id}/termination-preview`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | [order-lifecycle.md 三｜終止合作結算](order-lifecycle.md#三金額與結算公式)                                    |
| 目前進行到哪一階段                                             | `GET /orders/{id}/stages` 查哪一筆未確認                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [data-model.md 5.1](data-model.md#51-哪些值刻意不存)                                                          |
| 結案方式（已完成／已終止，是否雙輸）                           | `GET /orders/{id}` 回應需帶 `status`、`completed_at`、`terminated_by_id`、`terminated_at`，前端依此判斷                                                                                                                                                                                                                                                                                                                                                                                                                                                | [data-model.md 4.4](data-model.md#44-order-結案方式判斷不另設欄位)                                            |
| followers／following 數字                                      | `GET /users/{id}` 回應帶計數，或 `GET /users/{id}/followers`／`following` 回應帶 `total`                                                                                                                                                                                                                                                                                                                                                                                                                                                               | [data-model.md 5.1](data-model.md#51-哪些值刻意不存)                                                          |

---

## 4. 跨模組規則

### 4.1 契約快照欄位

`GET /orders/{id}` 要回傳的快照欄位與各自鎖定時間點，詳細規則見 [order-lifecycle.md 五](order-lifecycle.md#五契約快照與同意)：

| 欄位                                                                                                | 鎖定時間點                                                                                                                                           | 送出委託申請後／契約成立後回應是否不同                                                                                |
| --------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `item_title_snapshot`、`item_description_snapshot`、項目範例圖片快照（`ORDER_ITEM_SNAPSHOT_IMAGE`）、`deposit_percent_snapshot`、`revision_count_snapshot`、`stage_plan_snapshot` | 甲方**送出委託申請時**（2026/10/15 定案，這些欄位都在這個時間點一起鎖；說明與圖片為 2026/10/17 新增，不是契約成立時，見 [data-model.md 4.1](data-model.md#41-order-契約快照欄位)） | 申請時就鎖定，申請後與契約成立後回應相同                                                                              |
| 授權層級                                                                                            | 送出委託申請時                                                                                                                                       | 同上                                                                                                                  |
| 報價金額、起訖日、各階段 `deadline`                                                                 | 乙方報價時填寫，甲方確認報價時鎖定                                                                                                                   | 報價前為空，確認後鎖定                                                                                                |
| `creator_agreed_at`／`commissioner_agreed_at`                                                       | 乙方送出報價時／甲方在付款頁按「確認送出」時（2026/10/15 起，甲方不再記在送出申請時）                                                                | 申請後、報價後、付款前，`commissioner_agreed_at` 皆為空                                                               |
| 其餘契約快照（表單填寫內容、付款方式等，不含上面四個欄位）                                          | 平台確認入帳（契約成立）時                                                                                                                           | 成立前後欄位內容不同，待確認是否要用同一個 API 回應區分「草稿版」與「鎖定版」，見 [5.1](#51-後端框架與基礎慣例未定案) |

### 4.2 權限規則（動作 → 允許呼叫者）

同一筆 `ORDER`，甲方（`commissioner_id`）與乙方（`creator_id`）能呼叫的動作不同：

| 動作                                                                                | 允許呼叫者                     |
| ----------------------------------------------------------------------------------- | ------------------------------ |
| `POST /orders`（送出申請）                                                          | 甲方（帳號被鎖時擋下）         |
| `POST /orders/{id}/quote`（報價）                                                   | 乙方                           |
| `POST /orders/{id}/confirm-quote`（確認報價）                                       | 甲方                           |
| `POST /orders/{id}/stages/{stage_no}/submissions`（上傳稿件）                       | 乙方                           |
| `POST /orders/{id}/stages/{stage_no}/confirm`（確認階段；`stage_no`＝5 為確認收稿） | 甲方                           |
| `POST /orders/{id}/cancel`（取消委託）                                              | 甲方或乙方皆可（契約成立前）   |
| `POST /orders/{id}/terminate`（終止合作）                                           | 甲方或乙方皆可（契約成立後）   |
| `POST /orders/{id}/ratings`（評價）                                                 | 甲方或乙方皆可，雙向各自評對方 |
| `POST /orders/{id}/settlement-due/*`（補付應補付金額）                              | 甲方                           |

判斷方式已定案，見 [5.6](#56-同一帳號雙重身份時訂單類-api-怎麼判斷現在是哪一方)。

### 4.3 狀態機對應（API → 狀態轉移）

對照 [order-lifecycle.md 一｜ORDER.status](order-lifecycle.md#一委託狀態機)：

| API                                                                                                                                                                        | 觸發的狀態轉移                                                                                 |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `POST /orders`                                                                                                                                                             | 建立，進入「待乙方報價」                                                                       |
| `POST /orders/{id}/quote`                                                                                                                                                  | 「待乙方報價」→「待甲方付款」                                                                  |
| `POST /orders/{id}/payment/actions`（`action=simulate_paid`）                                                                                                              | 「待甲方付款」→「製作中」（契約成立）                                                          |
| `POST /orders/{id}/stages/5/confirm`（確認收稿，一次付清）／`POST /orders/{id}/topup/actions`（`action=simulate_paid`，只付訂金）／後端排程（一次付清，完稿 7 天窗口期滿） | 「製作中」→「已完成」                                                                          |
| `POST /orders/{id}/terminate`                                                                                                                                              | 「製作中」→「已終止」                                                                          |
| `POST /orders/{id}/cancel`                                                                                                                                                 | 「待乙方報價」或「待甲方付款」→「已取消」                                                      |
| 無對應 API（後端排程觸發；demo／測試用 `POST /orders/{id}/debug/actions` 代替）                                                                                            | 付款逾期自動取消、逾期強制結算、補款逾期結算，見 [5.4](#54-系統排程觸發的動作與測試用時間控制) |

---

## 5. 待確認事項

以下每項都是跟淳對規格前要先討論的議程，寫清楚情境、選項與影響範圍，方便直接拿去開會用。

### 5.1 後端框架與基礎慣例未定案

**狀態：框架已定案（FastAPI），錯誤格式／分頁／驗證方式仍待確認**

- **框架：FastAPI**（2026/10/15 定案），理由：
  1. **自動產生互動式 API 文件**（Swagger UI／ReDoc，直接從程式碼的型別標註與 Pydantic model 生成）。這正好解決這份 `api.md` 存在的問題——對規格時不用再手動一條條核對 request/response，FastAPI 生出來的文件就是即時、保證跟程式碼一致的規格，之後 api.md 可以只留「這支 API 做什麼、對應哪個畫面」，細節欄位直接看自動文件。
  2. 用 Python type hint 寫，概念上接近 TypeScript 的型別系統，對完全沒寫過後端但寫 TS 的人會比較好上手。
  3. 內建檔案上傳處理（`UploadFile`），正好對上 [5.2](#52-圖片上傳機制未定案) 已定案的「直傳後端」方案，不用額外找套件。
  4. 比 Django 輕量，這個專案明確「沒有營運後台」（[prd.md](prd.md#mvp-範圍)），用不到 Django admin 這類開箱即用但這裡不需要的功能；搭配 **SQLModel**（FastAPI 作者寫的套件，整合 SQLAlchemy＋Pydantic）可以直接對應 ER 圖的 20 張表，schema 只要寫一次。
- **還沒定案的部分**：錯誤格式、分頁、驗證方式，待跟淳對規格時一起定，暫定對齊 FastAPI 慣例：`HTTPException`＋統一的錯誤 response model、`page`＋`page_size` 查詢參數、`OAuth2PasswordBearer`＋JWT。
- **影響範圍**：幾乎所有 list／寫入類 API 的格式，建議第一次對規格會議就先定這三件事，其他 endpoint 才能照同一套慣例寫。

### 5.2 圖片上傳機制未定案

**狀態：已定案 — 方案 A（直傳後端）**

- 涉及的上傳情境：`USER.avatar_url`（單張）、`WORK_IMAGE`（1～20 張）、`COMMISSION_ITEM_IMAGE`（1～5 張）、`ORDER_STAGE_IMAGE`（每次 1～5 張）、`ORDER_REFERENCE_IMAGE`、聊天室 `MESSAGE.image_url`（單張），全部採用**方案 A：前端 `multipart/form-data` 直接丟後端存檔、回網址**，不做 presigned URL／雲端直傳。
- 實務上：有圖片的建立／更新 API（`POST /works`、`POST /commission-items`、`PATCH /me/profile`、`POST /orders`、`POST /orders/{id}/messages`、`POST /orders/{id}/stages/{stage_no}/submissions`）body 都改成 `multipart/form-data`，圖片檔案與其他欄位一起送；後端存檔位置（本機磁碟或之後換物件儲存）是淳的實作細節，不影響這裡的 API 形狀。
- **`ORDER_STAGE_IMAGE` 整組覆蓋**（見 [data-model.md 4.3](data-model.md#43-order_stage_image-上傳規則)）：重傳時前端把這次要送的全部圖片（1～5 張）一次放進同一個 `POST /orders/{id}/stages/{stage_no}/submissions` 請求，後端收到後刪掉該 `stage_id` 底下所有舊列、整組重寫、`seq_no` 從 1 重編——不是逐張增量上傳，前端也不需要先呼叫刪除 API。
- **影響範圍**：第 1 節共用慣例已標註圖片類 API 是 `multipart/form-data` 的例外；對規格時跟淳確認檔案大小限制（各畫面已有的 5MB／張）在後端怎麼擋（框架層級或自己寫驗證）。

### 5.3 模擬金流按鈕要不要獨立 API

**狀態：已定案 — 用 action 參數合併**

- 「模擬入帳」「通知已付款」「通知已補款」這幾個 demo 用按鈕，改成兩個各自吃 `action` 參數的 API（已反映在第 2.4 節）：
  - `POST /orders/{id}/payment/actions`，body `{ "action": "notify_paid" | "simulate_paid" }`
  - `POST /orders/{id}/topup/actions`，body `{ "action": "notify_paid" | "simulate_paid" }`
- **決定原因**：這次 demo 不打算真的串綠界代收轉付，所以不需要為了「之後好拔掉換 webhook」預先把每個按鈕切成獨立 endpoint；用 action 參數省事，之後如果真要做 v1.1 金流串接，屆時這塊 API 會整個重新設計，現在不用先做乾淨切分。

### 5.4 系統排程觸發的動作與測試用時間控制

**狀態：已定案 — 可調閾值 ＋ demo-only 手動觸發，組合使用**

> 這一項直接對應 [acceptance.md 第 5 節「還沒決定」#6](acceptance.md#5-待確認事項)：E2E-05、E2E-06、E2E-07 子情境 D 都標了「需時間控制」，acceptance.md 寫明「這點留給 api.md 決定」——這裡是決定內容，對規格時跟淳確認可行性，確定後回 `acceptance.md` 把 #6 從「還沒決定」移到「已定案」。

- **情境**：以下動作都是後端排程／cron 觸發，不是前端按按鈕（見 [order-lifecycle.md 二｜計時規則](order-lifecycle.md#二計時規則)），前端完全不呼叫 API 觸發，只被動收到通知後重新 `GET /orders/{id}`：
  - 付款期限 3 天到期未入帳 → 自動取消（E2E-07 子情境 D）
  - 完稿 7 天窗口期滿（起算點為最近一次完稿上傳時間，每次重傳重算，提前交稿也從上傳當下起算）：一次付清 → 自動完成（E2E-09）；只付訂金 → 進入 7 天等待期，等待期滿仍未補齊 → 依「完稿視同已確認」結算（E2E-05）
  - 乙方尚未上傳完稿、整筆委託最終截稿日 +30 天仍未了結 → 系統強制結算，只看委託人有沒有確認（E2E-06）
  - 任何結算後委託人應補付未付 → 鎖帳號（只擋 `POST /orders`，見 [order-lifecycle.md 三](order-lifecycle.md#結算後的鎖帳號與補付入口)）
- **可調閾值**：把 3／7／30 天抽成設定值（環境變數或設定檔），正式環境用正常天數，測試／demo 環境調小（例如 3 天→30 秒、30 天→5 分鐘）。要注意「完稿窗口 7 天接著等待期 7 天」這種連續計時，調小後測試仍要等兩段。
- **demo-only 手動觸發**：新增一組 debug 專用 API，風格對齊 [5.3](#53-模擬金流按鈕要不要獨立-api) 的 action 參數決定，不用再想獨立 endpoint 命名：
  `POST /orders/{id}/debug/actions`，body `{ "action": "expire_payment" | "expire_topup" | "expire_topup_grace" | "force_settle" }`，直接把指定 `ORDER` 推進到對應的逾期／結算後狀態（`expire_topup`＝完稿 7 天窗口期滿：一次付清自動完成、只付訂金進等待期；`expire_topup_grace`＝等待期滿結算）。只在非正式環境開放（例如用環境變數整組關掉這個 router），上線前拔掉或鎖權限。
- **分工**：可調閾值驗的是「到期後邏輯對不對」，手動觸發驗的是「結算金額對不對」且不用等任何時間，兩者互補，Playwright 測試視情境選一種用。
- **影響範圍**：`acceptance.md` E2E-05、E2E-06、E2E-07 子情境 D、E2E-09 這幾條測試可以直接用 `/debug/actions` 寫，不用真的等待（或搭配調小的閾值測「到期後」的那一刻）。

### 5.5 聊天室與通知更新機制：輪詢頻率，還是要做推播

**狀態：已定案 — 輪詢，依場景分頻率**

- **不做 WebSocket／SSE**：`MESSAGE`、`NOTIFICATION` 在 [features.md](features.md#47-聊天室) 都明確定義為「非即時」，這是產品本身的決定，不是技術妥協。真正的即時推播要處理持久連線、斷線重連、多分頁／多裝置同步、後端要維護連線狀態——這些成本對一個兩人、demo 規模、本來就設計成非即時的專案不成比例。輪詢足夠，且前後端都是目前已經在用的一般 REST API，不需要多學一套技術。
- **輪詢怎麼切，依場景給不同頻率**：
  | 場景 | 打哪支 API | 頻率 | 什麼時候跑／停 |
  | --- | --- | --- | --- |
  | header 通知鈴鐺未讀數 | `GET /notifications/unread-count` | 15～30 秒 | 使用者登入後全站背景跑；分頁切到背景（`document.visibilitychange`）就停止或降頻，省後端資源 |
  | 通知中心「對話」頁籤 | `GET /conversations` | 不常駐輪詢，打開頁籤時打一次即可 | 頁籤收起就停 |
  | 討論牆聊天室（浮動視窗展開時） | `GET /orders/{id}/messages` | 5～8 秒 | 只有視窗展開時才跑，收起就停；使用者正盯著看，可以比鈴鐺快一點 |
- **實作方式**：Vue3 寫一個共用 composable（例如 `usePolling(fetchFn, intervalMs)`），在 `onMounted` 啟動 `setInterval`，在 `onUnmounted` 與 `visibilitychange` 切背景時清掉，三個場景共用同一套邏輯，不用各自重複寫計時器管理。
- **實作時留意**：demo 使用者數量是個位數，上面的頻率對後端負擔不是問題；`GET /notifications/unread-count` 這支輕量 API 若淳覺得沒差，也可以直接拿掉，前端改打 `GET /notifications` 自己算未讀數，不影響其他設計。

### 5.6 同一帳號雙重身份時，訂單類 API 怎麼判斷「現在是哪一方」

**狀態：已定案**

- **情境**：約稿管理「已委託」「已接案」兩個頁籤是同一個使用者，但一個是甲方視角一個是乙方視角；報價、確認階段、終止合作這些 API 的權限判斷都要分清楚呼叫者是 `ORDER.commissioner_id` 還是 `creator_id`。
- **定案**：後端直接用 token 裡的 `user_id` 比對該筆 `ORDER` 的兩個欄位、依結果決定能呼叫哪些動作（例如報價只能乙方呼叫、確認階段只能甲方呼叫），**前端不傳任何「視角」參數**，全部由後端依資料判斷，這樣前端沒有機會傳錯視角造成誤判。
- **影響範圍**：[4.2 權限規則](#42-權限規則動作-→-允許呼叫者)整張表就是這個判斷邏輯的具體動作清單，對規格時逐條跟淳確認錯誤回應格式（例如呼叫者不是允許的一方時回什麼錯誤碼）。

### 5.7 防自刷（同帳號不能委託自己）由後端擋，前端只是提示

**狀態：延後討論（先不處理，等之後再定）**

- **情境**：[features.md 1.3](features.md#13-身份)「委託人與創作者是同一個帳號時，擋下送出委託」。
- **先維持現狀**：`POST /orders` 的錯誤碼／訊息格式這次先不決定，等之後有空再跟淳討論；這個檢查本身確定要放在後端（防止前端被跳過），前端頂多提前做 UI 隱藏，不影響這份文件其他部分的設計。

### 5.8 標籤比對（去空白、英文不分大小寫、沿用既有標籤）由前端查詢還是後端處理

**狀態：已定案 — 後端處理**

- **情境**：上傳作品／委託項目時掛標籤，規則是「比對到既有標籤就沿用，不新增」（見 [features.md 0](features.md#0-共用規則分類與標籤)）。
- **定案**：`POST /works`、`POST /commission-items` 的標籤欄位送**文字陣列**，不送 `tag_id`；去空白、英文不分大小寫、沿用既有標籤或新建的正規化邏輯全部由後端處理（避免兩人同時建立同義字造成競態重複）。
- `GET /tags?q=` 保留，作為前端輸入時的自動完成建議，純粹方便使用者打字，不影響送出格式、也不是必要欄位。
