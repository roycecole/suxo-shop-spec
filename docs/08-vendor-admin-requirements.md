# 08 - 賣家後台需求 (Vendor Admin Requirements)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分出後台細節需求，回應「賣家上架/數據/版型/自家功能開關」與「拾夜科技資訊人員權限」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增第 6 節：後台匯出 WooCommerce 商品 CSV，回應「方便廠商移轉 WooCommerce」需求，含實際 WooCommerce CSV 欄位對照與變體展開規則 |
| v0.3 | 2026-09-08 | ordinarycas | 新增第 7 節：RWD/PWA 要求，指向 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md)（後台不強制無障礙規範，僅前台） |
| v0.4 | 2026-09-08 | ordinarycas | §1 補充內容編輯採 Markdown 編輯器，回應「後台內容編輯使用 Markdown」需求 |
| v0.5 | 2026-09-08 | ordinarycas | §1 補上 Promotions/Shipping/Reviews 三個服務的賣家後台管理介面需求（後端 API 已在各自服務文件補齊，見 [10-gap-analysis.md](10-gap-analysis.md) §12）；§2 移除 `StoreSettings.GuestCheckoutEnabled`——訪客結帳是 [07-storefront-requirements.md](07-storefront-requirements.md) §1 訂定的平台鎖定硬性需求，不應做成賣家可自行關閉的功能開關，此為前一輪複查發現的矛盾（見 10-gap-analysis.md §12） |
| v0.6 | 2026-09-08 | ordinarycas | §4.3 移除角色列舉裡的「Admin」——[11-service-identity.md](11-service-identity.md) §2 目前定案的 `User.Role` 枚舉本來就沒有這個值，[05-scope-and-open-items.md](05-scope-and-open-items.md) 也明確排除本輪的平台管理員規格，此處純屬文字誤植，非保留給未來的角色 |
| v0.7 | 2026-09-10 | ordinarycas | §6 稅務欄位待決議項已解決：定案不新增，維持匯出固定值，回應「將待決議事項列出來實作」需求 |
| v0.8 | 2026-09-10 | ordinarycas | §5.3 訂正 Grouped 商品匯出分隔符號錯誤（逗號→`|`，查證 WooCommerce 官方格式後發現）；§6 解決其餘 5 項待決議：PlatformSupportStaff 診斷端點盤點（順手補上 Promotions/Notification 兩個遺漏端點）、即時通知定案不需要、會員資訊遮罩顯示定案、匯出連結時效定案 30 分鐘，回應「將待決議事項列出來實作」需求 |
| v0.9 | 2026-09-10 | ordinarycas | 新增 §4.6：§6 盤點出的 5 個服務／6 個診斷端點先前在 `ecommerce-services` 完全不可達（誤掛服務身分 JWT 專用的 `InternalAny` Policy，Gateway 路由表也整個排除 `/internal/v1/*`），本輪真正修正——補上 `PlatformSupportStaffOnly` Policy、Gateway 新增對應具名路由（[25-service-gateway.md](25-service-gateway.md) §4.2）、落實 §4.4 稽核要求的結構化 log 欄位；已用真實簽發的使用者 JWT 經 Gateway 實測驗證。回應「查核 §6 盤點出的端點是否真的能被呼叫到」的發現 |

> 針對「爸芭樂」微服務平台的賣家角色具體化，並新增拾夜科技支援權限章節（第 4 節）。不含客戶自己的「平台管理員」規格（賣家審核、全站金流物流設定、客訴仲裁）——爸芭樂案例暫定為單一賣家自營，見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2。

## 1. 賣家（爸芭樂店主）核心功能

| 功能 | 對應服務 |
|---|---|
| 商品上架/編輯（品種、規格、價格、圖片） | Catalog Service、Media Service |
| 庫存管理（入庫、盤點、批次/有效期） | **WMS Service** |
| 訂單處理（確認、出貨、標記已完成） | Order Service |
| 銷售數據（銷售趨勢、熱銷品種排行） | Analytics Service |
| 首頁/形象頁版型編輯 | CMS Service |
| 優惠券管理（新增/編輯/刪除，設定折扣內容、期限、適用範圍） | **Promotions Service**（[16-service-promotions.md](16-service-promotions.md) §5） |
| 運費規則設定（新增/編輯/刪除運費區域與物流方式） | **Shipping Service**（[21-service-shipping.md](21-service-shipping.md) §4） |
| 評價管理（檢視商店所有評價、回覆買家評價） | **Reviews Service**（[24-service-reviews.md](24-service-reviews.md) §4） |

