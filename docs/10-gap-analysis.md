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

> 本文件分析 [00-overview.md](00-overview.md)–[30-open-decisions-register.md](30-open-decisions-register.md) 目前規格的缺口，供下一輪規劃排優先序。

## 1. 技術面

| 項目 | 說明 |
|---|---|
| ~~Correlation ID 貫穿追蹤~~ | **已解決**：Gateway 產生/沿用 `X-Correlation-Id`，往下游強制傳遞，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §1.1 |
| Catalog 與 WMS 的一致性 | 把庫存權責從 Catalog 分離到 WMS 後，商品詳情頁需要同時打兩個服務（Catalog 拿資訊、WMS 拿庫存）；WMS 短暫不可用時，商品頁該顯示「查詢中」還是「暫時隱藏庫存」，尚未定義降級行為（見 [13-service-wms.md](13-service-wms.md) 待決議） |
| `PlatformSupportStaff` 的跨服務授權模式 | [08](08-vendor-admin-requirements.md) §4 只在 WMS、Order 各舉了一個唯讀診斷端點，這個角色實際上需要跨所有服務的一致授權策略（如統一的 Policy-based Authorization），目前是逐服務各自加端點，容易遺漏或不一致 |
| ~~內部 API 的服務間認證機制~~ | **已解決**：網路隔離（`/internal/v1/...` 只綁定 Docker 內部網路）+ 服務身分 JWT 縱深防禦，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §3 |
| 多倉支援 | WMS Service 目前設計未區分「爸芭樂是否有多個實體倉庫」，若未來有多倉需求（如不同產地各自出貨），批次/庫存模型需要再加 `WarehouseId` 維度（見 [13-service-wms.md](13-service-wms.md)） |
| 效期商品的自動下架/促銷 | WMS 記錄了批次有效期，但「快到期商品要不要自動下架、自動加入促銷」是業務邏輯缺口，目前只有資料，沒有對應流程 |
| 整合測試 | 目前只有領域層單元測試，微服務拆分後**更需要**跨服務整合測試（尤其 Saga 補償路徑），本輪未涵蓋 |
| ~~跨服務共通慣例未定義~~ | **已解決**：健康檢查端點、結構化 log 格式、Correlation ID 傳遞規則，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §1 |
| **`docker-compose.yml` 骨架尚未撰寫** | [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 定調了單一 VPS + Docker Compose + 三種 DB 連線模式，但連最基本的 15 服務 compose 檔案骨架都還沒有，[05-scope-and-open-items.md](05-scope-and-open-items.md) 已將完整開發環境文件列為待辦，此為其中最基礎的一步 |
| **單一 VPS 的備份/災難復原策略空白（新）** | 決策採單一 VPS 部署（[06](06-ecommerce-platform-architecture.md) §6.1），意味著這台主機是單點故障——不管 DB 走 Docker 內建/外部/內部哪種模式，若走 Docker 內建 DB，這台主機的資料庫備份、還原演練完全沒有規劃，比多機部署的既有白牌模式風險更集中 |
| **網域與 SSL 憑證管理未提及（新）** | 客戶自訂網域是白牌系統的常態需求，但本輪規格完全沒有提到憑證簽發/續簽（如 Let's Encrypt 自動化）由誰負責、怎麼跟 Gateway 的單一入口角色搭配 |
| ~~CORS 政策未定義~~ | **已解決**：每個客戶部署明確列出允許來源網域，禁止萬用字元，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 |

## 2. 流程/商業面

| 項目 | 說明 |
|---|---|
| StoreSettings 的歸屬服務未定案 | [14-service-vendor.md](14-service-vendor.md) 與 [20-service-cms.md](20-service-cms.md) 都提到 `StoreSettings` 歸屬未定，本輪未做最終決定，會影響哪個服務要開發這組 API |
| ShyeCMS 零連接後，拾夜科技如何得知客戶異常 | 決策 C/D 確認不連接、不取資料後，拾夜科技完全依賴客戶主動回報問題（工單/電話）才會知道系統異常——這是刻意的取捨（見 [01-architecture.md](01-architecture.md) §4），但代表**沒有任何主動監控告警的管道**，回應時間可能因此更難達成，值得提醒業務/客服團隊 |
| `PlatformSupportStaff` 存取透明度 | [08](08-vendor-admin-requirements.md) §6 已列為待決議：客戶端目前不會即時知道拾夜科技人員登入查看了什麼，只能事後翻 AuditLog |
| 訪客結帳的防詐風險 | [07](07-storefront-requirements.md) §5 已提出，生鮮商品退貨/報廢成本高，惡意下單（尤其大量小額測試）風險比一般電商更需要留意 |

## 3. 維運面

| 項目 | 說明 |
|---|---|
| 各服務版本不同步的實際治理 | [09](09-api-specification.md) §1 已將「各服務版本不同步」訂為常態而非問題，但沒有配套的「版本相容性矩陣」文件，維運人員難以一眼看出「目前這個客戶的 Order v2 是否能跟 WMS v1 相容」 |
| 單一客戶部署的服務數量（15 個）資源評估 | 沿用 [06](06-ecommerce-platform-architecture.md) §6.3 既有待決議，粗估規格未經實測校正 |
| Deprecation Window（3 個月）的實際執行機制 | [09](09-api-specification.md) §1 訂了政策，但沒有工具/流程確保「舊版本到期後真的會被下線」，容易變成口頭政策 |
| **CI/CD 策略空白（新）** | 15 個服務各自獨立部署，但完全沒有規劃是「統一一條 pipeline 建置全部」還是「服務各自獨立 pipeline、可獨立升級」——這直接影響 [09](09-api-specification.md) 允許的「各服務版本不同步」在實務上如何落地 |

## 4. 建議下一步（依風險排序）

1. **補寫 ShyeCMS 前端需求規格**——目前完全空白，是唯一一個「連怎麼動工都無法回答」的缺口，優先度最高（見 §7）。
2. **撰寫 `docker-compose.yml` 骨架**——共通慣例、Correlation ID、服務間認證、Markdown 管線、資安基準都已定案（[29-shared-service-conventions.md](29-shared-service-conventions.md)），剩下缺的是把它們落實成實際可跑的 compose 檔案。
3. **決定 Catalog/WMS 呼叫失敗時的前台降級行為**——直接影響買家體驗，且是拆分 WMS 直接產生的新風險。
4. **定案 `StoreSettings` 歸屬服務**——影響後續 API 開發分工，屬於小決策但會卡住實作排程。
5. **單一 VPS 的備份/災難復原策略**——決策集中風險到一台主機，上線前必須有備份與還原演練規劃，不是可以無限期擱置的項目。
6. **共用函式庫的資安修補傳播機制**（見 §9）——決策 H 把共用邏輯改成版本化套件、各服務自行決定升級時機，這對一般版本更新是對的，但對資安修補是新風險，需要另訂「資安修補限期全部升級」的例外規則，否則會削弱 §8 剛建立的資安基準的實際保護力。
7. **熱銷排行/付款分布的圖表函式庫選型**——Lightweight Charts 明確排除這兩種圖表類型，目前無替代方案，賣家後台這兩個既有功能實際上卡住無法動工。
8. 效期商品自動化、多倉支援、防詐機制、網域/憑證管理、CI/CD、ShyeCMS 認證機制、2FA、本機多 repo 開發流程——屬於功能性增強或維運細節，可排入下一輪迭代，非規格階段必須解決。

> 「補齊其餘服務的 API 大綱」已於後續一輪完成（見 [11-service-identity.md](11-service-identity.md)–[25-service-gateway.md](25-service-gateway.md)），故不再列於本節。

## 5. 設計/前端面（新，因應動畫決策浮現）

| 項目 | 說明 |
|---|---|
| 設計系統 Token 完全缺失 | [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5.1 已決定動畫效果主要用 CSS，但色彩、字級、間距、動畫時長/緩動曲線（`--duration-fast`、`--ease-standard` 等）目前完全沒有正式的設計系統文件定義，前台/後台開發時容易各自硬編碼數值，導致視覺不一致 |
| 無障礙的動畫降級規則未成文 | §5.1 提到需搭配 `prefers-reduced-motion`，但目前只是原則性一句話，沒有具體規則（如降級後動畫時長歸零還是簡化成瞬時切換） |
| 元件庫/UI 框架尚未選型 | 前台/後台目前只確定 Next.js（SSG）與 Vite SPA 兩種前端專案形態，但用什麼元件庫（如 Tailwind CSS、Ant Design、MUI 或全自訂）尚未決定，會直接影響 CSS 動畫怎麼組織（Utility Class vs CSS Module vs CSS-in-JS） |

**建議**：這三項屬於同一個缺口（設計系統文件從缺）的不同面向，建議合併規劃成一份新文件（如 `31-design-system.md`，`26`–`30` 已分別被 [26](26-project-structure.md)、[27](27-pwa-and-accessibility.md)、[28](28-i18n.md)、[29](29-shared-service-conventions.md)、[30](30-open-decisions-register.md) 使用），而不是逐項零星補丁，比照本文件集其餘服務規格的拆分精神。

## 6. ShyeCMS 專案面（因應「確認專案清單」浮現）

確認 ShyeCMS 技術棧（[00-overview.md](00-overview.md) 決策 F）並整理出完整專案結構（[26-project-structure.md](26-project-structure.md)）後，發現一個先前被完全忽略的缺口：

| 項目 | 說明 |
|---|---|
| **ShyeCMS 前端頁面/操作流程規格完全空白（最重要）** | 電商平台有 [07](07-storefront-requirements.md)（前台需求）、[08](08-vendor-admin-requirements.md)（後台需求）對應兩個前端；但 ShyeCMS 確認要做成獨立 React SPA 後，**沒有任何文件描述 `shyecms-admin` 實際有哪些頁面、拾夜科技員工的操作流程長什麼樣子**——[02-data-model.md](02-data-model.md)、[03-client-lifecycle.md](03-client-lifecycle.md) 只講資料模型與流程步驟，缺少 UI/UX 層級的規格 |
| ShyeCMS 的認證方式未指定 | 電商平台各服務用 JWT（[09-api-specification.md](09-api-specification.md)），但 ShyeCMS 身為獨立系統，`StaffUser` 登入要用什麼機制（JWT？Session？是否需要 2FA，畢竟能操作所有客戶的合約資料）完全沒有規格 |
| **六個 repo 各自的 CI/CD 與跨 repo 版本協調都未定案（範圍擴大）** | 沿用 §3 已列的 CI/CD 缺口，[26-project-structure.md](26-project-structure.md) 決策 H 把電商平台從 1 個 repo 拆成 4 個（services/storefront/admin/deploy）後，缺口從「2 條 pipeline」變成「6 條 pipeline + 1 套跨 repo 版本協調流程」：某個 repo 發新版後，`ecommerce-deploy` 何時、由誰更新映像檔標籤，目前只有問題本身被寫下來（[26](26-project-structure.md) §7），還沒有答案 |
~~Monorepo 建置工具未選型~~ | **已解決（前提改變）**：電商平台不再是單一 monorepo——前台、後台、15 個服務已拆成三個獨立 repo（[26-project-structure.md](26-project-structure.md) 決策 H），`ecommerce-services` 內部也**不設共用 `.sln`**，改為各服務獨立建置，不需要 Nx/Turborepo 等級的跨語言建置編排工具 |

**建議**：ShyeCMS 前端規格的優先度應提升——目前 ShyeCMS 的角色定義、資料模型都完備，但完全沒有人能依現有文件動工做出 `shyecms-admin` 這個介面，這是規格完整度上最大的落差，建議下一輪比照 [07](07-storefront-requirements.md)/[08](08-vendor-admin-requirements.md) 的規格深度補一份 ShyeCMS 前端需求文件。

## 7. 多語系/圖表/Markdown 決策的連鎖影響

這三項決策彼此獨立，但都對既有規格產生了連鎖影響。多數已在本輪解決，逐一列出現況：

| 項目 | 現況 |
|---|---|
| ~~各服務的 Translation 表尚未落實到個別文件~~ | **已解決**：Catalog（[12](12-service-catalog.md)）、CMS（[20](20-service-cms.md)）、Promotions（[16](16-service-promotions.md)）、Shipping（[21](21-service-shipping.md)）都已補上 `Translation` 表 |
| ~~Markdown 渲染/清理邏輯是否共用~~ | **已解決**：統一為共用函式庫 `RenderMarkdownToSafeHtml`，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §2 |
| **圖表方案只解決了一部分（仍未解決）** | [22-service-analytics.md](22-service-analytics.md) §4 明確排除 Lightweight Charts 不適用長條圖/圓餅圖，代表「熱銷商品排行」「付款方式分布」兩個既有報表項目目前**沒有圖表函式庫可用**，賣家後台這兩個既有功能實際上還無法動工 |
| ~~i18n × SSG 的建置成本未重新估算~~ | **已解決**：執行期主機規格不受語言數量影響，Next.js 建置步驟改移到 CI/CD 執行，見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.3 |

**建議**：唯一還沒解決的是熱銷排行/付款分布的圖表選型，其餘三項已在本輪處理完畢。

## 8. 資安面（因應「資訊安全性非常重要」正式收斂）

本輪新增 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 作為所有服務的資安基準，取代原本散落在各文件裡的零星提及。現況：

| 已定案 | 仍待決議 |
|---|---|
| HTTPS/HSTS、密碼雜湊、SQL Injection 防護、XSS 防護（含 Markdown 管線）、CSRF、CORS 政策、敏感憑證加密、Data Protection 金鑰持久化、API 速率限制範圍、PCI DSS（金流不落地）、服務間認證（網路隔離 + JWT） | 高權限帳號 2FA（見 §6）、CSP 詳細規則、log 集中收集方案選型、依賴套件掃描是否納入 CI（CI/CD 本身仍未定案，見 §3） |

**建議**：資安基準文件已建立，但**執行面**（CI 是否真的擋得住有漏洞的依賴套件、CSP 規則是否真的夠嚴謹）要等 CI/CD 策略（§3）與各服務實際開發時才能驗證，本文件的角色是提供規則，不是保證規則會被遵守——建議正式開發階段安排至少一次滲透測試或第三方資安稽核，而非只靠文件層級的規範。

## 9. Repo 拆分（決策 H）帶來的新缺口

[26-project-structure.md](26-project-structure.md) 決策 H 把電商平台從 1 個 monorepo 拆成 4 個 repo 後，除了 §3、§6 已更新的 CI/CD 缺口，還浮現以下先前不存在的問題：

| 項目 | 說明 |
|---|---|
| **本機多 repo 開發流程未定義** | 過去 monorepo 下 `docker compose up` 一次啟動全部即可本機開發；現在 `ecommerce-services`/`ecommerce-storefront`/`ecommerce-admin` 是三個獨立 repo，開發者若要同時改動「Catalog 新增欄位 + 後台顯示該欄位」這種橫跨兩個 repo 的功能，需要並排 clone 多個 repo 並手動處理彼此依賴（如後台想測試 Catalog 未發版的新端點，`api-client` 套件版本要怎麼指到「本機開發中」的版本而非已發布版本），目前沒有規範這個流程，容易讓每個開發者各自摸索出不同做法 |
| **共用函式庫的安全性修補傳播沒有例外機制** | [29-shared-service-conventions.md](29-shared-service-conventions.md) 定調共用邏輯改版本化套件、各服務自行決定升級時機（§4.2 的理由是保留獨立升級節奏）。但如果 `SuxoShop.Shared.Markdown` 修的是一個嚴重 XSS 漏洞，「各服務自行決定何時升級」就不該適用——目前沒有區分「一般版本更新（可以慢慢升）」與「資安修補（必須限期全部升級）」，這是決策 H 為了解決 §5 的耦合問題而**新產生**的風險，不是原本就有的 |
| **`api-client` 套件版本與後端服務版本的相容性矩陣更複雜** | §3 已列「各服務版本不同步的實際治理」缺口；決策 H 之後多一個維度——`api-client` 套件本身也獨立版本化，一份 `api-client@3.2.0` 對應的是「呼叫哪些服務的哪個版本」需要額外追蹤，不是單純服務對服務的相容性問題 |
| **私有套件/映像檔倉庫的建置與維運成本** | 決策 H 需要私有 NuGet feed、私有 npm registry、容器映像檔倉庫三種基礎設施才能運作，這是 monorepo 時代不需要的額外維運項目，目前只在 [26](26-project-structure.md) §7 列為選型待決議，其建置與維運成本（含金錢與人力）未被評估過 |

**建議**：安全性修補傳播機制（第二項）建議優先處理——這直接關係到 [29-shared-service-conventions.md](29-shared-service-conventions.md) 好不容易建立的資安基準（§8）能不能真正落實，如果共用函式庫修了漏洞但服務們各自「之後再升級」，資安基準文件本身的保護力會大打折扣。其餘三項屬於開發流程/維運成本問題，可在正式建置團隊成形後排入 SOP 制定，不阻塞規格本身。
