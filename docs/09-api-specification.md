# 09 - 微服務 API 文件規範與版本控管 (API Specification)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「微服務各功能 API 文件，含 CMS、會員系統、WMS、訂單系統，API 需要版本控管」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §8 Catalog Service 匯出功能 API，回應「後台可產生 WooCommerce 匯入文件」需求 |
| v0.3 | 2026-09-08 | ordinarycas | 回應「微服務拆成多個規格」需求：各服務的 API 大綱移至 [11](11-service-identity.md)–[25](25-service-gateway.md) 各自的文件，本文件只保留跨服務通用的版本控管策略與文件格式規範 |

> 本文件只定義 API 文件的**格式規範與版本控管策略**，是所有微服務都要遵守的共通規則。各服務自己的端點清單見下方索引。

## 1. 版本控管策略

依 URL 前綴版本控管的原則：

- **每個服務各自的 API 版本**：URL 路徑 `/api/v{n}/...`，版本號**服務各自累加**，不是全平台統一版本（例如 Order Service 可能已在 `v2`，CMS Service 還在 `v1`）——同一客戶部署內不同服務版本本來就可能不同步，本文件把它訂為**允許的常態**，而非需要避免的問題。
- **相容性策略**：新版本上線後，舊版本至少維持 **3 個月**向下相容（Deprecation Window），到期前在回應標頭加註 `Deprecation` 與 `Sunset`（比照 RFC 8594），呼叫端可提前準備遷移。
- **破壞性變更**一律走新版本號（如 `v1` → `v2`），不可在既有版本內做破壞性修改；非破壞性變更（新增欄位、新增端點）可在同版本內追加。
- **服務間內部呼叫**（非對外）：因同一客戶部署內的服務通常同批次部署，內部呼叫的版本要求可以放寬，但仍建議加版本號以支援跨服務的滾動升級（Rolling Deployment）過渡期。

## 2. API 文件格式規範

- 每個服務各自提供 **OpenAPI 3.0** 規格（`GET /openapi/v{n}.json`），Development 環境自動輸出。
- Open API Gateway（[25-service-gateway.md](25-service-gateway.md)）聚合各服務的 OpenAPI 文件，提供統一的文件入口（`/docs` 頁面模式），讓串接方（無論是爸芭樂自己的前端，或未來其他自建前端）能在一處看到所有服務的最新 API。
- 錯誤格式統一採 **Problem Details (RFC 7807)**，微服務拆分後每個服務都要遵守同一格式，避免串接方需要為不同服務寫不同的錯誤處理邏輯。

## 3. 各服務 API 文件索引

| 服務 | 文件 |
|---|---|
| Identity Service | [11-service-identity.md](11-service-identity.md) |
| Catalog Service | [12-service-catalog.md](12-service-catalog.md) |
| WMS Service | [13-service-wms.md](13-service-wms.md) |
| Vendor Service | [14-service-vendor.md](14-service-vendor.md) |
| Cart Service | [15-service-cart.md](15-service-cart.md) |
| Promotions Service | [16-service-promotions.md](16-service-promotions.md) |
| Order Service | [17-service-order.md](17-service-order.md) |
| Payment Service | [18-service-payment.md](18-service-payment.md) |
| Media Service | [19-service-media.md](19-service-media.md) |
| CMS Service | [20-service-cms.md](20-service-cms.md) |
| Shipping Service | [21-service-shipping.md](21-service-shipping.md) |
| Analytics Service | [22-service-analytics.md](22-service-analytics.md) |
| Notification Service | [23-service-notification.md](23-service-notification.md) |
| Reviews Service | [24-service-reviews.md](24-service-reviews.md) |
| Open API Gateway | [25-service-gateway.md](25-service-gateway.md) |

## 4. 待決議事項
- [ ] Gateway 聚合文件的實際呈現方式（單一 Swagger UI 選單切換服務，或每服務各自子網址）
- [ ] 內部 API（`/internal/v1/...`）是否也要對外揭露文件供拾夜科技工程團隊參考，或僅存於程式碼註解/內部 Wiki
