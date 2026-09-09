# 00 - 專案總覽 (Overview)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「拾夜科技需要一套管理客群的系統（ShyeCMS）」需求，經可行性分析與 4 項架構決策後建立文件集 |
| v0.2 | 2026-09-08 | ordinarycas | 使用者推翻 v0.1 決策 C、D：確認 **ShyeCMS 不與任何客戶平台連接**、**不取得客戶的商品/售價/會員資料**，ShyeCMS 純粹是拾夜科技內部的商業/合約管理系統；新增決策 E，將「爸芭樂」電商平台本身規劃為獨立的微服務產品（C# .NET 10 + React SSG + PostgreSQL），見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) |
| v0.3 | 2026-09-08 | ordinarycas | 新增 [07](07-storefront-requirements.md)–[10](10-gap-analysis.md)：前台需求（免登入下單）、後台需求（含拾夜科技 `PlatformSupportStaff` 支援權限）、API 版本控管規範、缺口分析 |
| v0.4 | 2026-09-08 | ordinarycas | 新增後台匯出 WooCommerce 商品 CSV 功能（[08](08-vendor-admin-requirements.md) §6、[09](09-api-specification.md) §8），回應「方便廠商移轉 WooCommerce」需求 |
| v0.5 | 2026-09-08 | ordinarycas | 使用者刪除本文件集的前身規格（原 `docs/00`–`16`，涵蓋單一客戶部署的完整電商平台規格），並將本文件集由 `docsv2/` 更名為 `docs/`，成為唯一的規格來源。整份文件移除對已刪除文件的引用，改為自我完整敘述；新增第 8 節記錄前身規格涵蓋但本文件集未涵蓋的範圍 |
| v0.6 | 2026-09-08 | ordinarycas | 新增決策 F：確認 ShyeCMS 自己的技術棧（C# .NET 10 單體後端 + React SPA 前端，獨立 repo），回應「確認專案清單」需求；新增 [26-project-structure.md](26-project-structure.md) 記錄完整的 repo/專案結構 |
| v0.7 | 2026-09-08 | ordinarycas | 新增 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md)：前台+後台皆支援 RWD/PWA（可安裝），前台另需符合 WCAG 2.1 AA（台灣網站無障礙規範 110.07 版基準） |
| v0.8 | 2026-09-08 | ordinarycas | 新增決策 G：多語系支援（繁中/英/日，繁中為主），見 [28-i18n.md](28-i18n.md)；圖表函式庫確定用 TradingView Lightweight Charts（[22-service-analytics.md](22-service-analytics.md)）；後台內容編輯改用 Markdown（[12](12-service-catalog.md)、[20](20-service-cms.md)） |
| v0.9 | 2026-09-08 | ordinarycas | 解決 [10-gap-analysis.md](10-gap-analysis.md) §7 列出的連鎖缺口：新增 [29-shared-service-conventions.md](29-shared-service-conventions.md)（跨服務共通慣例＋資安基準，含服務間認證決策）；Promotions/Shipping 補上 Translation 表；[06](06-ecommerce-platform-architecture.md) §6.3 重新評估多語系對主機規格的影響 |
| v0.10 | 2026-09-08 | ordinarycas | 回應「預期資料夾目錄結構」需求：[26-project-structure.md](26-project-structure.md) 補上兩個 repo 的實際樹狀圖、服務內部分層慣例、共用函式庫落地位置，並解決 `.sln`/`packages/api-client` 兩項既有待決議 |
| v0.11 | 2026-09-08 | ordinarycas | 新增決策 H：React 前端獨立成自己的 repo、前台後台也彼此分開，共用邏輯改為版本化套件而非專案參照——推翻 v0.10 的部分結構決定，repo 數量由 2 個變成 6 個；同時採納使用者對「共用 `.sln`」的質疑，取消單一 `.sln` 設計 |
| v0.12 | 2026-09-08 | ordinarycas | 回應「待決議事項」需求：新增 [30-open-decisions-register.md](30-open-decisions-register.md) 彙整全部 29 份文件的 77 項待決議並排出優先處理前 10 名；修正 [16-service-promotions.md](16-service-promotions.md) 缺少待決議章節的疏漏；[28-i18n.md](28-i18n.md) 新增幣別/金流在地化的明確排除說明；釐清 [05-scope-and-open-items.md](05-scope-and-open-items.md) 的彙整範圍僅限 ShyeCMS |
| v0.13 | 2026-09-09 | ordinarycas | repo 更名 `ecommerce-deploy`→`ecommerce-launch`，呼應實際 checkout 的資料夾命名（見 [26-project-structure.md](26-project-structure.md)）；新增 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)（`shyecms-admin` 頁面/操作流程規格）並收錄進 §6 文件索引，解決 [10-gap-analysis.md](10-gap-analysis.md) §6 已列的最高優先缺口 |