商品描述（Catalog）與首頁形象內容（CMS 的 RichText 區塊）**皆採 Markdown 編輯器**（如 `react-md-editor` 之類，含即時預覽），不提供所見即所得的 HTML 富文字編輯器——理由是 Markdown 格式簡單、跨服務儲存/轉譯一致（見 [12-service-catalog.md](12-service-catalog.md)、[20-service-cms.md](20-service-cms.md)），也降低賣家貼入未過濾 HTML 造成 XSS 的風險面。

## 2. 自家系統功能開關（賣家自己管理，與 ShyeCMS 無關）

**這是賣家對「自己這間店」的功能開關，不是 ShyeCMS 對客戶的功能授權**——兩者容易混淆，需明確區分：

| | ShyeCMS 的 `ClientFeatureEntitlement`（見 [02-data-model.md](02-data-model.md)） | 本節：賣家後台的 StoreSettings |
|---|---|---|
| 誰設定 | 拾夜科技員工，依合約 | 爸芭樂店主自己，在自己的後台介面 |
| 儲存位置 | ShyeCMS 自己的資料庫 | 爸芭樂平台自己的資料庫（如 `StoreSettings` 表，掛在 Vendor Service 或 CMS Service 下） |
| 影響範圍 | 決定「合約允許用哪些功能」（上限） | 在合約允許的範圍內，決定「目前要不要啟用」（實際開關） |
| 是否連線 | 不連線（決策 C） | 就是本地資料庫的一筆設定，本來就不需要連線 |

範例：合約（`ClientFeatureEntitlement`）允許爸芭樂使用「優惠券模組」，但爸芭樂店主自己在後台可以選擇「目前不啟用」（`StoreSettings.CouponModuleEnabled = false`），這是店主自己的營運決定，跟拾夜科技沒有關係。

### StoreSettings（示意）
| 欄位 | 型別 | 說明 |
|---|---|---|
| CouponModuleEnabled | bool | 是否開放使用優惠券 |
| CodPaymentEnabled | bool | 是否開放貨到付款 |
| ReviewsVisible | bool | 是否於商品頁顯示評價 |

> **不含訪客結帳開關**：訪客結帳是 [07-storefront-requirements.md](07-storefront-requirements.md) §1 明訂的**平台鎖定硬性需求**（「不是可選功能」），與本節其餘項目性質不同——其餘都是賣家在合約允許範圍內自行決定要不要用的營運選項，訪客結帳則是不論賣家意願都必須提供的能力，因此不應該、也不會出現在 `StoreSettings` 這種賣家可自行關閉的開關清單裡。

## 3. 賣家角色與子帳號

沿用 `VendorStaff` 設計（子帳號權限，如僅出貨、僅看報表），本輪不重新設計，只確認此設計掛在 **Vendor Service** 底下。

## 4. 拾夜科技資訊人員的支援權限（新增）

### 4.1 目的

拾夜科技需要在客戶回報系統異常時，能夠進入該客戶的平台調查問題（如訂單卡住、金流回調失敗、庫存數字異常）。

### 4.2 與 ShyeCMS 零連接原則的界線

這**不是** ShyeCMS 與客戶平台的技術連接（決策 C 仍然成立：ShyeCMS 本身不會呼叫客戶平台的任何 API，也不會被客戶平台呼叫）。這是**客戶自己平台的 Identity Service 裡，多一種角色**，供拾夜科技的支援人員在需要時登入使用——性質上等同任何 SaaS 廠商的「客服後台支援帳號」，帳號本身、登入方式、權限範圍都由**每個客戶平台自己的 Identity Service** 管理，彼此獨立，不透過 ShyeCMS 做任何形式的單一登入或帳號同步。

### 4.3 角色設計

延伸拾夜科技「System Vendor Ops（系統商維運人員）」角色，在 Identity Service 具體化為：

