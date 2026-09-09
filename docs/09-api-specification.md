# 09 - 微服務 API 文件規範與版本控管 (API Specification)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「微服務各功能 API 文件，含 CMS、會員系統、WMS、訂單系統，API 需要版本控管」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §8 Catalog Service 匯出功能 API，回應「後台可產生 WooCommerce 匯入文件」需求 |
| v0.3 | 2026-09-08 | ordinarycas | 回應「微服務拆成多個規格」需求：各服務的 API 大綱移至 [11](11-service-identity.md)–[25](25-service-gateway.md) 各自的文件，本文件只保留跨服務通用的版本控管策略與文件格式規範 |
| v0.4 | 2026-09-08 | ordinarycas | §2 補充健康檢查端點的統一聲明；新增 §3 清單端點分頁慣例（原 §3/§4 遞移為 §4/§5），解決 [10-gap-analysis.md](10-gap-analysis.md) §10/§11 已列的兩項系統性缺口 |
| v0.5 | 2026-09-09 | ordinarycas | §4 解決 2 項待決議：Gateway 聚合文件呈現方式、內部 API 是否對外揭露文件，回應「將待決議事項列出來實作」需求 |

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
- 每個服務都必須提供 `/health/live`、`/health/ready`（見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §1.2），這是**所有服務共通適用**的規則。各服務自己的 API 大綱表格**一律不重複列出**這兩個端點——看到某服務文件裡沒寫健康檢查端點，不代表該服務沒做，一律以本規則為準。

## 3. 清單端點分頁慣例

所有回傳集合的 `GET` 端點（商品清單、批次清單、優惠券清單、賣家訂單清單等）一律遵守同一套分頁參數，不由各服務各自定義：

| Query 參數 | 說明 |
|---|---|
| `page` | 頁碼，從 `1` 開始，未帶時預設 `1` |
| `pageSize` | 每頁筆數，未帶時預設 `20`，上限 `100`（超過上限視為 `100`，不回應錯誤） |
| `sort`（可選） | 排序欄位，未帶時預設依 `CreatedAt DESC` |

回應格式統一採分頁 envelope，不直接回傳裸陣列：

```json
{
  "items": [ /* ... */ ],
  "page": 1,
  "pageSize": 20,
  "totalCount": 137
}
```

各服務文件的 API 大綱表格看到清單類 `GET` 端點時，預設即適用本節慣例，不需要逐一重複列出這三個參數；若某端點的分頁行為需要偏離本節慣例（如採 cursor-based 分頁），須在該服務文件個別註明並說明理由。

## 4. 各服務 API 文件索引

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

## 5. 待決議事項
- [x] ~~Gateway 聚合文件的實際呈現方式（單一 Swagger UI 選單切換服務，或每服務各自子網址）~~——**已解決：單一 Swagger UI + 服務切換選單**，理由與現況見 [25-service-gateway.md](25-service-gateway.md) §7
- [x] ~~內部 API（`/internal/v1/...`）是否也要對外揭露文件供拾夜科技工程團隊參考，或僅存於程式碼註解/內部 Wiki~~——**已解決：不另外揭露**，維持現狀（程式碼 XML doc 註解 + 本規格庫每份服務文件本身就是「內部 Wiki」，已逐一描述各服務的 `/internal/v1/*` 端點）。理由：`/internal/v1/*` 一律不經 Gateway 路由（見根目錄 CLAUDE.md 鎖定決策），唯一的消費者是拾夜科技自己的工程團隊，這批人本來就有本規格庫與程式碼庫的完整存取權，另外維護一份對外文件是重複勞動，沒有對應的獨立讀者
