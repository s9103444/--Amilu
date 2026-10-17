# Amilu 資料模型

> 這份說明「FigJam 上的 ER 圖怎麼讀、有哪些表、表之間怎麼連、哪些值刻意不存」。
> **欄位清單以 FigJam 為準**，這裡不重抄，避免兩邊各自維護。要查某張表的完整欄位，請直接開圖。
> 訂單狀態怎麼變、各種倒數天數與金額公式，見 [order-lifecycle.md](order-lifecycle.md)。

**ER 圖**：[FigJam｜ER Diagram - Amilu](https://www.figma.com/board/KNj96DaqlW6fQ4NuWFu52b/ER-Diagram---Amilu?node-id=0-1)

---

## 1. 怎麼讀這張圖

### 1.1 每張表的格式

每張表是一個 FigJam table，4 欄：

| 欄 | 內容 |
| --- | --- |
| 1 | 資料型別：`string`、`number`、`boolean`、`date`、`datetime` |
| 2 | 欄位名稱（英文 snake_case） |
| 3 | 鍵：`PK` 主鍵、`FK` 外鍵、`UK` 唯一；複合主鍵寫 `PK, FK` 或兩列都標 `PK` |
| 4 | 中文說明 |

- 第一列是表名，格式為「`TABLE_NAME`（中文名稱）」。
- 表名一律大寫加底線，例如 `COMMISSION_ITEM`。
- 型別欄是「概念型別」，實際資料庫型別由後端決定。

### 1.2 關聯線與基數符號

線上的文字是關係名稱（例如「刊登」「分段紀錄」），線的兩端是 crow's foot 基數符號。畫面左上角的 `Components` 區塊有四種符號的圖例：

| 符號 | 意思 |
| --- | --- |
| One | 剛好一個 |
| Zero or One | 零或一個 |
| Zero or Many | 零或多個 |
| One or Many | 一或多個（圖例有，目前沒有關聯線用到） |

圖上所有關聯線都是「一方是 One、另一方是 Zero or Many 或 Zero or One」，沒有多對多的線。多對多一律拆成中介表（見 2.2）。

---

## 2. 有哪些表（共 20 張）

### 2.1 帳號與身份

| 表 | 中文 | PK | 重點 |
| --- | --- | --- | --- |
| `USER` | 使用者帳號 | `id` | 使用者自訂、全站唯一；`email` 是 UK 但不是主鍵；`account_status` 含正常／鎖定／封鎖／已刪除 |
| `CREATOR_PROFILE` | 創作者接案設定 | `creator_id` | `user_id` 指向 USER 且為 UK，一個帳號最多一份；`store_approved` 判斷是否已開通 |
| `FOLLOW` | 追蹤關係 | `follower_id` ＋ `following_id` | USER 對 USER 的自我關聯，一組關係一列 |

### 2.2 作品與分類

| 表 | 中文 | PK | 重點 |
| --- | --- | --- | --- |
| `WORK` | 作品集（一般作品） | `work_id` | `user_id` 指向 USER（不需開通商店也能發）；`category_id`、`purpose_id` 皆可為空 |
| `COMMISSION_ITEM` | 可委託項目 | `item_id` | `creator_id` 指向 CREATOR_PROFILE；`commercial_allowed`、`stage_plan_pref` 各項目各自設定 |
| `CATEGORY` | 作品類型（查找表） | `category_id` | `name` 為 UK；WORK 與 COMMISSION_ITEM 共用 |
| `PURPOSE` | 作品用途（查找表） | `purpose_id` | `name` 為 UK；WORK 與 COMMISSION_ITEM 共用 |
| `TAG` | 自訂標籤 | `tag_id` | `name` 為 UK，全站共用字典；`created_by_id` 只供稽核，不做權限控制 |
| `WORK_TAG` | 作品標籤中介表 | `work_id` ＋ `tag_id` | WORK 與 TAG 多對多 |
| `COMMISSION_ITEM_TAG` | 可委託項目標籤中介表 | `item_id` ＋ `tag_id` | COMMISSION_ITEM 與 TAG 多對多 |
| `WORK_IMAGE` | 作品集圖片 | `work_id` ＋ `seq_no` | 弱實體，`seq_no` 兼顯示順序，第一張為封面 |
| `COMMISSION_ITEM_IMAGE` | 可委託項目圖片 | `item_id` ＋ `seq_no` | 弱實體，同 WORK_IMAGE |

### 2.3 訂單與金流

| 表 | 中文 | PK | 重點 |
| --- | --- | --- | --- |
| `ORDER` | 委託案件／契約 | `order_id` | 核心表；含契約快照欄位、雙方同意時間、評分、終止資訊 |
| `ORDER_STAGE` | 訂單分段紀錄 | `stage_id` | 一筆訂單 3 列（三點分段）或 5 列（五點分段） |
| `ORDER_STAGE_IMAGE` | 階段稿件圖片 | `stage_id` ＋ `seq_no` | 弱實體；每階段最多 5 張，存浮水印預覽與正式稿兩個網址 |
| `ORDER_REFERENCE_IMAGE` | 委託申請參考圖片 | `order_id` ＋ `seq_no` | 弱實體；只存申請時附的原始參考圖 |
| `ORDER_ITEM_SNAPSHOT_IMAGE` | 項目範例圖片快照 | `order_id` ＋ `seq_no` | 弱實體；送出申請時從 `COMMISSION_ITEM_IMAGE` 複製（2026/10/17 新增） |
| `PAYMENT` | 模擬金流／託管紀錄 | `payment_id` | 一筆訂單多筆：託管入款、撥款、退款、結算後補付入款（`type` 區分） |

### 2.4 溝通與通知

| 表 | 中文 | PK | 重點 |
| --- | --- | --- | --- |
| `MESSAGE` | 非即時聊天室 | `message_id` | 綁定單一訂單；一則最多 1 張圖；沒有已讀欄位 |
| `NOTIFICATION` | 通知 | `notification_id` | `order_id` 可為空（系統公告不綁案件）；`email_sent` 記錄是否已寄信 |

---

## 3. 表之間怎麼連

下表依圖上的 29 條關聯線整理。「一方」永遠是第一個，「多方」是第二個。

### 3.1 帳號與身份

| 關係名稱 | 一方 | 多方 | 基數 |
| --- | --- | --- | --- |
| 開通創作者身份 | USER | CREATOR_PROFILE | 1 對 0..1 |
| 追蹤（follower） | USER | FOLLOW | 1 對 0..N |
| 被追蹤（following） | USER | FOLLOW | 1 對 0..N |

FOLLOW 有兩條線都連到 USER，分別對應 `follower_id` 與 `following_id`。

### 3.2 作品與分類

| 關係名稱 | 一方 | 多方 | 基數 |
| --- | --- | --- | --- |
| 上傳 | USER | WORK | 1 對 0..N |
| 刊登 | CREATOR_PROFILE | COMMISSION_ITEM | 1 對 0..N |
| 分類 | CATEGORY | WORK | 1 對 0..N |
| 分類 | PURPOSE | WORK | 1 對 0..N |
| 分類 | CATEGORY | COMMISSION_ITEM | 1 對 0..N |
| 分類 | PURPOSE | COMMISSION_ITEM | 1 對 0..N |
| 包含 | WORK | WORK_IMAGE | 1 對 0..N |
| 包含 | COMMISSION_ITEM | COMMISSION_ITEM_IMAGE | 1 對 0..N |
| 掛標籤／被掛用 | WORK、TAG | WORK_TAG | 各 1 對 0..N |
| 掛標籤／被掛用 | COMMISSION_ITEM、TAG | COMMISSION_ITEM_TAG | 各 1 對 0..N |
| 建立標籤 | USER | TAG | 1 對 0..N，對應 `TAG.created_by_id`（只供稽核） |

### 3.3 訂單

| 關係名稱 | 一方 | 多方 | 基數 |
| --- | --- | --- | --- |
| 委託（甲方） | USER | ORDER | 1 對 0..N，對應 `ORDER.commissioner_id` |
| 承接（乙方） | CREATOR_PROFILE | ORDER | 1 對 0..N，對應 `ORDER.creator_id` |
| 對應項目 | COMMISSION_ITEM | ORDER | 1 對 0..N，對應 `ORDER.item_id` |
| 分段紀錄 | ORDER | ORDER_STAGE | 1 對 0..N |
| 稿件圖片 | ORDER_STAGE | ORDER_STAGE_IMAGE | 1 對 0..N |
| 金流紀錄 | ORDER | PAYMENT | 1 對 0..N |
| 參考圖片 | ORDER | ORDER_REFERENCE_IMAGE | 1 對 0..N |
| 項目圖片快照 | ORDER | ORDER_ITEM_SNAPSHOT_IMAGE | 1 對 0..N |
| 留言 | ORDER | MESSAGE | 1 對 0..N |
| 關聯通知 | ORDER | NOTIFICATION | 1 對 0..N |
| 終止發起 | USER | ORDER | 1 對 0..N，對應 `ORDER.terminated_by_id`（可為空） |

### 3.4 溝通與通知

| 關係名稱 | 一方 | 多方 | 基數 |
| --- | --- | --- | --- |
| 發送 | USER | MESSAGE | 1 對 0..N，對應 `MESSAGE.sender_id` |
| 接收 | USER | NOTIFICATION | 1 對 0..N，對應 `NOTIFICATION.user_id` |

---

## 4. 圖上的四張便利貼

FigJam 上有四張標題為「備註」的便利貼，貼在對應的表旁邊，記錄無法用欄位表達的規則。

### 4.1 ORDER 契約快照欄位

- `item_title_snapshot`、`deposit_percent_snapshot`、`revision_count_snapshot`、`stage_plan_snapshot` 都在**甲方送出委託申請時**就從可委託項目與接案設定複製一份存進 ORDER（2026/10/15 定案），避免乙方在報價前改了設定，讓甲方申請時看到的條件被改掉。
- 原因：創作者之後修改或刪除可委託項目、調整接案設定時，已成立的案件仍要顯示並依照當下的內容，不會跟著變動。
- **項目說明與範例圖片也要快照**（2026/10/17 定案）：`item_description_snapshot` 與 `ORDER_ITEM_SNAPSHOT_IMAGE` 同樣在送出申請時複製。案件畫面（申請頁帶入的項目資訊、討論牆「委託項目檢視」）一律讀快照，不讀即時的 `COMMISSION_ITEM`。
- 因此可委託項目可以**真的刪除**，不用軟刪除；刪除後 `ORDER.item_id` 設為空值，所以 `item_id` 要改成 nullable。

### 4.2 ORDER_STAGE.stage_no 編碼

- 五點分段：1 草稿、2 線稿、3 彩稿、4 精稿、5 完稿，共 5 列。
- 三點分段（原「單段」，2026/10/15 起中間多一次審核）：1 草稿、4 精稿、5 完稿，共 3 列，沿用五點分段的編號，不另編、不遞補，所以 `stage_no` 會是 1、4、5。
- 判斷規則：`stage_no` ＝ 5 永遠代表完稿（最後階段）；`stage_no` ＝ 1 永遠是第一階段（訂金階段）。兩種分段方式共用這兩個判斷。

### 4.3 ORDER_STAGE_IMAGE 上傳規則

- 乙方每次上傳 1～5 張，單張 5MB 以內（JPG／PNG／WebP）。
- 同一階段重傳視為覆蓋：刪除該 `stage_id` 底下所有列再重寫，`seq_no` 從 1 重編，並更新 `ORDER_STAGE.submitted_at`。
- 階段 1～4：甲方確認該階段後即鎖定，不可重傳。
- 完稿（`stage_no` ＝ 5）：定稿（確認收稿、7 天窗口期滿或補款入帳）前都可以重傳覆蓋，每次重傳除了更新 `submitted_at`，也要重算 `ORDER.finalize_due_at`；定稿後鎖定。

### 4.4 ORDER 結案方式判斷（不另設欄位）

- **已完成**：`status` ＝ 已完成，時間看 `completed_at`；階段 1～4 的 `ORDER_STAGE.confirm_status` 一律是人工確認（過程沒有自動確認機制）；完稿（`stage_no` ＝ 5）則是乙方上傳即視同確認，定稿時點見 order-lifecycle.md。
- **已終止**：`status` ＝ 已終止，發起方送出時寫入 `terminated_by_id`，與 `commissioner_id` 比對即知是甲方或乙方發起；第一階段 `ORDER_STAGE.paid` 決定是雙輸或已撥款結算。
- **補款金額**：後端即時計算 ＝ `quote_amount` − PAYMENT 已託管入款的 `amount` 加總，不存欄位。

---

## 5. 設計決策

### 5.1 哪些值刻意不存

這些值都能從來源資料算出來，存起來只會有跟來源對不上的風險：

| 值 | 怎麼得到 |
| --- | --- |
| 是否為創作者 | 看 `CREATOR_PROFILE.store_approved`，USER 沒有身份欄位 |
| Amilu 燈對外顯示 | 手動開放 ＋ 可委託項目至少 1 筆 ＋ 已開通商店，三者同時成立才顯示「開放中」 |
| 每階段金額 | 由總價、訂金比例、階段數算出，ORDER_STAGE 沒有金額欄位 |
| 目前進行到哪一階段 | 查 ORDER_STAGE，ORDER 不存快取 |
| 補款金額 | 見 4.4 |
| 結案方式 | 見 4.4 |
| followers／following 數字 | 由 FOLLOW 即時計數 |

### 5.2 什麼時候拆表、什麼時候併回父表

- **會隨時間累積的資料要拆表**：PAYMENT、ORDER_STAGE、MESSAGE，以及所有圖片。
- **固定只發生一兩次的單次事實併回父表**：契約同意時間（`commissioner_agreed_at`、`creator_agreed_at`）、評分（`rating_for_creator`、`rating_for_commissioner`）、終止資訊（`terminated_by_id`、`terminated_at`），都直接放在 ORDER。
- 之後遇到「這個要不要開新表」，延用這個判斷。

### 5.3 其他

- **弱實體與複合主鍵**：圖片類五張表用「父表 PK ＋ `seq_no`」當複合主鍵。`seq_no` 在同一個父表底下從 1 開始，刪除不遞補。
- **標籤共用字典**：`TAG.name` 全站唯一。寫入前去除前後空白、英文不分大小寫比對；比對到既有標籤（含 CATEGORY、PURPOSE 的項目）就沿用，不新增。
- **單選分類用直接 FK，不用中介表**：同一軸只能選一個，所以 WORK 與 COMMISSION_ITEM 直接放 `category_id`、`purpose_id`，且可為空。
- **軟刪除**：刪除帳號只改 `USER.account_status` ＝ 已刪除並寫入 `deleted_at`，不刪列，歷史案件才不會指向不存在的帳號。
- **`ORDER.creator_id` 必須保留**：原本可從 `item_id` 推得、算是冗餘 FK；2026/10/17 起項目可真刪、`item_id` 可能為空，`creator_id` 成為判斷乙方的唯一依據。
- **金額與狀態欄位的型別**：訂金比例、分段方案、各種狀態在圖上都標為 `string`，合法值見欄位說明；是否改用 enum 由後端決定。

---

## 6. 圖上目前已知的問題

以下是比對圖與其他文件時發現的問題與處理結果。2026/10/17 已逐表核對過 FigJam，目前**沒有未解決的項目**：

1. **已解決**（2026/10/17）：「關聯通知」那條線原本終點掛在畫布上，已重接到 NOTIFICATION。
2. **已解決**（2026/10/17）：`TAG.created_by_id`、`ORDER.terminated_by_id` 原本沒有畫線，已補上「建立標籤」「終止發起」兩條關聯線。
3. **已解決**：惡意棄單機制已經拿掉（2026/10/12，見 `order-lifecycle.md`、changelog），改用統一的「最終截稿日 +30 天系統強制結算」，結算一律走「已終止」，不需要新增 `status` 值，原本的問題不存在了。FigJam 上 `USER.lock_reason` 已改為「鎖定原因（結算後應補付未付）」，「惡意棄單」列舉值已清掉（2026/10/17 確認）。
4. **已解決**（2026/10/15）：補款期限（一次付清則為完稿窗口）定為 7 天，起算點 ＝ 乙方最近一次上傳完稿的時間（提前交稿也從上傳當下起算，不等截稿日），每次重傳都重算；只付訂金案件到期後另有 7 天等待期，兩者是獨立的兩段。`ORDER.finalize_due_at`（原 `topup_due_at`）記的是這個期限的到期時間。
5. **已解決**（2026/10/15）：`cancel_reason` 維持單一欄位，說明為「取消／終止合作原因」，不拆欄位；是取消還是終止，由 `status` 與 `terminated_by_id` 判斷。

6. **已決定改名**（2026/10/15）：`topup_due_at` 一次付清的完稿窗口也用它，不再只是補款期限，改名為 `finalize_due_at`（完稿定稿／補款期限到期時間）。FigJam 已於 2026/10/15 同步改名。
7. **已解決**（2026/10/15）：結算後的補付入款不新增欄位，用既有的 `PAYMENT.type` 新增一個值「補付入款」（現有值：託管入款／訂金撥款／尾款撥款／退款）；`PAYMENT.status` 不新增狀態。
8. **已解決**（2026/10/15）：完稿窗口期間可以申請終止合作，完稿視同已確認階段，創作者可得包含完稿費用。只付訂金案件在這個情境終止時，委託人要先補付「應補付」才能下載完稿圖，結算當下不能下載（2026/10/15 定案，與其他終止結算的稿件下載規則一致）。
9. **已解決**（2026/10/15）：終態案件的聊天室唯讀，可看歷史訊息、不能再傳訊。
10. **已同步**（2026/10/15，已由 Claude 直接改 FigJam 的欄位說明，欄位名稱 `finalize_due_at` 一併改好；便利貼 4.1～4.4 的內容已於 2026/10/17 依本文 4.1～4.4 改寫同步）。同步項目如下：①`ORDER.topup_due_at` 改名 `finalize_due_at`，說明改為「完稿定稿／補款期限到期時間（乙方最近一次上傳完稿起算 7 天）」，原說明「最後階段確認前」已過時②`ORDER_STAGE.submitted_at` 說明「auto-confirm 與棄單判定起算」已過時（auto-confirm 與惡意棄單機制 10/12 已拿掉），改為「乙方最近一次上傳時間（重傳覆蓋）；完稿的上傳時間是 7 天期限起算點」③`COMMISSION_ITEM.stage_plan_pref`、`ORDER.stage_plan_snapshot` 的合法值「單段」改為「三點分段」④`ORDER_STAGE.stage_no` 說明「單段僅 5」改為「三點分段 1／4／5」⑤`PAYMENT.type` 新增「補付入款」⑥`ORDER.cancel_reason` 說明已是「取消／終止原因」，不用改⑦便利貼 4.2、4.3 對應本文 4.2、4.3 的更新。

11. **已同步**（2026/10/17）：①`ORDER` 新增 `item_description_snapshot`（項目說明快照）②新增表 `ORDER_ITEM_SNAPSHOT_IMAGE`（`order_id`＋`seq_no`、圖片網址），並畫「項目圖片快照」關聯線③`ORDER.item_id` 改為 nullable，說明加「項目刪除後為空，案件一律讀快照」。
12. **已同步**（2026/10/17）：`CREATOR_PROFILE.user_id` 補標 `UK`（對應 1 對 0..1）；`ORDER.payment_plan` 合法值由「全額／訂金」改為「一次付清／先付訂金」，與其他文件用詞一致。

另外 `延伸檔案/ER模型-實體初步草案.md` 是較早的文字草案，若與 FigJam 不一致，以 FigJam 為準。