## 1. 本文件集的緣起

拾夜科技有限公司（ShyeTech）是白牌電商平台系統商，開發後分別部署給多個客戶（如「爸芭樂」），每個客戶各自獨立一套環境與資料庫。本文件集回應兩個需求：

1. **拾夜科技需要一套系統（ShyeCMS）管理所有客戶**——客戶檔案、訂閱方案、合約內容、功能授權紀錄，見第 2–5 節、[01](01-architecture.md)–[05](05-scope-and-open-items.md)。
2. **具體設計賣給客戶的電商平台本身**——以「爸芭樂」（線上賣芭樂）為案例，直接採微服務架構，技術棧為 C# .NET 10 + React（可生成靜態頁面）+ PostgreSQL，見 [06](06-ecommerce-platform-architecture.md)–[10](10-gap-analysis.md)。

這兩塊是**兩個獨立、沒有技術連接的系統**：ShyeCMS 是拾夜科技內部的商業管理工具；電商平台是賣給客戶、在客戶自己環境獨立運作的產品。

## 2. 原始需求與可行性分析結論（含 v0.2 修正）

使用者提出 7 項需求，第一輪分析後做出 4 項決策；使用者在第二輪明確**推翻其中 2 項**並提出新指示，最終結論如下：

| # | 原始需求 | 最終結論 |
|---|---|---|
| 1 | 拾夜科技目標客群含形象網站/部落格/電商 | 本輪範圍排除，見 [05-scope-and-open-items.md](05-scope-and-open-items.md)（決策 A，未變） |
| 2 | 需要 ShyeCMS 管理客群網站/主機/收費/訂閱/功能權限 | 可行；但「功能權限」是**商業合約紀錄**，不是技術強制（v0.2 修正，見決策 C） |
| 3 | 客戶（如爸芭樂）資料需先登入 ShyeCMS | 確認為拾夜科技員工代為建檔，客戶無帳號（決策 B，未變） |
| 4 | 客戶微服務依 ShyeCMS 開關功能確認狀態 | **v0.2 推翻**：ShyeCMS 不跟客戶平台連接，不存在任何即時查詢（見決策 C、[01-architecture.md](01-architecture.md)） |
| 5 | 客戶拿到的頁面/後台功能清單 | 具體化為 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 的服務清單與 §8 頁面清單 |
| 6 | 拾夜科技如何取得客戶商品/售價/會員資料 | **v0.2 推翻**：不取得，ShyeCMS 是獨立賣給廠商的系統，與客戶資料無關（決策 D） |
| 7 | 先分析可行性 | 第一輪分析見本文件 v0.1 版本歷史；第二輪指示已直接採納為決策 |

## 3. 架構決策（v0.2 現況，取代 v0.1 決策 C、D）

