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

1. ~~補寫 ShyeCMS 前端需求規格~~——**已解決**：新增 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)，見 §6。
2. **撰寫 `docker-compose.yml` 骨架**——共通慣例、Correlation ID、服務間認證、Markdown 管線、資安基準都已定案（[29-shared-service-conventions.md](29-shared-service-conventions.md)），剩下缺的是把它們落實成實際可跑的 compose 檔案。
3. **決定 Catalog/WMS 呼叫失敗時的前台降級行為**——直接影響買家體驗，且是拆分 WMS 直接產生的新風險。
4. **定案 `StoreSettings` 歸屬服務**——影響後續 API 開發分工，屬於小決策但會卡住實作排程。
5. **單一 VPS 的備份/災難復原策略**——決策集中風險到一台主機，上線前必須有備份與還原演練規劃，不是可以無限期擱置的項目。
6. ~~共用函式庫的資安修補傳播機制~~（見 §9）——**已解決**：[29-shared-service-conventions.md](29-shared-service-conventions.md) 新增 §4.1，訂出 7 個日曆天強制升級窗口，區分一般版本更新與資安修補。
7. ~~熱銷排行/付款分布的圖表函式庫選型~~——**已解決**：[22-service-analytics.md](22-service-analytics.md) §4 採 Chart.js（`react-chartjs-2`），與 Lightweight Charts 職責互補。
8. ~~補齊 Gateway 匿名端點速率限制~~——**已解決**：[25-service-gateway.md](25-service-gateway.md) 新增 §3.1，見 §11。
9. ~~補上賣家後台 Promotions/Shipping/Reviews 三個服務的管理介面規格~~——**已解決**：[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1 已補上對應列，見 §12。
10. 效期商品自動化、多倉支援、防詐機制、網域/憑證管理、CI/CD、高權限帳號 2FA、本機多 repo 開發流程——屬於功能性增強或維運細節，可排入下一輪迭代，非規格階段必須解決。
11. ~~訪客升級為會員的機制與信箱驗證時機~~（見 §14）——**已解決**：[11-service-identity.md](11-service-identity.md) §5.1 定案沿用同一 `User.Id`＋密碼暫存至驗證通過才生效。

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
| ~~ShyeCMS 的認證方式未指定~~ | **已解決**：[31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md) §1 定案採 JWT Bearer（獨立簽發，與電商平台零共用），2FA 是否強制仍沿用 [29-shared-service-conventions.md](29-shared-service-conventions.md) §5 既有待決議，未隨本項一併定案 |
| **六個 repo 各自的 CI/CD 與跨 repo 版本協調都未定案（範圍擴大）** | 沿用 §3 已列的 CI/CD 缺口，[26-project-structure.md](26-project-structure.md) 決策 H 把電商平台從 1 個 repo 拆成 4 個（services/storefront/admin/`ecommerce-launch`）後，缺口從「2 條 pipeline」變成「6 條 pipeline + 1 套跨 repo 版本協調流程」：某個 repo 發新版後，`ecommerce-launch` 何時、由誰更新映像檔標籤，目前只有問題本身被寫下來（[26](26-project-structure.md) §7），還沒有答案 |
| ~~Monorepo 建置工具未選型~~ | **已解決（前提改變）**：電商平台不再是單一 monorepo——前台、後台、15 個服務已拆成三個獨立 repo（[26-project-structure.md](26-project-structure.md) 決策 H），`ecommerce-services` 內部也**不設共用 `.sln`**，改為各服務獨立建置，不需要 Nx/Turborepo 等級的跨語言建置編排工具 |

**建議**：ShyeCMS 前端規格已補齊（見上）。本節剩餘缺口只有六個 repo 的 CI/CD/版本協調 SOP，與 §3、§9 的維運面缺口性質相同，可併入同一輪處理。

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

| 已定案 | 仍待決議 |
|---|---|
| HTTPS/HSTS、密碼雜湊、SQL Injection 防護、XSS 防護（含 Markdown 管線）、CSRF、CORS 政策、敏感憑證加密、Data Protection 金鑰持久化、API 速率限制範圍、PCI DSS（金流不落地）、服務間認證（網路隔離 + JWT） | 高權限帳號 2FA（見 §6）、CSP 詳細規則、log 集中收集方案選型、依賴套件掃描是否納入 CI（CI/CD 本身仍未定案，見 §3） |

**建議**：資安基準文件已建立，但**執行面**（CI 是否真的擋得住有漏洞的依賴套件、CSP 規則是否真的夠嚴謹）要等 CI/CD 策略（§3）與各服務實際開發時才能驗證，本文件的角色是提供規則，不是保證規則會被遵守——建議正式開發階段安排至少一次滲透測試或第三方資安稽核，而非只靠文件層級的規範。

## 9. Repo 拆分（決策 H）帶來的新缺口

[26-project-structure.md](26-project-structure.md) 決策 H 把電商平台從 1 個 monorepo 拆成 4 個 repo 後，除了 §3、§6 已更新的 CI/CD 缺口，還浮現以下先前不存在的問題：

| 項目 | 說明 |
|---|---|
| **本機多 repo 開發流程未定義** | 過去 monorepo 下 `docker compose up` 一次啟動全部即可本機開發；現在 `ecommerce-services`/`ecommerce-storefront`/`ecommerce-admin` 是三個獨立 repo，開發者若要同時改動「Catalog 新增欄位 + 後台顯示該欄位」這種橫跨兩個 repo 的功能，需要並排 clone 多個 repo 並手動處理彼此依賴（如後台想測試 Catalog 未發版的新端點，`api-client` 套件版本要怎麼指到「本機開發中」的版本而非已發布版本），目前沒有規範這個流程，容易讓每個開發者各自摸索出不同做法 |
| ~~共用函式庫的安全性修補傳播沒有例外機制~~ | **已解決**：[29-shared-service-conventions.md](29-shared-service-conventions.md) 新增 §4.1，資安修補獨立分級——7 個日曆天強制升級窗口 ＋ `[SECURITY]` Release Notes 標示 ＋ 人工追蹤清單，與一般版本更新的自由升級節奏區分開來 |
| **`api-client` 套件版本與後端服務版本的相容性矩陣更複雜** | §3 已列「各服務版本不同步的實際治理」缺口；決策 H 之後多一個維度——`api-client` 套件本身也獨立版本化，一份 `api-client@3.2.0` 對應的是「呼叫哪些服務的哪個版本」需要額外追蹤，不是單純服務對服務的相容性問題 |
| **私有套件/映像檔倉庫的建置與維運成本** | 決策 H 需要私有 NuGet feed、私有 npm registry、容器映像檔倉庫三種基礎設施才能運作，這是 monorepo 時代不需要的額外維運項目，目前只在 [26](26-project-structure.md) §7 列為選型待決議，其建置與維運成本（含金錢與人力）未被評估過 |

**建議**：安全性修補傳播機制已解決（見上，[29-shared-service-conventions.md](29-shared-service-conventions.md) §4.1）。其餘兩項（`api-client` 相容性矩陣、私有倉庫維運成本）屬於開發流程/維運成本問題，可在正式建置團隊成形後排入 SOP 制定，不阻塞規格本身。

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
| §5.1 的登入/Refresh Token/密碼重設機制隱含適用 Seller/SellerStaff，但文字聚焦在「消費者會員」，易被誤讀為僅限 Buyer | [11-service-identity.md](11-service-identity.md) §5.1 開頭寫「聚焦消費者會員登入」，但 `login`/`refresh-token`/`forgot-password` 等端點是 Identity Service 對所有 `Role` 共用的機制，`Seller`/`SellerStaff` 帳號同樣是 `PasswordHash` 登入，理論上同樣適用——只有 `register`（固定建立 `Role=Buyer`）才是消費者限定，其餘端點的適用範圍需要更明確的文字釐清，避免未來誤以為要幫賣家另外設計一套登入機制 |
| **結帳 Saga 循序圖遺漏「Payment 建立失敗」分支** | [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §7、[17-service-order.md](17-service-order.md) §4 的 Mermaid 循序圖只畫出「庫存不足」「優惠券失敗」「Vendor 查詢失敗」三種失敗分支，但文字說明第 7 點明確寫「任一步驟失敗 → 觸發補償」，隱含 Payment Service 建立付款紀錄失敗時同樣要走 WMS 釋放庫存 + Promotions 還原優惠券的補償鏈——圖表沒有畫出這第四種失敗路徑，容易讓人誤以為 Payment 步驟不會失敗，或漏掉它也適用 [17-service-order.md](17-service-order.md) §4.1 新增的補償失敗統一設計 |
| ~~Gateway 路由範例與 Reviews Service 實際端點路徑不一致~~ | **已直接修正**：[25-service-gateway.md](25-service-gateway.md) §4.1 原寫「`/api/v1/orders/{id}/reviews`」與 [24-service-reviews.md](24-service-reviews.md) §4 實際端點 `POST /api/v1/orders/{subOrderId}/review` 對不上，純格式錯誤，非待決議，已訂正 |
| Notification 的「Email/簡訊是否併入本服務」待決議項急迫性提升 | 這項待決議（[23-service-notification.md](23-service-notification.md) §7）先前只是抽象的「未來可能需要」；[11-service-identity.md](11-service-identity.md) §5.1 新增的信箱驗證信/密碼重設信現在**具體依賴**這個管道才能真正寄出，不再是假設性需求——本項待決議的優先度應提升，見 §4「建議下一步」 |

**建議**：唯一牽涉帳號安全的第一項已解決；Gateway 路由錯字已直接修正；剩餘 2 項（Seller/SellerStaff 適用範圍澄清、Notification Email 管道優先度）屬於文件釐清，可視時間排入下一輪。