| 欄位/設計 | 說明 |
|---|---|
| Role = `PlatformSupportStaff` | 新增於 Identity Service 的角色列舉，與 Buyer/Seller/SellerStaff 並列（見 [11-service-identity.md](11-service-identity.md) §2；本輪不含「Admin」角色，見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2） |
| 帳號建立方式 | 部署當下由維運人員在該客戶環境**手動建立**，不透過 ShyeCMS 下發，也不與其他客戶環境共用帳密 |
| 權限範圍 | 預設**唯讀**：可查看訂單狀態、系統錯誤紀錄、金流回調紀錄、庫存異動紀錄；**不可**匯出會員個資清單、不可修改商品價格或商業設定 |
| 特殊診斷操作 | 少數需要寫入的診斷操作（如手動重試卡住的訂單狀態機）需個別列為白名單動作，且動作本身要能被追蹤（見 4.4） |
| 存取方式 | 一般登入頁面 + 角色授權，不需要獨立的後門或隱藏入口 |

### 4.4 稽核要求

所有 `PlatformSupportStaff` 的操作，比照 `AuditLog` 設計，額外標記 `IsSystemVendorAccess = true`，讓客戶（賣家/平台管理員）自己也能在稽核紀錄裡看到「這是拾夜科技支援人員的操作」，避免對客戶不透明。

### 4.5 明確排除

- 此角色**不提供**跨客戶查看能力——一個 `PlatformSupportStaff` 帳號只能存取它所屬的那一個客戶環境，不存在拾夜科技用一組帳號登入所有客戶站台的機制。
- 此角色**不是** ShyeCMS 的功能，也不會讓 ShyeCMS 因此間接取得客戶資料——決策 D（不取得客戶商品/售價/會員資料）依然成立，這裡的資料存取行為完全發生在客戶自己的環境內，由客戶自己的 AuditLog 留痕，資料不會回傳到 ShyeCMS。

### 4.6 授權與路由實作（2026-09-10 新增）

§6 盤點出的 5 個服務、6 個診斷端點（見該節表格）**曾經完全不可達**：`ecommerce-services` 程式碼原本誤把這些端點掛在 `InternalAny` Policy 下（驗證的是服務身分 JWT，服務對服務呼叫專用），真人 `PlatformSupportStaff` 使用者拿自己登入取得的使用者 JWT 完全無法通過；Gateway 路由表又整個排除 `/internal/v1/*`（[29-shared-service-conventions.md](29-shared-service-conventions.md) §3），即使角色驗證修好，也沒有對外入口能把請求轉發過去。這個落差從骨架階段存在到本輪才被發現並修正（詳見 [10-gap-analysis.md](10-gap-analysis.md) 對應項目的訂正說明），過程記錄於此，避免之後又被誤判為「已解決」。

真正落地的三件事：

1. **授權**：`SuxoShop.Shared.Security` 新增 `PlatformSupportStaffOnly` Policy（驗證使用者 JWT 的 `Role == PlatformSupportStaff`，比照既有 `VendorScoped` Policy 的模式），6 個端點全數由 `InternalAny` 改掛此 Policy；Notification Service 原本完全沒有註冊使用者 JWT 驗證，一併補上。
2. **路由**：Gateway 新增 6 條具名路由（`/api/v1/support/...`，不經過 `/api/public/v1` 的 API 金鑰驗證），逐一精確轉發到對應的 `internal/v1/.../support/...` 路徑，是「`internal/v1/*` 不對外路由」原則唯一、範圍鎖死的例外，細節見 [25-service-gateway.md](25-service-gateway.md) §4.2。
3. **稽核**（落實 §4.4 要求）：5 個服務收到請求時皆輸出一行結構化 JSON log，固定欄位 `IsSystemVendorAccess`（固定 `true`）、`StaffUserId`、`StaffEmail`、`Endpoint`、`ResourceType`、`ResourceId`、`VendorId`、`AccessedAtUtc`——目前**不是**獨立的 `AuditLog` 資料表（那是遠比修正這 6 個端點更大的工程，超出本輪範圍，未來若要新增，欄位命名應延續這裡的既有慣例），而是沿用各服務已接線的結構化 log 管線（[29-shared-service-conventions.md](29-shared-service-conventions.md) §1.3），供集中收集後查詢。

驗證方式：5 個服務各自新增 HTTP 層整合測試（合法 `PlatformSupportStaff` Token 應得 200 + 真實資料、角色不符 Token 應得 403、無 Token 應得 401），Gateway 新增路由轉發測試；另外以真實簽發的使用者 JWT，經由 Gateway（非直連服務）對 `docker compose` 實際執行中的環境（真實 Postgres）發出真實請求，確認回傳真實資料，且對應服務的 log 裡真的寫入了上述稽核欄位。