- **決策 A（範圍，未變）**：本輪只規劃電商平台（沿用「爸芭樂」案例）。形象網站/部落格產品線列為非本輪範圍，見 [05](05-scope-and-open-items.md)。
- **決策 B（ShyeCMS 定位，未變）**：ShyeCMS 是純拾夜科技內部工具，客戶沒有帳號，一切建檔/操作由拾夜科技員工代為執行。
- **決策 C（ShyeCMS 與客戶平台的關係，v0.2 取代原決策 C）**：**ShyeCMS 不與任何客戶平台連接**——沒有 API 呼叫、沒有功能開關查詢、沒有任何執行期依賴。「客戶方案含哪些功能」在 ShyeCMS 裡只是一筆**商業/合約紀錄**，實際客戶環境要開通哪些功能，由維運人員在部署當下依合約**手動設定**該環境的設定檔，兩邊不存在技術驗證。詳見 [01-architecture.md](01-architecture.md)。
- **決策 D（資料取得方式，v0.2 取代原決策 D）**：拾夜科技**不取得**客戶的商品、售價、會員等任何經營資料。ShyeCMS 就是一套獨立賣給廠商的系統（如同賣一套軟體授權），不因為賣了這套系統就順帶取得客戶的營運資料。GMV 計費等若未來真的需要用量數據，須由客戶自行申報或另立雙方同意的機制，不在 ShyeCMS 的預設設計內。
- **決策 E（電商平台技術棧）**：以「爸芭樂」案例設計的電商平台，直接以**微服務**架構規劃（不從單體架構演進，因為這是全新產品線的具體設計），技術棧固定為：後端 **C# / ASP.NET Core (.NET 10)**、前台 **React（須可生成靜態頁面，即 Next.js SSG/ISR）**、資料庫 **PostgreSQL**，部署拓樸為單一虛擬主機 + Docker Compose。詳見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)。
- **決策 F（ShyeCMS 技術棧，新增）**：ShyeCMS 後端為 **C# / ASP.NET Core (.NET 10) 單體**（**不採微服務**——與電商平台的差異是刻意的：ShyeCMS 範疇是客戶/合約/訂閱等 CRUD 為主的內部管理，沒有跨服務交易的 Saga 需求，微服務化只會增加不必要的維運複雜度），前端為**獨立的 React SPA（Vite）**，與電商平台的賣家後台同技術棧；資料庫為獨立的 **PostgreSQL** 執行個體（與任何客戶的電商平台資料庫完全分開，呼應決策 C 的零連接原則）。詳見 [26-project-structure.md](26-project-structure.md)。
- **決策 G（多語系範圍，新增）**：電商平台前台+後台須支援**繁體中文（預設）、英文、日文**，ShyeCMS 不在範圍內（維持繁中單一語系）。詳見 [28-i18n.md](28-i18n.md)。
- **決策 H（Repo 拆分，取代決策 F 原本「2 個 repo」的說法）**：React 前端一律獨立成自己的 repo，不與後端放在一起；電商平台的前台（買家）與後台（賣家）也彼此獨立成兩個 repo，避免其中一邊的原始碼/建置產物意外牽涉到另一邊。實際變成 **6 個 repo**：`shyecms-api`、`shyecms-admin`、`ecommerce-services`（15 微服務）、`ecommerce-storefront`、`ecommerce-admin`、`ecommerce-launch`（新增，統整前三者建置出的映像檔，是唯一實際 clone 到客戶 VPS 上的 repo）。詳見 [26-project-structure.md](26-project-structure.md)。

## 4. 名詞定義

| 名詞 | 說明 |
|---|---|
| ShyeCMS | 拾夜科技內部系統，記錄所有客戶的合約、訂閱、收費與功能授權**紀錄**；本身不對客戶開放登入，也**不與客戶平台有任何技術連接** |
| Client（客戶） | ShyeCMS 裡的一筆客戶檔案，如「爸芭樂」；純粹是商業紀錄，與客戶實際部署的軟體之間沒有技術關聯 |
| Client Deployment | 某客戶實際跑起來的一套環境的**盤點紀錄**（網域、版本號），供拾夜科技內部掌握現況用，非連線端點 |
| Feature Entitlement | 某客戶的合約內含哪些功能——**商業紀錄**，非技術強制；實際開通由部署當下手動設定 |
| 電商平台（爸芭樂案例） | 拾夜科技賣給客戶的獨立微服務產品本身，見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)，與 ShyeCMS 完全解耦 |

