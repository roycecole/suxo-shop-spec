# 29 - 跨服務共通慣例與資安基準 (Shared Service Conventions & Security Baseline)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應 [10-gap-analysis.md](10-gap-analysis.md) §1、§7 累積的多項缺口：跨服務共通慣例未定義、Markdown 處理管線各服務各自實作、資訊安全性需要正式收斂 |
| v0.2 | 2026-09-08 | ordinarycas | §3 補充「各服務文件 API 大綱的讀法」統一約定，解決 [10-gap-analysis.md](10-gap-analysis.md) §11 已列的內部端點認證註記不一致疑慮——以本節為準，不需逐服務重複載明兩層防禦 |
| v0.3 | 2026-09-09 | ordinarycas | 新增 §4.1：共用套件（`SuxoShop.Shared.*`）的資安修補強制升級窗口（7 個日曆天＋`[SECURITY]` Release Notes 標示＋人工追蹤清單，與一般版本更新的自由升級節奏區分），解決 [10-gap-analysis.md](10-gap-analysis.md) §9 已列的例外機制缺口 |
| v0.4 | 2026-09-10 | ordinarycas | §5 解決 4 項待決議：高權限帳號 2FA 定案 TOTP、結構化 log 集中收集定案 Grafana Loki、CSP 規則逐服務盤點完成、服務身分 JWT 定案不需要快取（純本機簽章運算），回應「將待決議事項列出來實作」需求 |
| v0.5 | 2026-09-10 | ordinarycas | 新增 §3.1：`SuxoShop.Shared.Security` 補上 JWT 雙金鑰輪替支援後，跨服務層級補一筆對應說明並盤點另兩類憑證（DB 密碼、金流商／LINE 憑證）的輪替方式，解決先前完全沒有任何憑證輪替流程文件的缺口（`SuxoShop.Shared.Security/README.md` 自己的說明原本明講「沒有實作金鑰輪替機制」） |

> 本文件是 15 個微服務**都必須遵守**的共通規則，不是某一個服務的規格。凡是本文件定義過的慣例，各服務文件（[11](11-service-identity.md)–[25](25-service-gateway.md)）不重複定義，只在需要偏離慣例時特別註明。

## 1. 可觀測性

### 1.1 Correlation ID

每個進入 Open API Gateway（[25-service-gateway.md](25-service-gateway.md)）的請求，若沒有帶 `X-Correlation-Id` Header 則由 Gateway 產生一個（UUID），並在往下游服務轉發時強制帶上。每個服務收到請求時：
- 若 Header 已存在，原樣沿用並寫入自己的 log。
- 對外（呼叫其他服務）時，把同一個 Correlation ID 繼續往下傳遞。
- Saga 呼叫鏈（[17-service-order.md](17-service-order.md)）內的每一步都要帶這個 ID，這是 `/internal/v1/orders/support/{id}/trace` 診斷端點能運作的前提。

### 1.2 健康檢查端點

每個服務**必須**提供兩個端點：

| 端點 | 用途 |
|---|---|
| `GET /health/live` | 存活探針，不碰資料庫，僅確認程序還在跑 |
| `GET /health/ready` | 就緒探針，含資料庫連線檢查 |

兩者分開的原因：資料庫短暫異常時不應讓容器被反覆重啟（存活探針失敗才會重啟），但應該暫停導流（就緒探針失敗只是移出負載平衡）。

### 1.3 結構化 Log 格式

統一採 JSON 結構化 log（欄位至少含：`timestamp`、`level`、`service`、`correlationId`、`message`），方便未來集中收集（如 Serilog + Seq/ELK）時可以跨服務關聯查詢，不是各服務自訂格式。

## 2. Markdown 處理管線（統一實作，避免各服務各自為政）

[12-service-catalog.md](12-service-catalog.md)（商品描述）與 [20-service-cms.md](20-service-cms.md)（CMS RichText 區塊）都需要「Markdown → HTML」轉換與輸出前清理，**不可各自實作**，理由是清理規則（哪些 HTML 標籤/屬性允許）一旦兩邊不一致，其中一邊就會變成資安破口。