## 5. 資料可攜出：匯出 WooCommerce 商品 CSV

### 5.1 目的

賣家後台提供「匯出 WooCommerce 商品 CSV」功能，讓客戶（如爸芭樂）若要移轉到 WooCommerce，能直接把商品資料匯出成 WooCommerce 官方 CSV Importer 認得的格式，不需要人工重新輸入。這也解決了「客戶終止合作時商品資料如何交還」這個問題——但不含會員/訂單資料與完整資料庫備份的交還（見 5.6）。

### 5.2 欄位對照表

依 [WooCommerce 官方文件](https://woocommerce.com/document/product-csv-importer-exporter/)核實的實際欄位名稱，對照我們系統的資料來源：

| WooCommerce CSV 欄位 | 我們的資料來源 | 轉換規則 |
|---|---|---|
| `SKU` | Catalog.Product.SKU | 直接複製 |
| `Name` | Catalog.Product.Name | 直接複製 |
| `Type` | Catalog.Product.Type | `Simple→simple`、`Variable→variable`、`Grouped→grouped` |
| `Description` | Catalog.Product.Description | 直接複製（保留 HTML） |
| `Short description` | Catalog.Product.ShortDescription | 直接複製 |
| `Regular price` | Catalog.Product.RegularPrice | 直接複製 |
| `Sale price` | Catalog.Product.SalePrice | 為 null 時留空 |
| `Date sale price starts` / `ends` | *（我們的資料模型目前沒有特價期間欄位）* | 留空；若未來需要，Catalog.Product 需新增 `SalePriceStartAt`/`EndAt` |
| `Tax status` / `Tax class` | *（我們的資料模型目前沒有稅務欄位）* | 固定匯出 `taxable` / 空白稅別，並在匯出頁面提示賣家匯入後需自行到 WooCommerce 核對稅務設定 |
| `Stock` | WMS.Inventory.StockQuantity | 直接複製 |
| `In stock?` | WMS.Inventory.StockQuantity > 0 | 換算為 `1`/`0` |
| `Backorders allowed?` | WMS.Inventory.BackorderPolicy（**新增欄位**，見 5.4） | `Allowed→1`、`AllowedWithNotify→notify`、`NotAllowed→0` |
| `Weight (unit)` / `Length (unit)` / `Width (unit)` / `Height (unit)` | Catalog.Product.Weight/Length/Width/Height | 直接複製，匯出頁面提示賣家核對單位（公斤/公分）是否與 WooCommerce 商店設定一致 |
| `Categories` | Catalog.Category（多對多） | 依 CategoryId 查出名稱，父子分類用 `>` 連接，多筆分類用逗號分隔（階層分類設計） |
| `Tags` | Catalog.Tag（多對多） | 依 TagId 查出名稱，逗號分隔 |
| `Images` | Media.ProductImage（依 SortOrder） | 依 `SortOrder` 排序後的圖片 URL 合併成一個逗號分隔字串，**排序第一者即為主圖**（WooCommerce 沒有獨立的主圖/附圖欄位，只有這一欄） |
| `Parent`、`Attribute 1 name`、`Attribute 1 value(s)` 等 | Catalog.ProductVariation | 見 5.3 變體展開規則 |

> 含逗號的欄位值（如商品名稱裡有逗號）需依 WooCommerce 規則以反斜線轉義（`\,`），CSV 產生器需處理這個跳脫規則。

### 5.3 變體（Variable）商品的展開規則

WooCommerce CSV 用「多列」表示一個變體商品：第一列是父商品（`Type=variable`），之後每個變體各一列（`Type=variation`，`Parent` 欄填父商品的 SKU）。匯出邏輯：

1. `Product.Type = Variable` 的商品，先輸出一列 `Type=variable` 的父列（不含庫存/價格，這些留給變體列）。
2. 依 `Product.ProductVariation` 逐筆輸出子列：`Type=variation`、`Parent={父商品SKU}`、`SKU={變體SKU}`、`Regular price`/`Stock` 取自該變體。
3. 變體對應的屬性（如「顏色=紅、尺寸=M」）需要先在父列宣告 `Attribute 1 name`/`Attribute 2 name`（即 `ProductAttribute.Name`），子列填對應的 `Attribute 1 value(s)` 等（即 `ProductAttributeValue.Value`）。

`Product.Type = Grouped` 的商品，WooCommerce 對應欄位是 `Grouped products`（填組內商品 SKU）。**分隔符號已核對訂正**：原先假設逗號分隔，經查證 WooCommerce 官方匯出格式實際用**直線符號 `|`分隔**（如 `SKU1|SKU2|SKU3`；若以 ID 對應則為 `id:100|id:101`），逗號是錯的假設，匯出實作需採 `|` 分隔，否則匯入 WooCommerce 端會整串被當成一個 SKU 而失敗。

### 5.4 資料模型的必要新增

匯出功能需要 WMS Service 新增一個目前不存在的欄位：

| 新增欄位 | 位置 | 說明 |
|---|---|---|
| `BackorderPolicy` | WMS.Inventory | enum：`NotAllowed` / `Allowed` / `AllowedWithNotify`，對應 WooCommerce 的 `Backorders allowed?` 三種值。我們目前只有 `StockStatus`（InStock/OutOfStock/OnBackorder）是庫存的**當下狀態快照**，不是「是否允許缺貨下單」的**政策設定**，兩者語意不同，不能直接借用 |

### 5.5 匯出流程（非同步作業）

商品量多時全量匯出耗時，比照已驗證的背景工作模式（縮圖佇列 Worker）設計為非同步：

1. 賣家在後台點擊「匯出 WooCommerce CSV」→ `POST /api/v1/vendor/products/export/woocommerce`（見 [12-service-catalog.md](12-service-catalog.md)），Catalog Service 建立一筆匯出工作紀錄，立即回應 `202 Accepted` + `jobId`。
2. 背景 Worker 依序處理：讀取該賣家所有商品（Catalog）→ 逐筆呼叫 WMS 取得庫存與 `BackorderPolicy`→ 呼叫 Media 取得排序後圖片 URL → 組成 CSV → 上傳到物件儲存（沿用既有 Storage 抽象層）。
3. 完成後更新工作狀態為 `Done` 並產生一個**有時效性的下載連結**；賣家可輪詢 `GET /api/v1/vendor/products/export/woocommerce/{jobId}` 查詢進度，或收到站內通知。

### 5.6 明確排除

- **不含會員資料匯出**——延續決策 D 的精神（即使這裡匯出對象是賣家自己而非 ShyeCMS，仍應避免把會員個資做成可攜出檔案，降低外洩風險），會員資料的搬遷不在此功能範圍。
- **不含訂單歷史匯出**——WooCommerce 訂單 CSV 格式與商品 CSV 完全不同結構，且訂單搬遷牽涉更複雜的財務對帳問題，本輪不處理。
- **不含 WooCommerce 反向匯入**（把 WooCommerce 資料匯入我們平台）——本功能單向解決「離開時怎麼帶走商品資料」，不處理新客戶從 WooCommerce 搬過來的匯入需求（那是另一個獨立功能，可比照本文件的欄位對照表反向設計，但需另外規劃）。

## 6. 待決議事項
- [x] ~~`PlatformSupportStaff` 的白名單診斷操作清單需要逐服務盤點~~——**已解決（盤點完成，順手補上 2 個發現的缺漏端點）**：

| 服務 | 現有診斷端點 | 用途 |
|---|---|---|
| Order | `GET /internal/v1/orders/support/{id}/trace`、`GET /internal/v1/orders/support/compensation-failures` | 查 Saga 執行軌跡、查待人工介入的補償失敗清單 |
| WMS | `GET /internal/v1/wms/support/stock-ledger/{productId}` | 查庫存異動歷程 |
| Payment | `GET /internal/v1/payments/support/{orderId}/callback-log` | 查金流回調紀錄 |
| Promotions | `GET /internal/v1/promotions/support/{code}/usage-log`（**本次補上，原本遺漏**） | 查優惠券使用/還原歷程 |
| Notification | `GET /internal/v1/notifications/support/failed-log`（**本次補上，原本遺漏**） | 查重試 4 次仍失敗的 LINE 推播清單 |
| Cart／Catalog／Media／CMS／Analytics／Shipping／Vendor／Reviews／Gateway／Identity | 無 | 這些服務沒有「補償/重試會卡住」的非同步失敗狀態（多為同步 CRUD），目前沒有已知需要診斷端點的情境，維持沒有 |

盤點依據：本規格庫已設計的 Saga 補償統一模式（[17-service-order.md](17-service-order.md) §4.1）——凡是 Saga 補償參與者（WMS、Promotions）或有自己重試機制的非同步流程（Payment 回調、Notification 推播），都該有唯讀診斷端點；純同步 CRUD 服務目前沒有對應需求。Promotions 與 Notification 原本各自遺漏，已在 [16-service-promotions.md](16-service-promotions.md)、[23-service-notification.md](23-service-notification.md) 補上（含 Promotions 新增 `CouponUsageLog` 實體支撐查詢）。**這些端點如何被授權存取、如何路由、如何稽核，見 §4.6**——本表只列「有哪些端點」，實作落地過程另有一段先誤判為已解決的插曲，記錄在該節。
- [x] ~~是否需要要求 `PlatformSupportStaff` 存取時客戶端能即時看到通知~~——**已解決：現階段不需要，AuditLog 已足夠**。理由：即時通知需要額外的推播管道（比照 [23-service-notification.md](23-service-notification.md) 的 LINE 推播基礎設施，或另建 Email 通知），對「支援人員查看一筆訂單」這種相對低風險、高頻率的操作即時通知客戶，效益與開發成本不成比例，且可能造成不必要的客戶焦慮（每次支援人員例行查詢都推播，反而稀釋掉真正異常存取的警示效果）。既有的 `AuditLog` 已完整記錄每次存取的人員/時間/對象，客戶如有疑慮可要求提供存取紀錄，事後可稽核已滿足透明度需求，不需要做成即時推播
- [x] ~~唯讀範圍是否需要對會員 Email/電話做遮罩顯示（如 `t***@example.com`）而非完全不可見~~——**已解決：需要，預設遮罩顯示**。`PlatformSupportStaff` 查詢會員相關資料（訂單詳情、Saga trace 等）時，Email 顯示為 `t***@example.com`（保留首字元＋網域）、電話顯示為 `09XX-XXX-123`（保留區碼與末3碼）格式，兼顧「支援人員能辨識/核對是不是同一個人」與隱私最小化原則；如遇到真的需要完整聯絡資訊處理客訴的情境（如需要回電），另外設計「檢視完整資訊」的明確動作並寫入 `AuditLog`（誰在何時查看了誰的完整聯絡資訊），不做成預設全露出
- [x] ~~稅務欄位（`Tax status`/`Tax class`）匯出固定值是否足夠，或需要在 Catalog.Product 正式新增稅務欄位~~——**已解決：固定值已足夠，不新增欄位**，理由見 [12-service-catalog.md](12-service-catalog.md) §6
- [x] ~~Grouped 商品的 WooCommerce 語意核對~~——**已解決（且發現並訂正一處實際錯誤）**：查證 WooCommerce 官方 CSV 匯出格式後，發現本文件先前假設的「逗號分隔」是錯的，實際為 `|`（直線符號）分隔，已於 §5.3 訂正——這個差異如果沒抓到，匯出的 CSV 匯入 WooCommerce 時會直接失敗或誤判，是本次「列出待決議事項逐項處理」意外揪出的真實缺陷，不只是走個確認流程
- [x] ~~匯出檔案的下載連結時效與存取權限（比照 Media Service 既有的檔案存取控管機制）~~——**已解決：簽章網址（Signed URL），30 分鐘時效**。WooCommerce CSV 等匯出檔案產生後，回傳一個帶時效簽章的下載連結（比照 Media Service 私有檔案的存取模式），30 分鐘內有效，逾時需重新觸發匯出；不做成永久公開連結，避免連結外流後任何人都能下載商店的商品/訂單資料。與[19-service-media.md](19-service-media.md) 的差異：Media 的簽章網址是給前台公開圖片以外的私有素材用，這裡是給賣家自己觸發的批次匯出檔案用，時效更短（30 分鐘 vs Media 可能需要的更長時效），因為匯出檔案的使用情境是「觸發後立刻下載」，不是「長期保存的連結」

## 7. RWD / PWA

賣家後台須支援響應式設計並做成可安裝的 PWA，讓賣家可在手機上安裝、隨時查看訂單與銷售數據。**不強制**符合 WCAG 2.1 AA（僅前台需要，見 [07-storefront-requirements.md](07-storefront-requirements.md) §6）。完整規格見 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md)。