## 5. 角色定義

| 角色 | 說明 |
|---|---|
| ShyeCMS 操作人員（拾夜科技員工） | 拾夜科技跨所有客戶站台的維運/業務人員，細分見 [02-data-model.md](02-data-model.md) StaffUser |
| 客戶（Client） | **不是** ShyeCMS 的使用者角色——客戶方（如爸芭樂）只使用自己電商平台裡的賣家角色，對 ShyeCMS 沒有存取權 |
| 買家 / 賣家 / 拾夜科技支援人員（PlatformSupportStaff） | 電商平台本身的角色，見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) |

## 6. 文件索引

| 文件 | 內容 |
|---|---|
| [00-overview.md](00-overview.md) | 本文件：緣起、可行性分析結論、架構決策、名詞與角色定義 |
| [01-architecture.md](01-architecture.md) | ShyeCMS 的定位邊界：與客戶平台**零技術連接**、功能授權如何以人工方式落地 |
| [02-data-model.md](02-data-model.md) | ShyeCMS 核心實體：Client、Deployment（盤點用）、SubscriptionPlan、FeatureEntitlement（商業紀錄） |
| [03-client-lifecycle.md](03-client-lifecycle.md) | 客戶從建檔、開通、營運到終止的生命週期流程（純商業/人工作業） |
| [04-feature-entitlement-and-metering.md](04-feature-entitlement-and-metering.md) | **已停用設計**：v0.1 曾規劃的即時查詢/用量拉取機制，因 v0.2 決策 C、D 而廢止，保留於此供追溯 |
| [05-scope-and-open-items.md](05-scope-and-open-items.md) | 本輪明確排除項目與待決議清單 |
| [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md) | `shyecms-admin` 頁面清單、操作流程、角色權限矩陣（內容屬 ShyeCMS，編號延續在 30 之後，見該文件開頭說明） |
| [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) | 「爸芭樂」案例：獨立微服務電商平台架構，C# .NET 10 + React SSG + PostgreSQL |
| [07-storefront-requirements.md](07-storefront-requirements.md) | 前台需求：免登入下單、LINE/Google 登入（保留） |
| [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) | 賣家後台需求：上架/數據/版型/自家功能開關、拾夜科技 `PlatformSupportStaff` 支援權限 |
| [09-api-specification.md](09-api-specification.md) | API 版本控管策略與文件格式規範（跨服務通用） |
| [10-gap-analysis.md](10-gap-analysis.md) | 缺口分析與建議調整項目 |
| [11-service-identity.md](11-service-identity.md) | Identity Service：會員/賣家帳號、認證、`PlatformSupportStaff` 角色 |
| [12-service-catalog.md](12-service-catalog.md) | Catalog Service：商品主檔、分類、變體、WooCommerce 匯出 API |
| [13-service-wms.md](13-service-wms.md) | WMS Service：庫存、批次/效期、原子扣減 |
| [14-service-vendor.md](14-service-vendor.md) | Vendor Service：商店資料、子帳號、StoreSettings |
| [15-service-cart.md](15-service-cart.md) | Cart Service：購物車 |
| [16-service-promotions.md](16-service-promotions.md) | Promotions Service：優惠券 |
| [17-service-order.md](17-service-order.md) | Order Service：訂單狀態機、結帳 Saga 協調者 |
| [18-service-payment.md](18-service-payment.md) | Payment Service：金流串接、回調三道防線 |
| [19-service-media.md](19-service-media.md) | Media Service：檔案儲存後端、縮圖、配額 |
| [20-service-cms.md](20-service-cms.md) | CMS Service：首頁/形象頁版型 |
| [21-service-shipping.md](21-service-shipping.md) | Shipping Service：物流方式與運費試算 |
| [22-service-analytics.md](22-service-analytics.md) | Analytics Service：報表/數據分析（唯讀） |
| [23-service-notification.md](23-service-notification.md) | Notification Service：LINE 官方帳號整合 |
| [24-service-reviews.md](24-service-reviews.md) | Reviews Service：商品評價 |
| [25-service-gateway.md](25-service-gateway.md) | Open API Gateway：對外入口、金鑰驗證、速率限制 |
| [26-project-structure.md](26-project-structure.md) | 完整 repo/專案結構：共 6 個 repo（含 `ecommerce-launch` 部署設定），各自的資料夾樹狀圖與部署單位 |
| [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md) | RWD/PWA（前台+後台皆可安裝）與前台無障礙規範（WCAG 2.1 AA） |
| [28-i18n.md](28-i18n.md) | 多語系支援：繁中/英/日，URL 路由策略，內容翻譯資料模型 |
| [29-shared-service-conventions.md](29-shared-service-conventions.md) | 跨服務共通慣例與資安基準：Correlation ID、健康檢查、Markdown 管線、服務間認證、資安規則 |
| [30-open-decisions-register.md](30-open-decisions-register.md) | 全部 29 份文件的待決議事項總表（77 項），含優先處理前 10 名 |

