# 29 - 跨服務共通慣例與資安基準 (Shared Service Conventions & Security Baseline)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應 [10-gap-analysis.md](10-gap-analysis.md) §1、§7 累積的多項缺口：跨服務共通慣例未定義、Markdown 處理管線各服務各自實作、資訊安全性需要正式收斂 |

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

## 5. 待決議事項
- [ ] 高權限帳號（SuperAdmin、PlatformSupportStaff）是否強制 2FA，及採用哪種方式（TOTP/簡訊）
- [ ] 結構化 log 的集中收集方案（Serilog + Seq/ELK 等）選型，屬於既有維運面缺口的延伸
- [ ] Content-Security-Policy 的實際規則內容（哪些外部網域允許載入資源，如金流商的付款頁 iframe）需要逐服務盤點後才能定案，不可用過於寬鬆的預設值
- [ ] 服務身分 JWT（§3）的簽發頻率與快取策略，避免每次內部呼叫都重新簽發造成效能負擔
