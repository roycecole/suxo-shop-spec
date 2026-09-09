# 25 - Open API Gateway

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §3.1 匿名端點速率限制，補上 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 已要求、但本文件先前只涵蓋持金鑰呼叫的限流缺口（見 [10-gap-analysis.md](10-gap-analysis.md) §11） |

## 1. 職責

所有對外流量的**唯一入口**：前台/賣家後台走的內部路由，以及供外部系統（ERP、POS、比價平台等）整合的公開 API（`/api/public/v1`），皆經過本服務。負責金鑰驗證、scope 權限控管、速率限制，並路由/聚合到內部各微服務。

## 2. 對內路由 vs 對外公開 API

| 對象 | 端點前綴 | 驗證方式 |
|---|---|---|
| 前台 / 賣家後台 | `/api/v1` | JWT（`Authorization: Bearer`） |
| 外部系統整合 | `/api/public/v1` | API 金鑰（`X-Api-Key`） |

## 3. 金鑰與權限

| 權限 (Scope) | 可存取 |
|---|---|
| `CatalogRead` | 查詢商品、分類 |
| `OrdersRead` | 查詢訂單（金鑰須綁定賣家） |
| `OrdersWrite` | 更新出貨資訊 |
| `AnalyticsRead` | 查詢報表 |

- 金鑰可綁定特定賣家，資料自動限縮於該賣家；**平台層級金鑰不開放批次拉取全站訂單**，避免消費者個資大量外流。
- 金鑰只在建立當下回傳一次明文，資料庫僅存 SHA-256 雜湊。
- 每把金鑰有獨立的速率上限設定（`RateLimitPerMinute`），**必須實際強制執行**，否則分級收費（若未來與 ShyeCMS 商業模式掛鉤）沒有意義。

### 3.1 匿名端點的速率限制（回應 29 §4）

[29-shared-service-conventions.md](29-shared-service-conventions.md) §4 明訂「未經認證的公開端點（登入、訪客結帳、訪客查單）也要有速率限制」，這類請求沒有 `X-Api-Key`，不能沿用 §3 的 `ApiKey.RateLimitPerMinute` 機制，需要 Gateway 額外**依來源 IP 位址**計數：

| 端點類型 | 端點範例 | 限制基準 | 建議上限（初估，待實測校正） |
|---|---|---|---|
| 登入 | `POST /api/v1/identity/login` | 來源 IP | 10 次/分鐘 |
| 訪客結帳 | `POST /api/v1/orders/checkout`（訪客） | 來源 IP | 20 次/分鐘 |
| 訪客查單 | `GET /api/v1/orders/lookup`（訂單編號 + Email） | 來源 IP | 20 次/分鐘 |

超過上限回應 `429 Too Many Requests`（沿用 [09-api-specification.md](09-api-specification.md) 的 Problem Details 格式）。這組規則是 Gateway 層級的固定設定（不像 `ApiKey` 是逐把金鑰動態調整），資料模型見 §5 新增的 `AnonymousRateLimitRule`。計數的實際儲存方式（記憶體 vs Redis）與 §3 持金鑰限流共用同一套機制，仍是 §7 既有待決議，本節只解決「規則涵蓋範圍」，不重複解決「用什麼存」。

## 4. 對內路由設計原則

Gateway 不新開一組「內部 API」給自己呼叫，而是直接路由到各微服務既有的端點（如 `/internal/v1/...` 或帶授權的 `/api/v1/vendor/...`），避免內部端點與對外契約不同步。公開、無需授權範圍的資料（如商品查詢）直接原始位元流轉發，不重複定義回應 DTO，避免型別漂移。

## 5. 資料模型

| 實體 | 說明 |
|---|---|
| ApiKey | Scope、RateLimitPerMinute、VendorId（可為 null，代表平台層級）、KeyHash |
| AnonymousRateLimitRule（新增，見 §3.1） | EndpointPattern（如 `/api/v1/identity/login`）、LimitPerMinute、KeyBy=`IpAddress`（固定，不隨金鑰變化） |

## 6. API 文件

| 位置 | 內容 |
|---|---|
| `GET /docs` | 可讀的 HTML 文件頁 |
| `GET /openapi/v{n}.json` | 各服務聚合後的 OpenAPI 3 規格 |

## 7. 待決議事項
- [ ] 速率限制的實際執行方式（記憶體計數 vs Redis 分散式計數）
- [ ] 是否需要 Webhook 主動推播（訂單成立時通知外部系統），目前僅支援輪詢
- [ ] 是否開放建立訂單的公開 API（目前只讀不寫，避免外部系統誤建訂單）
- [ ] Gateway 聚合文件的實際呈現方式（單一 Swagger UI 選單切換服務，或每服務各自子網址）