## 7. ShyeCMS 對系統商既有維運缺口的回應

- ShyeCMS 是「系統商內部維運後台（Ops Console）」缺口的**部分**回應（僅涵蓋客戶/合約管理，不含健康監控儀表板）；「License / 授權驗證機制」缺口因決策 C（不連接）**仍未解決**，本輪明確排除用技術手段強制授權。
- 客戶部署登記，正式化為 [02-data-model.md](02-data-model.md) 的 `ClientDeployment`——僅作內部盤點紀錄，不是連線端點。
- 訂閱分級＋混合制框架，正式化為 `SubscriptionPlan`；GMV 超額抽成因決策 D（不取得客戶資料）**仍是完全空白的缺口**，未被本輪解決。

## 8. 前身規格涵蓋、本文件集未涵蓋的範圍

本文件集的前身是一份更完整的規格（原編號 `00`–`16`，涵蓋單一客戶部署電商平台的完整需求），使用者已將其刪除，本文件集（原 `docsv2`）更名為現在的 `docs/`，成為唯一規格來源。**以下內容曾經存在於前身規格，現已不存在，需要時須重新撰寫**，列在此處避免日後誤以為「本來就沒規劃過」：

| 曾涵蓋的內容 | 說明 |
|---|---|
| 完整資料模型（欄位級） | 商品、訂單、優惠券、物流、評價等實體的完整欄位定義；本文件集的 [06](06-ecommerce-platform-architecture.md) 只有服務邊界層級的說明，沒有到欄位級 |
| 平台管理員需求 | 賣家審核、全站金流物流設定、客訴仲裁——[08](08-vendor-admin-requirements.md) 只涵蓋賣家與拾夜科技支援角色，見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2 的排除說明（暫定爸芭樂為單一賣家自營） |
| UI 設計規範與色彩系統 | 前台/後台的設計 Token（色彩/字級/間距/元件規範） |
| 非功能需求 | 效能/安全/法規遵循/監控/測試策略的具體目標值 |
| 維護/SLA/版本派送流程 | 客戶部署登記表、分批推送、Migration 執行、Rollback 計畫、SLA 分級表 |
| 開發環境建置指令 | Docker Compose 啟動指令、EF Core Migration 指令、示範帳號——見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2，本文件集尚未補上對應 15 服務新架構的版本 |
| 金流/檔案儲存實作細節 | 各金流廠商驗章方式、回調安全機制、檔案儲存三種後端的安全處理規則——[06](06-ecommerce-platform-architecture.md) §12 只留概念性引用，細節已隨前身規格一併刪除 |
| 資料保存政策 | 訪客個資去識別化/刪除期限的具體天數與判斷邏輯 |
| 需求異動歷史 | 前身規格從專案啟動到本文件集誕生前的完整決策歷程（原編號 R-001 至 R-013） |

這些屬於**已知且刻意接受的缺口**（使用者在刪除前已確認並接受內容永久遺失），不是本文件集的疏漏。若之後任一項目需要用到，需重新撰寫，不能假設內容還存在於某處。