**做法**：抽成一個共用的 .NET 函式庫（如 `SuxoShop.Shared.Markdown`，隨各服務的專案引用），統一提供：

```
string RenderMarkdownToSafeHtml(string markdown)
```

內部實作：
1. 用 `Markdig` 之類的函式庫把 Markdown 轉成 HTML。
2. 轉出的 HTML 一律經過白名單式 sanitizer（如 `HtmlSanitizer` NuGet 套件）過濾，只允許安全標籤（`p`、`strong`、`em`、`ul`/`ol`/`li`、`a`、`img`、標題標籤等），**不允許** `<script>`、內嵌事件屬性（`onclick` 等）、`javascript:` 開頭的連結。
3. 兩個服務都呼叫同一份邏輯，任何清理規則的調整只需要改一個地方。

## 3. 服務間認證（解決既有待決議事項）

`/internal/v1/...` 端點的存取控制，確定採**兩層防禦**：

| 層 | 做法 |
|---|---|
| 第一層：網路隔離 | `/internal/v1/...` 端點只綁定 Docker Compose 內部網路，**不對外開放連接埠**——外部（含 VPS 主機本身以外）完全連不到，這是主要防線，沿用單一 VPS + Docker Compose 拓樸（[06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.1）天然具備的隔離能力 |
| 第二層：服務身分 JWT（縱深防禦） | 呼叫方（如 Order Service 呼叫 WMS）夾帶一個內部專用的短效 JWT，內含 `service` claim（如 `service: order-service`），被呼叫方驗證此 claim 屬於允許呼叫的服務清單——即使網路隔離被繞過（如設定錯誤），這層仍能擋下非授權服務的呼叫 |

簽發方式沿用既有 Open API Gateway pilot 已驗證的模式（同一把 `Jwt:SigningKey` 簽發短效 token，見 [25-service-gateway.md](25-service-gateway.md)），不另外引入 mTLS——單一 VPS 內部網路的威脅模型，mTLS 的額外憑證管理成本換不到相應的效益。

**各服務文件 API 大綱的讀法（統一約定，解決註記不一致的疑慮）**：任何端點只要標示為「內部」或路徑帶 `/internal/v1/...` 前綴，就代表**兩層防禦同時適用**，不需要每個服務文件都重複寫一次「網路隔離＋服務身分 JWT」。若某端點在「內部」之外額外註明「僅限某服務呼叫」（如「內部（僅 Order Service 可呼叫）」），代表的是第二層服務身分 JWT 的 `service` claim 只允許該服務通過，是對本節規則的**進一步限縮**，不是另一套獨立機制；沒有額外註明時，預設允許任何持有效服務身分 JWT 的內部服務呼叫。

### 3.1 憑證輪替

本節解決一項先前完全空白的缺口：平台用到的共用憑證（JWT 簽章金鑰、DB 密碼、金流商／LINE 憑證）都沒有任何輪替流程文件，`SuxoShop.Shared.Security` 自己的 README 原本就明講「沒有實作金鑰輪替機制」。三類憑證的輪替方式並不相同：

| 憑證類型 | 輪替方式 | 停機/中斷影響 |
|---|---|---|
| JWT 簽章金鑰（`Jwt:SigningKey`，本節/§3 所述的服務身分 JWT 與使用者 JWT 共用同一把） | `SuxoShop.Shared.Security` 已支援雙金鑰驗證：`TokenValidationParameters.IssuerSigningKeys`（複數）同時接受現行與選填的「上一代」（`Jwt:PreviousSigningKey`）兩把候選金鑰，簽發端永遠只用現行金鑰簽新 token。輪替步驟：① 現行金鑰移入 `Jwt:PreviousSigningKey`、設定新的 `Jwt:SigningKey` ② 15 個服務**不需要同時重啟**（滾動式部署即可，交接期間新舊 token 都能互相驗證通過）③ 等過最長的 Access Token 存活時間（服務身分 JWT 5 分鐘、使用者 JWT 30 分鐘——注意使用者的 30 天 Refresh Token 不是 JWT，是雜湊比對的隨機字串，不受此影響）④ 清空 `Jwt:PreviousSigningKey`，輪替才算完成。刻意只留一代（不是清單），逼迫每次輪替都要走完清空這一步。完整實作細節與測試見 `SuxoShop.Shared.Security/README.md`「金鑰輪替」一節 |
| DB 密碼（`ConnectionStrings__Postgres`） | 沒有熱重載機制——連線字串來自 Docker Compose 環境變數，容器生命週期內固定不變，換密碼一定需要 `ALTER USER` 之後同步更新所有服務的環境變數並重建容器 | 有，重建期間服務短暫不可用 |
| 金流商憑證（[18-service-payment.md](18-service-payment.md)）／LINE Channel Secret（[23-service-notification.md](23-service-notification.md)） | 不需要額外機制——這類憑證本來就以 ASP.NET Core Data Protection 加密存放（見 §4「敏感憑證加密」），Data Protection 原生支援多代金鑰解密，只要金鑰持久化 volume 沒被清空（§4「Data Protection 金鑰持久化」），透過既有設定 API 直接更新憑證值即可，舊版加密內容仍可正常解密 | 無 |

**已知缺口**：`services/cart` 的使用者 JWT 驗證是骨架階段遺留的手刻邏輯，未透過本套件的 `AddSuxoShopUserAuthentication` 註冊，尚未套用雙金鑰驗證——真的執行 JWT 金鑰輪替時，Cart Service 可能在交接期間出現間歇性 401，見 [10-gap-analysis.md](10-gap-analysis.md)。

## 4. 資安基準（所有服務適用）

| 項目 | 規則 |
|---|---|
| 傳輸層 | 全站強制 HTTPS，啟用 HSTS |
| 密碼儲存 | ASP.NET Core Identity 預設的雜湊機制（bcrypt/Argon2 等級），不自行實作雜湊 |
| SQL Injection | 一律透過 EF Core 參數化查詢，**禁止**字串拼接組 SQL；唯一例外是既有的原子庫存扣減（[13-service-wms.md](13-service-wms.md)）等必要的 Raw SQL，需個別 Code Review 把關 |
| XSS | React 預設跳脫 + 本文件 §2 的 Markdown 清理管線 + 回應加上 `Content-Security-Policy`、`X-Content-Type-Options: nosniff` 等安全標頭 |
| CSRF | API 一律用 JWT Bearer Token（非 Cookie-based Session），天然降低 CSRF 風險；訪客購物車 Cookie（[15-service-cart.md](15-service-cart.md)）需設定 `SameSite=Lax` |
| CORS | 每個客戶部署**明確列出**允許的來源網域（自己的前台/後台網域），**禁止**萬用字元 `*` |
| 敏感憑證加密 | 金流商金鑰（[18-service-payment.md](18-service-payment.md)）、儲存後端憑證（[19-service-media.md](19-service-media.md)）、LINE Channel Secret（[23-service-notification.md](23-service-notification.md)）皆以 ASP.NET Core Data Protection 加密儲存 |
| **Data Protection 金鑰持久化（重要）** | Data Protection 金鑰**必須**掛載到持久化磁碟區，**不可**留在容器預設的檔案系統——容器重建若金鑰未持久化，所有已加密的憑證會全部無法解密，這是實際發生過的既知風險等級問題，不是理論疑慮 |
| API 速率限制 | 不只 Open API Gateway 的金鑰限流（[25-service-gateway.md](25-service-gateway.md)），**未經認證的公開端點**（登入、訪客結帳、訪客查單）也要有速率限制，防止暴力破解與帳號列舉攻擊 |
| 高權限帳號 | ShyeCMS 的 `SuperAdmin`、電商平台的 `PlatformSupportStaff`（[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §4）**建議**強制 2FA，本輪列為待決議（見 §5），非本文件確定的硬性規定 |
| 依賴套件掃描 | CI/CD（既有缺口）應納入 NuGet/npm 套件的已知漏洞掃描（如 `dotnet list package --vulnerable`、`npm audit`），定期執行而非只在導入套件當下檢查一次 |
| 金流敏感資料 | 信用卡等資料不落地，交由金流商（[18-service-payment.md](18-service-payment.md)）Tokenization 處理，符合 PCI DSS 精神，本系統不儲存卡號 |

### 4.1 共用套件（`SuxoShop.Shared.*`）的資安修補強制升級窗口

[26-project-structure.md](26-project-structure.md) §4.2 把 `SuxoShop.Shared.*`（`Conventions`/`Markdown`/`Security`/`Translation`）改為版本化 NuGet 套件，讓各服務**自行決定何時升級**——這對一般版本更新是對的（保留獨立升級節奏，是決策 H 的核心用意），但**不該原封不動套用在資安修補上**：若某個共用套件修的是嚴重漏洞（如 `SuxoShop.Shared.Markdown` 的 XSS 漏洞），「各服務自行決定」代表某些服務可能長期不升級，§2、本節已建立的資安基準會被這個彈性架空。這是決策 H 為解決耦合問題而**新產生**的風險，需要一條明確的例外規則。

**分級與強制升級窗口**：

| 修補等級 | 判定標準 | 升級窗口 |
|---|---|---|
| 一般版本更新 | 不涉及已知安全漏洞的功能新增/修正 | 無強制窗口，各服務自行決定（維持既有彈性） |
| 資安修補 | 修補內容涉及已知或懷疑的安全漏洞（不論是否已取得正式 CVE 編號——內部發現的漏洞同樣適用，不能以「還沒申請 CVE」為由降級處理） | **7 個日曆天內**，所有直接依賴該套件的服務須完成升級並重新部署 |

**發布與追蹤（人工流程，非自動化機制）**：
- 資安修補版本的 NuGet 套件 Release Notes **必須**以 `[SECURITY]` 前綴標示，與一般版本更新的說明分開，方便掃過變更記錄即可辨識。
- 套件維護者發布 `[SECURITY]` 版本時，須建立一份追蹤清單，列出所有已知依賴該套件的服務（依 [26-project-structure.md](26-project-structure.md) §3.1 的 `shared/` 套件清單與各服務的套件參照回推），逐一勾選完成升級，直到全部勾完才算這次修補流程結束。這目前是唯一的強制手段，依賴人工執行——自動化（如 CI 擋下未升級到安全版本的服務）需要等 CI/CD 策略定案，見既有缺口（[10-gap-analysis.md](10-gap-analysis.md) §3）。
- **升級被阻擋時不能沉默逾期**：若某服務因修補版本含破壞性變更（Breaking Change）無法在 7 天內完成正式升級，須在同一個窗口內採取暫時緩解措施（如額外輸入驗證、暫時停用受影響功能），並在下一個發布週期內完成真正的套件升級，緩解措施與完成時程同樣記錄在追蹤清單，不能只升級不記錄或只記錄不升級。

## 5. 待決議事項
- [x] ~~高權限帳號（SuperAdmin、PlatformSupportStaff）是否強制 2FA，及採用哪種方式~~——**已解決：強制，採 TOTP**（非簡訊）。理由：TOTP（如 Google Authenticator/Authy）不需要簡訊閘道商合約與費用（呼應 [23-service-notification.md](23-service-notification.md) §7 簡訊現階段不做的同一個理由）、不依賴電信商送達率、離線也能產生驗證碼，是業界對高權限帳號的標準做法；這兩個角色能碰到所有客戶或所有訂單資料，權限範圍最大，值得要求這一步驟的登入摩擦。實作上是 Identity Service（ShyeCMS 這邊）/[11-service-identity.md](11-service-identity.md)（電商平台這邊）的登入流程各自加驗證步驟，非本文件範圍，這裡只定政策
- [x] ~~結構化 log 的集中收集方案（Serilog + Seq/ELK 等）選型~~——**已解決：Grafana Loki + Promtail**（非 ELK/Seq）。理由：ELK（Elasticsearch+Logstash+Kibana）對單一 VPS、單一客戶部署的規模是過重的方案（Elasticsearch 本身就需要可觀的記憶體）；Seq 是不錯的 .NET 生態選擇但屬於商業授權（免費層有使用限制）；Loki 專為「日誌量不大、想要輕量方案」的場景設計（索引只做標籤不做全文，儲存成本遠低於 Elasticsearch），且與本平台已經是 Docker Compose 單機部署的形態高度契合（Loki + Promtail + Grafana 三個容器即可，Grafana 還能同時拿來看其他監控指標，一魚兩吃）。各服務只需要維持既有的結構化 JSON log 輸出到 stdout（[29-shared-service-conventions.md](29-shared-service-conventions.md) §1.3 既有規範不變），Promtail 負責從 Docker log driver 收集，不需要改各服務程式碼
- [x] ~~Content-Security-Policy 的實際規則內容...需要逐服務盤點後才能定案~~——**已解決（盤點完成）**：
  
  | 對象 | 需要放行的外部來源 | 用途 |
  |---|---|---|
  | `ecommerce-storefront`（買家前台） | `form-action`：金流商網域（ECPay `payment.ecpay.com.tw`、藍新 `ccore.newebpay.com` 等，依實際簽約廠商而定） | 結帳導轉金流頁是**表單 POST 導頁**，不是 iframe 嵌入（見 [18-service-payment.md](18-service-payment.md)），所以是 `form-action` 而非 `frame-src`/`child-src` |
  | `ecommerce-storefront`／`ecommerce-admin` | `img-src`：Media Service 的實際儲存後端網域（依 [19-service-media.md](19-service-media.md) §2 選型而定，本機儲存則是同源，物件儲存則需放行對應網域） | 商品圖片等媒體檔案顯示 |
  | `ecommerce-storefront`／`ecommerce-admin` | `connect-src`：僅同源（前端一律呼叫自己 repo 的 API 代理層 `/api/*`，再由後端轉發到 Gateway，見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5，瀏覽器端不直接打 Gateway 網域） | 前端 fetch 呼叫範圍 |
  | `shyecms-admin` | `connect-src`：`shyecms-api` 的實際網域（依部署環境而定，開發環境見其 CORS 設定） | 前端呼叫後端 API |
  | 全部前端（`ecommerce-storefront`/`ecommerce-admin`/`shyecms-admin`） | `script-src`/`style-src`：`'self'`，**不允許** `unsafe-inline`（除非個別頁面有無法避免的內嵌 script，如 PWA 防閃爍腳本，屆時改用 nonce） | 一般 XSS 防護基準 |
  
  未列出的服務（純 API 後端，不直接回應瀏覽器渲染的頁面）不需要 CSP——CSP 是瀏覽器渲染 HTML 時才有意義的標頭，API 回應 JSON 不受此規範約束
- [x] ~~服務身分 JWT（§3）的簽發頻率與快取策略，避免每次內部呼叫都重新簽發造成效能負擔~~——**已解決：不需要快取，維持每次呼叫即時簽發**。理由：`ecommerce-services` 目前的實作（`ServiceTokenIssuer`/`UserTokenIssuer`）本身就是**純本機的 HMAC-SHA256 簽章運算**，不涉及任何網路往返（不像呼叫外部 OAuth 授權伺服器那種真的需要快取來省網路成本的情境），簽發一個 JWT 的運算成本是微秒等級，遠低於接下來要進行的實際 HTTP 呼叫本身——快取反而會增加程式碼複雜度（需要處理快取失效、Token 尚未過期但已被撤銷等情境）換取一個並不存在的效能問題。服務 JWT 效期已定案 **5 分鐘**（`ServiceJwtOptions.ExpirationMinutes` 預設值，短效降低外洩風險），這個頻率本身已經是合理的簽發密度，不需要額外的快取層
