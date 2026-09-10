# 10 - 缺口分析與建議調整 (Gap Analysis)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「分析規格還可以怎麼調整或補上什麼」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 因應「微服務拆成多個規格」更新：各服務 API 大綱已補齊（[11](11-service-identity.md)–[25](25-service-gateway.md)），移除已解決項目；修正服務數量（15 個）；移除對已刪除文件的引用 |
| v0.3 | 2026-09-08 | ordinarycas | 回應「分析規格還可以怎麼調整或補上什麼」第二輪分析：新增 §1 跨服務共通慣例、部署可觀測性、CI/CD 等技術缺口；新增 §5 設計/前端面（因應 CSS 動畫決策浮現的設計系統缺口）；更新建議下一步 |
| v0.4 | 2026-09-08 | ordinarycas | 回應「確認專案清單」後第三輪分析：新增 §6 ShyeCMS 專案面——確認 ShyeCMS 技術棧（[00](00-overview.md) 決策 F、[26-project-structure.md](26-project-structure.md)）後發現 ShyeCMS **完全沒有前端頁面/操作流程規格**，是本輪最重要的新發現 |
| v0.5 | 2026-09-08 | ordinarycas | 新增 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md) 後，設計系統文件建議編號由 `27` 改為 `28`（見 §5） |
| v0.6 | 2026-09-08 | ordinarycas | 回應「分析規格還可以怎麼調整或補上什麼」第四輪分析：新增 §7 多語系/圖表/Markdown 決策的連鎖影響；設計系統文件建議編號再改為 `29`（`28` 已被 [28-i18n.md](28-i18n.md) 使用） |
| v0.7 | 2026-09-08 | ordinarycas | 解決 §7 列出的三項連鎖缺口：新增 [29-shared-service-conventions.md](29-shared-service-conventions.md)（Markdown 管線、服務間認證、資安基準），Promotions/Shipping/CMS 補上 Translation 表，[06](06-ecommerce-platform-architecture.md) §6.3 重新評估多語系對主機規格影響；標記對應項目為已解決；修正 §6/§7 編號順序；設計系統文件建議編號再改為 `30`（`29` 已被本次新文件使用） |
| v0.8 | 2026-09-08 | ordinarycas | 因應 [26-project-structure.md](26-project-structure.md) 決策 H（repo 拆分、取消共用 `.sln`）更新：§6「Monorepo 建置工具未選型」標記已解決（前提改變，不再是 monorepo）；§6「兩個 repo 各自 CI/CD」擴大為「六個 repo + 跨 repo 版本協調」缺口 |
| v0.9 | 2026-09-08 | ordinarycas | 回應「分析規格還可以怎麼調整或補上什麼」第五輪分析：新增 §9 Repo 拆分帶來的新缺口（本機多 repo 開發流程、共用函式庫資安修補傳播機制、api-client 版本相容性、私有倉庫維運成本）；更新建議下一步 |
| v0.10 | 2026-09-08 | ordinarycas | 回應「分析規格還可以怎麼調整或補上什麼／待決議事項」第六輪分析：新增 [30-open-decisions-register.md](30-open-decisions-register.md) 彙整全部 77 項待決議；發現並修正 [16-service-promotions.md](16-service-promotions.md) 缺少待決議章節；新增 [28-i18n.md](28-i18n.md) 幣別/金流在地化排除說明；設計系統文件建議編號再改為 `31`（`30` 已被開放決議總表使用） |
| v0.11 | 2026-09-08 | ordinarycas | 回應「分析該專案規格文件檢查可以優化和補充的部分」第七輪跨文件複查（多份文件同時比對，非單一新決策觸發）：新增 §10 微服務 API/資料模型缺漏、§11 跨文件一致性與決策矛盾、§12 前後台需求規格落差，共 24 項新發現；修正 §6 表格遺漏的 Markdown 開頭 `\|`（純格式 bug）；§4 新增 2 項優先事項。另於 [13-service-wms.md](13-service-wms.md) §6 補上遺漏的「Saga 補償失敗」待決議項目（[30-open-decisions-register.md](30-open-decisions-register.md) 早已假設它存在但源文件其實沒寫），並同步至該總表 |
| v0.12 | 2026-09-08 | ordinarycas | 第八輪複查，補齊前一輪未專門檢查的 [00](00-overview.md)/[01](01-architecture.md)/[05](05-scope-and-open-items.md)/[06](06-ecommerce-platform-architecture.md)/[26](26-project-structure.md)/[29](29-shared-service-conventions.md)：直接修正 3 項客觀錯誤——[26-project-structure.md](26-project-structure.md) §3.1 Identity 範例誤植 Catalog 命名空間、[06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.1/§6.3 重複計算 Gateway 的措辭、[05-scope-and-open-items.md](05-scope-and-open-items.md) §3 少同步 2 項已在 [30-open-decisions-register.md](30-open-decisions-register.md) §2 的既有項目；新增 §13 記錄 1 項需要另外設計、不能直接修正的文件承諾落差 |
| v0.13 | 2026-09-08 | ordinarycas | 應使用者要求，實作 §4/§11/§12 列出的 4 項優先事項並回頭標記已解決：(1) [14-service-vendor.md](14-service-vendor.md) 新增 §4 API 大綱；(2) [25-service-gateway.md](25-service-gateway.md) 新增 §3.1 匿名端點速率限制、[16-service-promotions.md](16-service-promotions.md) 新增 §4 併發保護機制；(3) [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1 補上 Promotions/Shipping/Reviews 賣家後台介面，對應後端 CRUD 補進 [16](16-service-promotions.md)、[21-service-shipping.md](21-service-shipping.md)、[24-service-reviews.md](24-service-reviews.md)；(4) 移除 `StoreSettings.GuestCheckoutEnabled`，[07-storefront-requirements.md](07-storefront-requirements.md) §1 與 [08](08-vendor-admin-requirements.md) §2 補充說明訪客結帳是平台鎖定需求 |
| v0.14 | 2026-09-08 | ordinarycas | 應使用者要求，實作 §10/§11 剩餘的 5 項並回頭標記已解決：(1) [19-service-media.md](19-service-media.md) `MediaAsset` 補上 `VendorId`；(2) [20-service-cms.md](20-service-cms.md) §4 新增賣家草稿讀取端點；(3) [18-service-payment.md](18-service-payment.md) `Payment` 新增 `ProviderTransactionId` 與唯一索引，落實回調去重；(4) [17-service-order.md](17-service-order.md) §4、[06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §7 的結帳 Saga 補上查詢 Vendor Service `CommissionRate` 的步驟，[14](14-service-vendor.md) §4 新增對應內部端點；(5) 明訂 `StoreSettings.CodPaymentEnabled`（[14](14-service-vendor.md) §2）為 COD 開關唯一權威來源，[18](18-service-payment.md) §2 補充說明並新增賣家標記 COD 已收款端點 |
| v0.15 | 2026-09-08 | ordinarycas | 應使用者要求「繼續處理」，收尾 §10/§11 最後 6 項，至此本文件 §10–§12 列出的項目已全數解決：(1) [12-service-catalog.md](12-service-catalog.md) §5 明訂讀取端點回傳已轉換的安全 HTML；(2) [09-api-specification.md](09-api-specification.md) 新增 §3 清單端點分頁慣例；(3) [22-service-analytics.md](22-service-analytics.md) §1 修正「訂閱事件」矛盾措辭；(4) [09-api-specification.md](09-api-specification.md) §2 新增健康檢查端點的統一聲明；(5) [29-shared-service-conventions.md](29-shared-service-conventions.md) §3 新增內部端點認證註記的統一讀法；(6) [17-service-order.md](17-service-order.md) §6 改寫過時的 Correlation ID 待決議項並標記已解決 |
| v0.16 | 2026-09-08 | ordinarycas | 應使用者要求「繼續往下處理」，收尾 §12/§13 全部 7 項，至此本文件 §10–§13 第七/八輪複查列出的所有項目已全數解決：(1) [07-storefront-requirements.md](07-storefront-requirements.md) §3 補齊 Promotions/Reviews 頁面對應；(2) [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §4.3 移除誤植的「Admin」角色；(3) [03-client-lifecycle.md](03-client-lifecycle.md) §5 補上人工同步提醒；(4) [28-i18n.md](28-i18n.md) 新增 §3.1 定義 `SuxoShop.Shared.Translation` 套件內容；(5) [11-service-identity.md](11-service-identity.md) §5 定案帳號刪除採匿名化保留；(6) [23-service-notification.md](23-service-notification.md) §4 補上具體重試參數；(7) [01-architecture.md](01-architecture.md) §3、[06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §9 定案功能開關以 `ecommerce-deploy` 的 `.env` 環境變數存放 |
| v0.17 | 2026-09-09 | ordinarycas | repo 更名 `ecommerce-deploy`→`ecommerce-launch`；§6 ShyeCMS 前端規格/認證方式兩項缺口標記已解決（新增 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)）；§9 共用套件資安修補傳播缺口標記已解決（[29-shared-service-conventions.md](29-shared-service-conventions.md) §4.1）；§7 圖表函式庫缺口標記已解決（[22-service-analytics.md](22-service-analytics.md) §4 採 Chart.js）；§4 建議下一步同步勾選已解決項目；§5 設計系統文件建議編號因 `31` 已被佔用改為 `32` |
| v0.18 | 2026-09-09 | ordinarycas | 回應「分析還有哪些可以調整或修改的」第九輪複查：新增消費者會員登入（[11-service-identity.md](11-service-identity.md) §5.1）與 Saga 補償統一設計（[17-service-order.md](17-service-order.md) §4.1）之後，重新通讀全部 15 個微服務規格，新增 §14 記錄 5 項新發現——最重要的一項是「訪客升級為會員」機制與新增信箱驗證流程之間存在帳號/訂單歷史冒領風險，尚未修正，僅記錄分析結果；§4 新增項目 11 呼應此發現 |
| v0.19 | 2026-09-09 | ordinarycas | 使用者確認「訪客升級會員的自動關聯要等信箱驗證通過」：§14、§4 項目 11 對應項標記已解決，見 [11-service-identity.md](11-service-identity.md) §5.1 的具體設計 |
| v0.20 | 2026-09-10 | ordinarycas | 使用者要求「將 [30-open-decisions-register.md](30-open-decisions-register.md) 73 項未解決列出來實作」，逐輪處理過程中發現本文件多處已與實際規格/程式碼脫節（如 `docker-compose.yml` 骨架其實早已完成、Catalog/WMS 降級行為的架構前提已不成立），完整重新核對 §1–§4、§6、§8、§9、§14：共標記 23 項已解決（含本輪新增 2 項技術決策——`PlatformSupportStaff` 統一授權 Policy 慣例、網域/SSL 憑證管理採 Caddy 自動 HTTPS、依賴套件掃描納入 CI 用 `dotnet list package --vulnerable`），§4/§6/§8/§9/§14 的「建議」小結同步更新；本節列出的 8 項已全數處理完畢的既有小結（§10–§12）未受影響。剩餘真正還開放的項目：§2 ShyeCMS 監控告警管道空白、§3 版本相容性矩陣/Deprecation Window 執行機制/服務資源評估（後者已在 [30-open-decisions-register.md](30-open-decisions-register.md) 標記需要實測）、§5 設計系統文件（範疇比本輪已解決的斷點/對比更大，色彩/字級/動畫 token 仍缺）、§9 本機多 repo 開發流程／`api-client` 相容性矩陣、§2 ShyeCMS 監控——這些留待下一輪或建置團隊成形後處理 |
| v0.21 | 2026-09-10 | ordinarycas | 訂正 §1「單一 VPS 的備份/災難復原策略空白」一列：發現這項先前標記的「已解決」只是設計文件（[06](06-ecommerce-platform-architecture.md) §6.5）層級，`ecommerce-services`／`ecommerce-launch` 的 `docker-compose.yml` 從未真的包含備份機制——本輪補上真正的實作（`postgres-backup` sidecar + `scripts/pg-backup.sh`）並實測「排程備份→保留輪替→還原至全新 Postgres」全流程成功，見該列更新後的說明與 [06](06-ecommerce-platform-architecture.md) §6.5「已落地與驗證」；異地存放維持原設計、仍待部署時決定，非本輪範圍。回應「檢查 06 §6.5 是否真的落地」的發現 |
| v0.22 | 2026-09-10 | ordinarycas | §1 新增一項發現：`SuxoShop.Shared.Security` 補上 JWT 雙金鑰輪替支援（[29-shared-service-conventions.md](29-shared-service-conventions.md) §3.1 新增）過程中，發現 Cart Service 的使用者 JWT 驗證繞過共用套件、手刻邏輯未套用此次輪替支援，記錄為待修正項目，非本輪範圍 |
| v0.23 | 2026-09-10 | ordinarycas | §1 新增一項發現：修正 Catalog WooCommerce 匯出的 N+1 內部呼叫問題（[12](12-service-catalog.md) §5、[13](13-service-wms.md) §5、[19](19-service-media.md) §6 新增批次端點）過程中，發現 `MediaAsset` 從未有欄位關聯到 Product，WooCommerce 匯出圖片網址欄位從骨架階段至今實際上從未真正輸出過資料——記錄為獨立待決議項目，非本輪範圍 |
| v0.24 | 2026-09-10 | ordinarycas | **訂正 §1「`PlatformSupportStaff` 的跨服務授權模式」一列**：v0.20 標記的「已解決」查證後是**誤判**——當時只確認了 Policy 命名慣例的設計決策，從未真的檢查 5 個診斷端點程式碼；實際重新盤點 `ecommerce-services` 發現全部 5 個端點當時仍掛 `[Authorize(Policy = "InternalAny")]`（服務身分 JWT，服務對服務呼叫專用），真人 `PlatformSupportStaff` 使用者拿自己的使用者 JWT 完全無法通過，且 Gateway 路由表整個排除 `/internal/v1/*`（[29](29-shared-service-conventions.md) §3），即使角色驗證修好，Gateway（唯一對外入口）也沒有任何路徑能把請求轉發過去——功能從骨架階段至今對真人使用者完全不可達，是本文件目前為止唯一一項「記錄為已解決、但實際從未落地」的項目，已在本輪查出並真正修正，見該列更新後的說明 |

> 本文件分析 [00-overview.md](00-overview.md)–[30-open-decisions-register.md](30-open-decisions-register.md) 目前規格的缺口，供下一輪規劃排優先序。

## 1. 技術面

| 項目 | 說明 |
|---|---|
| ~~Correlation ID 貫穿追蹤~~ | **已解決**：Gateway 產生/沿用 `X-Correlation-Id`，往下游強制傳遞，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §1.1 |
| ~~Catalog 與 WMS 的一致性~~ | **已解決（釐清：實際架構不存在這個耦合）**：`ecommerce-storefront` 前台是直接呼叫 WMS 的 `GET /api/v1/wms/products/{productId}/availability`，不透過 Catalog 中介，兩者互不依賴；WMS 查詢失敗時前台已有明確定義的四態 UI（`loading`/`in-stock`/`out-of-stock`/`error`），見 [13-service-wms.md](13-service-wms.md) §6 |
| ~~`PlatformSupportStaff` 的跨服務授權模式~~ | **已解決（v0.24 訂正：真正落地，非僅設計決策）**：[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §6 已完成診斷端點逐服務盤點（Order/WMS/Payment/Promotions/Notification 共 5 個服務、6 個端點，其餘服務目前無此需求）。**誠實揭露**：v0.20 曾標記本項「已解決」，但那次只確認了 Policy 命名慣例的設計決策，未實際核對 `ecommerce-services` 程式碼——真正檢查後發現 5 個端點全數仍掛 `[Authorize(Policy = "InternalAny")]`（服務身分 JWT，服務對服務呼叫專用），且 Gateway 路由表整個排除 `/internal/v1/*`，兩個問題疊加導致功能從骨架階段至今對真人 `PlatformSupportStaff` 使用者完全不可達。本輪（2026-09-10）真正修正三件事：(1) `SuxoShop.Shared.Security` 新增 `AddPlatformSupportStaffOnlyPolicy`（檢查使用者 JWT 的 `Role == PlatformSupportStaff`，比照既有 `AddVendorScopedPolicy` 模式），5 個端點全數由 `InternalAny` 改掛 `PlatformSupportStaffOnly`（Notification 服務原本連使用者 JWT 驗證都沒註冊，一併補上）；(2) Gateway 新增 6 條具名路由（`support-wms-stock-ledger`、`support-promotions-usage-log`、`support-notifications-failed-log`、`support-orders-trace`、`support-orders-compensation-failures`、`support-payments-callback-log`），逐條精確比對路徑轉發到對應的 `internal/v1/.../support/...`，刻意不用萬用比對（避免誤連到 Order 服務同樣掛在 `support/` 底下、但其實是 Analytics 服務對服務端點的 `support/completed`），路由掛在 `/api/v1/support/...`（不需要 `/api/public/v1` 的 API 金鑰驗證），見 [25-service-gateway.md](25-service-gateway.md)；(3) 新增 `PlatformSupportStaffAuditLogger` 共用稽核 log 寫法（`SuxoShop.Shared.Security`），5 個端點呼叫時皆輸出一行結構化 JSON log（`IsSystemVendorAccess`/`StaffUserId`/`StaffEmail`/`Endpoint`/`ResourceType`/`ResourceId`/`VendorId`/`AccessedAtUtc`），落實 [08](08-vendor-admin-requirements.md) §4.4 的稽核要求。**驗證方式**：5 個服務各自新增 HTTP 層整合測試（合法 Token+200+真實資料、角色不符 Token+403、無 Token+401）、Gateway 新增路由轉發測試；另外重建 `wms`/`promotions`/`notification`/`payment`/`gateway`/`order` 六個服務的 docker 映像檔並在真實執行中的 docker-compose 環境（含真實 Postgres）以真實簽發的 `PlatformSupportStaff` 使用者 JWT，經由 Gateway（非直連服務）實測 `stock-ledger`／`usage-log` 兩個端點，確認回傳真實資料且寫入真實稽核 log。 |
| ~~內部 API 的服務間認證機制~~ | **已解決**：網路隔離（`/internal/v1/...` 只綁定 Docker 內部網路）+ 服務身分 JWT 縱深防禦，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §3 |
| ~~多倉支援~~ | **已解決**：現階段明確排除，單一倉庫，見 [13-service-wms.md](13-service-wms.md) §6 |
| ~~效期商品的自動下架/促銷~~ | **已解決**：新增每日背景排程，3 天內到期標記「即期品」、已過期扣除可售庫存，見 [13-service-wms.md](13-service-wms.md) §6 |
| 整合測試 | 目前只有領域層單元測試，微服務拆分後**更需要**跨服務整合測試（尤其 Saga 補償路徑），本輪未涵蓋 |
| Cart Service 的使用者 JWT 驗證繞過共用套件 | `services/cart` 骨架階段遺留的手刻驗證邏輯，未透過 `SuxoShop.Shared.Security` 的 `AddSuxoShopUserAuthentication` 註冊——本輪（2026-09-10）`SuxoShop.Shared.Security` 補上 JWT 雙金鑰輪替支援（[29-shared-service-conventions.md](29-shared-service-conventions.md) §3.1）後，Cart 因為沒有套用該套件，輪替期間可能出現間歇性 401，且原本就少了共用套件既有的 Audience 檢查。待修正：改用共用套件註冊，補齊對應測試 |
| `MediaAsset` 沒有 Product 關聯欄位，WooCommerce 匯出圖片網址從未真正輸出過資料 | 修正 Catalog WooCommerce 匯出工作的 N+1 內部呼叫問題（[12-service-catalog.md](12-service-catalog.md) §5、[19-service-media.md](19-service-media.md) §6）時發現：`MediaAsset`（見 [19-service-media.md](19-service-media.md) §3/§3.1 ERD）目前沒有任何欄位（如 `ProductId`）能把已上傳的圖片與商品關聯起來，Media 對「這個商品有哪些圖片」的查詢因此從骨架階段至今必然回傳空清單，WooCommerce 匯出 CSV 的圖片網址欄位從未真正有資料——這是比批次化更早就存在、獨立於本輪修正的資料模型缺口。待決議：是否幫 `MediaAsset` 補上 `ProductId`，以及 `VendorMediaController.UploadMedia` 上傳流程如何帶入這個值（賣家上傳圖片時目前的操作流程未要求指定所屬商品） |
| ~~跨服務共通慣例未定義~~ | **已解決**：健康檢查端點、結構化 log 格式、Correlation ID 傳遞規則，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §1 |
| ~~`docker-compose.yml` 骨架尚未撰寫~~ | **已解決（早已完成，本文件沒同步）**：`ecommerce-services` 的 `docker-compose.yml` 已是真實可跑的完整版本，涵蓋全部 15 服務 + Gateway + Postgres，本次「將待決議事項列出來實作」的多輪驗證（如 Catalog 索引 migration 實測）都是直接對這份既有 compose 檔案操作，不是新寫的 |
| ~~單一 VPS 的備份/災難復原策略空白~~ | **已解決（2026-09-10 訂正：先前這裡標記的「已解決」只是設計文件層級，`ecommerce-services`／`ecommerce-launch` 的 `docker-compose.yml` 實際上從未真的包含任何備份機制——本輪發現落差並補上真正的實作）**：`postgres-backup` sidecar（`scripts/pg-backup.sh`，排程 `pg_dump` + 依份數保留輪替，預設 `BACKUP_RETENTION_COUNT=7`）已加進 `ecommerce-services/docker-compose.yml` 與 `ecommerce-launch/docker-compose.yml`，並實測「排程備份→保留輪替正確運作→還原至全新獨立 Postgres 容器→資料完整」全流程成功（兩份 compose 檔案的寫法不完全相同，皆已分別驗證）。還原 SOP 見 `ecommerce-launch/README.md`「資料庫備份與還原」；機制設計取捨（改採自製 sidecar 取代原設想的第三方映像檔 `prodrigestivill/postgres-backup-local`）見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.5「已落地與驗證」。異地存放**仍未落地**，維持 §6.5 原本「留給實際部署時依客戶預算決定」的定位，不是本輪遺漏 |
| ~~網域與 SSL 憑證管理未提及~~ | **已解決**：Gateway（YARP）前面加一層 **Caddy** 做 TLS 終止——Caddy 內建自動 HTTPS（自動向 Let's Encrypt 簽發/續簽憑證，零額外設定），對外只需要客戶把網域 DNS 指到這台 VPS，Caddy 偵測到網域後自動處理憑證，續簽也全自動，不需要另外寫 cron 腳本或安裝 certbot；Caddy 與 Gateway 之間走內部網路的純 HTTP，TLS 只在 Caddy 這一層終止。選 Caddy 而非 nginx+certbot 的理由：Caddy 的自動 HTTPS 是零設定的（nginx+certbot 需要另外寫續簽腳本與 nginx reload 邏輯），對單一 VPS、逐客戶部署的維運模型更省事 |
| ~~CORS 政策未定義~~ | **已解決**：每個客戶部署明確列出允許來源網域，禁止萬用字元，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 |

## 2. 流程/商業面

| 項目 | 說明 |
|---|---|
| ~~StoreSettings 的歸屬服務未定案~~ | **已解決**：定案歸屬 Vendor Service，見 [14-service-vendor.md](14-service-vendor.md) §5 |
| ShyeCMS 零連接後，拾夜科技如何得知客戶異常 | 決策 C/D 確認不連接、不取資料後，拾夜科技完全依賴客戶主動回報問題（工單/電話）才會知道系統異常——這是刻意的取捨（見 [01-architecture.md](01-architecture.md) §4），但代表**沒有任何主動監控告警的管道**，回應時間可能因此更難達成，值得提醒業務/客服團隊 |
| ~~`PlatformSupportStaff` 存取透明度~~ | **已解決：不需要即時通知，AuditLog 已足夠**，理由見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §6 |
| ~~訪客結帳的防詐風險~~ | **已解決：現階段不強制簡訊驗證，採分層防詐**，見 [07-storefront-requirements.md](07-storefront-requirements.md) §5 |

## 3. 維運面

| 項目 | 說明 |
|---|---|
| 各服務版本不同步的實際治理 | [09](09-api-specification.md) §1 已將「各服務版本不同步」訂為常態而非問題，但沒有配套的「版本相容性矩陣」文件，維運人員難以一眼看出「目前這個客戶的 Order v2 是否能跟 WMS v1 相容」 |
| 單一客戶部署的服務數量（15 個）資源評估 | 沿用 [06](06-ecommerce-platform-architecture.md) §6.3 既有待決議，粗估規格未經實測校正 |
| Deprecation Window（3 個月）的實際執行機制 | [09](09-api-specification.md) §1 訂了政策，但沒有工具/流程確保「舊版本到期後真的會被下線」，容易變成口頭政策 |
| ~~CI/CD 策略空白~~ | **已解決**：各服務/repo 各自獨立 GitHub Actions pipeline（非統一單一 pipeline），呼應「各服務版本不同步」的既有設計，見 [26-project-structure.md](26-project-structure.md) §7 |

## 4. 建議下一步（依風險排序）

1. ~~補寫 ShyeCMS 前端需求規格~~——**已解決**：新增 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)，見 §6。
2. ~~撰寫 `docker-compose.yml` 骨架~~——**已解決**：`ecommerce-services` 的 `docker-compose.yml` 早已是完整可跑的版本，本文件先前沒同步到，見 §1。
3. ~~決定 Catalog/WMS 呼叫失敗時的前台降級行為~~——**已解決（釐清架構前提不成立）**：見 §1、[13-service-wms.md](13-service-wms.md) §6。
4. ~~定案 `StoreSettings` 歸屬服務~~——**已解決**：歸屬 Vendor Service，見 [14-service-vendor.md](14-service-vendor.md) §5。
5. ~~單一 VPS 的備份/災難復原策略~~——**已解決**：見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.5。
6. ~~共用函式庫的資安修補傳播機制~~（見 §9）——**已解決**：[29-shared-service-conventions.md](29-shared-service-conventions.md) 新增 §4.1，訂出 7 個日曆天強制升級窗口，區分一般版本更新與資安修補。
7. ~~熱銷排行/付款分布的圖表函式庫選型~~——**已解決**：[22-service-analytics.md](22-service-analytics.md) §4 採 Chart.js（`react-chartjs-2`），與 Lightweight Charts 職責互補。
8. ~~補齊 Gateway 匿名端點速率限制~~——**已解決**：[25-service-gateway.md](25-service-gateway.md) 新增 §3.1，見 §11。
9. ~~補上賣家後台 Promotions/Shipping/Reviews 三個服務的管理介面規格~~——**已解決**：[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1 已補上對應列，見 §12。
10. ~~效期商品自動化、多倉支援、防詐機制、網域/憑證管理、CI/CD、高權限帳號 2FA~~——**已解決**（見各自對應章節）；僅**本機多 repo 開發流程**仍待排入下一輪，屬於開發流程細節，非規格階段必須解決。
11. ~~訪客升級為會員的機制與信箱驗證時機~~（見 §14）——**已解決**：[11-service-identity.md](11-service-identity.md) §5.1 定案沿用同一 `User.Id`＋密碼暫存至驗證通過才生效。
12. ~~§5.1 適用範圍是否涵蓋 Seller/SellerStaff、結帳 Saga 循序圖遺漏 Payment 失敗分支、Notification Email 管道優先度~~（見 §14）——**已解決**：三項皆已定案，見 [11-service-identity.md](11-service-identity.md) §5.1、[17-service-order.md](17-service-order.md) §4、[23-service-notification.md](23-service-notification.md) §7。

> 「補齊其餘服務的 API 大綱」已於後續一輪完成（見 [11-service-identity.md](11-service-identity.md)–[25-service-gateway.md](25-service-gateway.md)），故不再列於本節。

## 5. 設計/前端面（新，因應動畫決策浮現）

| 項目 | 說明 |
|---|---|
| 設計系統 Token 完全缺失 | [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5.1 已決定動畫效果主要用 CSS，但色彩、字級、間距、動畫時長/緩動曲線（`--duration-fast`、`--ease-standard` 等）目前完全沒有正式的設計系統文件定義，前台/後台開發時容易各自硬編碼數值，導致視覺不一致 |
| 無障礙的動畫降級規則未成文 | §5.1 提到需搭配 `prefers-reduced-motion`，但目前只是原則性一句話，沒有具體規則（如降級後動畫時長歸零還是簡化成瞬時切換） |
| 元件庫/UI 框架尚未選型 | 前台/後台目前只確定 Next.js（SSG）與 Vite SPA 兩種前端專案形態，但用什麼元件庫（如 Tailwind CSS、Ant Design、MUI 或全自訂）尚未決定，會直接影響 CSS 動畫怎麼組織（Utility Class vs CSS Module vs CSS-in-JS） |

**建議**：這三項屬於同一個缺口（設計系統文件從缺）的不同面向，建議合併規劃成一份新文件（如 `32-design-system.md`，`26`–`31` 已分別被 [26](26-project-structure.md)、[27](27-pwa-and-accessibility.md)、[28](28-i18n.md)、[29](29-shared-service-conventions.md)、[30](30-open-decisions-register.md)、[31](31-shyecms-frontend-requirements.md) 使用），而不是逐項零星補丁，比照本文件集其餘服務規格的拆分精神。

## 6. ShyeCMS 專案面（因應「確認專案清單」浮現）

確認 ShyeCMS 技術棧（[00-overview.md](00-overview.md) 決策 F）並整理出完整專案結構（[26-project-structure.md](26-project-structure.md)）後，發現一個先前被完全忽略的缺口：

| 項目 | 說明 |
|---|---|
| ~~ShyeCMS 前端頁面/操作流程規格完全空白（最重要）~~ | **已解決**：新增 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)，比照 [07](07-storefront-requirements.md)/[08](08-vendor-admin-requirements.md) 的規格深度補齊 `shyecms-admin` 的頁面清單、操作流程、角色權限矩陣 |
| ~~ShyeCMS 的認證方式未指定~~ | **已解決**：[31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md) §1 定案採 JWT Bearer（獨立簽發，與電商平台零共用）；`SuperAdmin` 是否強制 2FA 當時未隨本項一併定案，現已由 [29-shared-service-conventions.md](29-shared-service-conventions.md) §5 統一解決（定案 TOTP，適用範圍涵蓋 ShyeCMS 的 `SuperAdmin` 與電商平台的 `PlatformSupportStaff`） |
| ~~六個 repo 各自的 CI/CD 與跨 repo 版本協調都未定案~~ | **已解決**：各 repo 獨立 GitHub Actions pipeline，`ecommerce-launch` 版本標籤現階段人工更新，見 [26-project-structure.md](26-project-structure.md) §7 |
| ~~Monorepo 建置工具未選型~~ | **已解決（前提改變）**：電商平台不再是單一 monorepo——前台、後台、15 個服務已拆成三個獨立 repo（[26-project-structure.md](26-project-structure.md) 決策 H），`ecommerce-services` 內部也**不設共用 `.sln`**，改為各服務獨立建置，不需要 Nx/Turborepo 等級的跨語言建置編排工具 |

**建議**：本節 3 項已全數處理完畢。

## 7. 多語系/圖表/Markdown 決策的連鎖影響

這三項決策彼此獨立，但都對既有規格產生了連鎖影響。多數已在本輪解決，逐一列出現況：

| 項目 | 現況 |
|---|---|
| ~~各服務的 Translation 表尚未落實到個別文件~~ | **已解決**：Catalog（[12](12-service-catalog.md)）、CMS（[20](20-service-cms.md)）、Promotions（[16](16-service-promotions.md)）、Shipping（[21](21-service-shipping.md)）都已補上 `Translation` 表 |
| ~~Markdown 渲染/清理邏輯是否共用~~ | **已解決**：統一為共用函式庫 `RenderMarkdownToSafeHtml`，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §2 |
| ~~圖表方案只解決了一部分~~ | **已解決**：[22-service-analytics.md](22-service-analytics.md) §4 補上 Chart.js（`react-chartjs-2`）作為熱銷商品排行/付款方式分布的方案，與 Lightweight Charts 職責互補 |
| ~~i18n × SSG 的建置成本未重新估算~~ | **已解決**：執行期主機規格不受語言數量影響，Next.js 建置步驟改移到 CI/CD 執行，見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.3 |
| ~~`SuxoShop.Shared.Translation` 套件的採用狀態未定案~~ | **已解決**（使用者 2026-09-09 要求「決定去留」）：定案不採用，`ecommerce-services` 的 Catalog/CMS/Promotions/Shipping 四服務維持各自獨立實作，套件降級為參考範本；同時定案 zh-Hant 不落在 `Translation` 表裡，見 [28-i18n.md](28-i18n.md) §3、§3.1 v0.6 |

**建議**：前四項已在當輪全數處理完畢；第五項（套件去留）於後續一輪（2026-09-09，使用者明確要求）補上決議。

## 8. 資安面（因應「資訊安全性非常重要」正式收斂）

本輪新增 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 作為所有服務的資安基準，取代原本散落在各文件裡的零星提及。現況：

| 已定案 | 說明 |
|---|---|
| HTTPS/HSTS、密碼雜湊、SQL Injection 防護、XSS 防護（含 Markdown 管線）、CSRF、CORS 政策、敏感憑證加密、Data Protection 金鑰持久化、API 速率限制範圍、PCI DSS（金流不落地）、服務間認證（網路隔離 + JWT） | 既有 |
| 高權限帳號 2FA（TOTP）、CSP 詳細規則、log 集中收集方案（Grafana Loki）、服務身分 JWT 快取策略（不需要） | 已解決，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §5 |
| ~~依賴套件掃描是否納入 CI~~ | **已解決：納入，用 `dotnet list package --vulnerable`**（.NET 內建工具，不需要額外服務）。本規格庫這幾輪的 `dotnet build`/`dotnet restore` 過程中，已經實際觀察到真實的套件漏洞警告（`NU1902`，如 `HtmlSanitizer`/`AngleSharp` 中度風險），證明這不是假設性風險——CI pipeline（見 §3、[26-project-structure.md](26-project-structure.md) §7 的 GitHub Actions）新增一個步驟跑 `dotnet list package --vulnerable --include-transitive`，發現高/嚴重風險漏洞時讓 build 失敗，中低風險先記錄不擋 build（避免每個第三方套件的例行 CVE 都卡住開發節奏，僅在風險等級真的高時才強制處理） |

**建議**：資安基準文件已建立，執行面（CSP 規則是否真的夠嚴謹、2FA 是否確實落地）仍要等各服務實際開發、CI pipeline 真的跑起來後才能驗證，本文件的角色是提供規則，不是保證規則會被遵守——建議正式開發階段安排至少一次滲透測試或第三方資安稽核，而非只靠文件層級的規範。

## 9. Repo 拆分（決策 H）帶來的新缺口

[26-project-structure.md](26-project-structure.md) 決策 H 把電商平台從 1 個 monorepo 拆成 4 個 repo 後，除了 §3、§6 已更新的 CI/CD 缺口，還浮現以下先前不存在的問題：

| 項目 | 說明 |
|---|---|
| **本機多 repo 開發流程未定義** | 過去 monorepo 下 `docker compose up` 一次啟動全部即可本機開發；現在 `ecommerce-services`/`ecommerce-storefront`/`ecommerce-admin` 是三個獨立 repo，開發者若要同時改動「Catalog 新增欄位 + 後台顯示該欄位」這種橫跨兩個 repo 的功能，需要並排 clone 多個 repo 並手動處理彼此依賴（如後台想測試 Catalog 未發版的新端點，`api-client` 套件版本要怎麼指到「本機開發中」的版本而非已發布版本），目前沒有規範這個流程，容易讓每個開發者各自摸索出不同做法 |
| ~~共用函式庫的安全性修補傳播沒有例外機制~~ | **已解決**：[29-shared-service-conventions.md](29-shared-service-conventions.md) 新增 §4.1，資安修補獨立分級——7 個日曆天強制升級窗口 ＋ `[SECURITY]` Release Notes 標示 ＋ 人工追蹤清單，與一般版本更新的自由升級節奏區分開來 |
| **`api-client` 套件版本與後端服務版本的相容性矩陣更複雜** | §3 已列「各服務版本不同步的實際治理」缺口；決策 H 之後多一個維度——`api-client` 套件本身也獨立版本化，一份 `api-client@3.2.0` 對應的是「呼叫哪些服務的哪個版本」需要額外追蹤，不是單純服務對服務的相容性問題 |
| ~~私有套件/映像檔倉庫的建置與維運成本~~ | **已解決：定案 GitHub Packages**（NuGet/npm/容器映像檔三種格式統一用同一個服務），維運成本評估：免費額度對目前規模足夠、沿用既有 GitHub 組織權限模型，不需要額外自架/維運任何基礎設施，見 [26-project-structure.md](26-project-structure.md) §7 |

**建議**：安全性修補傳播機制、私有倉庫選型與成本評估已解決（見上）。僅剩 `api-client` 相容性矩陣（本機多 repo 開發流程的延伸問題，見 §3 同性質項目）留待正式建置團隊成形後排入 SOP 制定，不阻塞規格本身。

## 10. 微服務 API/資料模型缺漏（第七輪跨文件複查）

第七輪針對 11–25 全數服務文件與 02、09 逐一複查後，發現以下具體的 API 大綱/資料模型缺漏（不含各服務自己「待決議事項」已列的項目）：

| 服務/文件 | 缺漏 |
|---|---|
| ~~Vendor（[14-service-vendor.md](14-service-vendor.md)）~~ | **已解決**：新增 §4 API 大綱（VendorProfile、VendorStaff、StoreSettings 端點） |
| ~~Media（[19-service-media.md](19-service-media.md)）~~ | **已解決**：`MediaAsset` 新增 `VendorId` 欄位 |
| ~~CMS（[20-service-cms.md](20-service-cms.md)）~~ | **已解決**：§4 新增賣家草稿讀取端點 |
| ~~Shipping（[21-service-shipping.md](21-service-shipping.md)）~~ | **已解決**：§4 補齊運費區域/物流方式的 `GET`/`PUT`/`DELETE` |
| ~~Reviews（[24-service-reviews.md](24-service-reviews.md)）~~ | **已解決**：新增賣家回覆評價的資料欄位與端點 |
| ~~Payment（[18-service-payment.md](18-service-payment.md)）~~ | **已解決**：`Payment` 新增 `ProviderTransactionId` 欄位與 `(Provider, ProviderTransactionId)` 唯一索引 |
| ~~Catalog（[12-service-catalog.md](12-service-catalog.md)）~~ | **已解決**：§5 明訂讀取端點回傳 `DescriptionHtml`/`ShortDescriptionHtml`（已轉換安全 HTML），原始 Markdown 只在賣家編輯情境雙向傳遞 |
| ~~跨服務（12/13/16/17 等清單端點）~~ | **已解決**：[09-api-specification.md](09-api-specification.md) 新增 §3 統一分頁參數與回應 envelope |

**建議**：本節列出的 8 項已全數處理完畢。

## 11. 跨文件一致性與決策矛盾（第七輪跨文件複查）

| 項目 | 說明 |
|---|---|
| ~~Analytics「訂閱事件」與不引入訊息佇列的決策矛盾~~ | **已解決**：[22-service-analytics.md](22-service-analytics.md) §1 改為「定期輪詢/批次拉取」，移除「訂閱事件」措辭 |
| ~~Gateway 匿名端點限流缺口~~ | **已解決**：[25-service-gateway.md](25-service-gateway.md) 新增 §3.1 依 IP 位址限流登入/訪客結帳/訪客查單三類端點，資料模型新增 `AnonymousRateLimitRule` |
| ~~健康檢查端點未反映在服務文件 API 大綱~~ | **已解決**：[09-api-specification.md](09-api-specification.md) §2 新增統一聲明，各服務 API 大綱不需重複列出 |
| ~~內部端點認證註記不一致~~ | **已解決**：[29-shared-service-conventions.md](29-shared-service-conventions.md) §3 新增「各服務文件 API 大綱的讀法」統一約定 |
| ~~`SubOrder.CommissionAmount` 的計算來源未定義~~ | **已解決**：[17-service-order.md](17-service-order.md) §4、[06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §7 的 Saga 流程圖補上呼叫 Vendor Service 查詢 `CommissionRate` 的步驟，[14-service-vendor.md](14-service-vendor.md) §4 新增對應內部端點 |
| ~~貨到付款開關雙重管轄~~ | **已解決**：明訂 `StoreSettings.CodPaymentEnabled`（[14](14-service-vendor.md) §2）為唯一權威來源，[18-service-payment.md](18-service-payment.md) 的 `PaymentProviderSettings` 只管轄需要金鑰的金流閘道商，不重複設定 COD |
| ~~優惠券使用次數無併發保護~~ | **已解決**：[16-service-promotions.md](16-service-promotions.md) 新增 §4 併發保護機制，套用 [13-service-wms.md](13-service-wms.md) §4 同款原子條件更新模式 |
| ~~Order 待決議事項與 29 現況脫節~~ | **已解決**：[17-service-order.md](17-service-order.md) §6 已改寫並標記已解決 |

**建議**：本節 8 項已全數處理完畢。

## 12. 前後台需求規格落差（第七輪跨文件複查）

| 項目 | 說明 |
|---|---|
| ~~賣家後台缺少 Promotions/Shipping/Reviews 三個服務的管理介面~~ | **已解決**：[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1 補上三列，對應的後端 CRUD 端點也已分別補進 [16](16-service-promotions.md) §5、[21](21-service-shipping.md) §4、[24](24-service-reviews.md) §4 |
| ~~免登入下單「硬性需求」與 StoreSettings 可關閉開關矛盾~~ | **已解決**：移除 `StoreSettings.GuestCheckoutEnabled`（[08](08-vendor-admin-requirements.md) §2、[14](14-service-vendor.md) §2），[07](07-storefront-requirements.md) §1 補充說明此為平台層級硬性需求、不是賣家可自行關閉的營運選項 |
| ~~前台頁面清單漏列 Promotions/Reviews 服務對應~~ | **已解決**：[07-storefront-requirements.md](07-storefront-requirements.md) §3 補上兩處對應 |
| ~~「Admin」角色提及與本輪排除範圍不一致~~ | **已解決**：確認為文字誤植，[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §4.3 已移除 |
| ~~Suspended 階段漏了人工同步提醒~~ | **已解決**：[03-client-lifecycle.md](03-client-lifecycle.md) §5 補上與 §3/§4 一致的提醒 |
| ~~`SuxoShop.Shared.Translation` 套件規格空白~~ | **已解決**：[28-i18n.md](28-i18n.md) 新增 §3.1 定義套件內容（共用實體設定＋含 Fallback 邏輯的查詢介面） |
| ~~會員帳號刪除的關聯資料處理未定義~~ | **已解決**：[11-service-identity.md](11-service-identity.md) §5 定案採匿名化保留（`User.Id` 不變，`Email`/`PasswordHash` 清空），不做級聯刪除 |
| ~~Notification 重試策略缺乏具體參數~~ | **已解決**：[23-service-notification.md](23-service-notification.md) §4 補上指數退避＋最多 4 次重試的具體參數 |

> 複查同時核對了 08 §5.4 新增的 `WMS.Inventory.BackorderPolicy` 欄位是否已同步進 [13-service-wms.md](13-service-wms.md)——**已確認一致**（見 13 §2），非缺口，特此記錄避免日後重複查核。

**建議**：本節 8 項已全數處理完畢。

## 13. 文件互相引用但承諾未兌現（第八輪複查）

~~[01-architecture.md](01-architecture.md) §3 說明「客戶平台本身如何讀取這些設定，見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)」，但 06 §9 實際上只重申同一句高層原則，並未真的說明具體機制~~

**已解決**：01 §3、06 §9 已同步定案具體機制——功能開關以 `FEATUREFLAGS__<FlagName>` 環境變數存放在 `ecommerce-launch-<客戶代稱>` repo 的 `.env`（不放在隨原始碼 commit 的各服務 `appsettings.{Environment}.json`，避免牴觸「同一份映像檔靠環境變數部署到任何客戶」的既有原則），並釐清這是**合約層級**的主開關，與 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §2 賣家自己的 `StoreSettings` 是不同層級（主開關關閉時 `StoreSettings` 對應選項不生效）。

## 14. 本輪新增設計（會員登入、Saga 補償）帶來的連鎖影響（第九輪複查，2026-09-09）

新增消費者會員登入（[11-service-identity.md](11-service-identity.md) §5.1）與 Saga 補償失敗統一設計（[17-service-order.md](17-service-order.md) §4.1）後，回頭複查全部 15 個微服務規格與跨文件一致性，發現以下新缺口：

| 項目 | 說明 |
|---|---|
| ~~訪客升級為會員的確切機制未定義，與新增的信箱驗證流程存在安全疑慮~~ | **已解決**（使用者 2026-09-09 確認：要等信箱驗證通過）：[11-service-identity.md](11-service-identity.md) §5.1 新增「訪客升級為會員」設計——沿用同一個 `User.Id`（訂單本來就指向它，不需搬移資料），密碼暫存於新增的 `AccountActionToken.PendingPasswordHash`，驗證通過那一刻才寫入 `User.PasswordHash`、帳號才能登入，攻擊者拿不到受害者信箱就永遠無法完成這一步 |
| ~~§5.1 的登入/Refresh Token/密碼重設機制隱含適用 Seller/SellerStaff，但文字聚焦在「消費者會員」，易被誤讀為僅限 Buyer~~ | **已解決**：[11-service-identity.md](11-service-identity.md) §5.1 開頭新增「適用範圍澄清」段落 |
| ~~結帳 Saga 循序圖遺漏「Payment 建立失敗」分支~~ | **已解決**：兩份文件的循序圖皆已補上第四個 `alt` 分支，見 [17-service-order.md](17-service-order.md) §4 |
| ~~Gateway 路由範例與 Reviews Service 實際端點路徑不一致~~ | **已直接修正**：[25-service-gateway.md](25-service-gateway.md) §4.1 原寫「`/api/v1/orders/{id}/reviews`」與 [24-service-reviews.md](24-service-reviews.md) §4 實際端點 `POST /api/v1/orders/{subOrderId}/review` 對不上，純格式錯誤，非待決議，已訂正 |
| ~~Notification 的「Email/簡訊是否併入本服務」待決議項急迫性提升~~ | **已解決**：定案 Email 併入本服務、簡訊現階段不做，解除 Identity 驗證信的既有卡點，見 [23-service-notification.md](23-service-notification.md) §7 |

**建議**：本節 5 項已全數處理完畢。
