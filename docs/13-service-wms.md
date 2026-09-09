# 13 - WMS Service（倉儲管理）

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)、[09-api-specification.md](09-api-specification.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) 第七輪跨文件複查發現：[30-open-decisions-register.md](30-open-decisions-register.md) 一直假設本服務有「Saga 補償失敗」待決議項（§1 前 10 名、§6 重複項目都引用了 §6 的這個項目），但本文件實際上從未寫過，屬於遺漏；本次補上 |

## 1. 職責

倉儲/庫存的唯一權責服務：實際庫存量、入庫/出庫紀錄、批次與有效期（生鮮商品用）、庫存原子扣減。取代原本掛在 Catalog 底下的 `StockQuantity` 欄位。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| Inventory | ProductId/VariationId、StockQuantity、`BackorderPolicy`（enum：`NotAllowed`/`Allowed`/`AllowedWithNotify`，供 WooCommerce 匯出的 `Backorders allowed?` 欄位使用；與 `StockStatus` 的當下狀態快照語意不同，不能借用） |
| StockBatch | 入庫批次，含 BatchNo、有效期、數量（生鮮商品先進先出用） |
| StockReservation | 結帳 Saga 建立的庫存預留紀錄，含 `Released` 旗標避免重複釋放 |
| StockLedger | 庫存異動歷程（入庫/出庫/預留/釋放），供 `PlatformSupportStaff` 排查異常 |

## 3. 爸芭樂案例

芭樂為生鮮商品，依採收批次記錄有效期；結帳扣庫存、退貨回補庫存都由此服務負責，未來若快到期需搭配促銷/下架邏輯（見 §6 待決議）。

## 4. 庫存扣減機制

採**原子條件更新**（`UPDATE ... WHERE StockQuantity >= N`），由資料庫對該列加鎖，影響列數為 0 即代表庫存不足，避免併發買超。

## 5. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/wms/products/{productId}/availability` | 查詢可售庫存（前台商品頁 CSR 呼叫，見 [07-storefront-requirements.md](07-storefront-requirements.md) §4） | 公開 |
| `POST /internal/v1/wms/reservations` | 結帳 Saga 呼叫：原子扣庫存並建立預留 | 內部（僅 Order Service 可呼叫） |
| `POST /internal/v1/wms/reservations/{id}/release` | Saga 補償：還原庫存 | 內部 |
| `POST /api/v1/wms/inbound` | 賣家登記入庫（含批次號、有效期） | 賣家 |
| `GET /api/v1/wms/batches?productId=...` | 查詢商品各批次庫存與有效期 | 賣家 |
| `PUT /api/v1/wms/products/{productId}/backorder-policy` | 設定 `BackorderPolicy` | 賣家 |
| `GET /internal/v1/wms/support/stock-ledger/{productId}` | 供 `PlatformSupportStaff` 唯讀查詢庫存異動歷程，用於排查庫存異常 | 內部 + PlatformSupportStaff |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 6. 待決議事項
- [ ] 多倉支援：目前模型未區分「爸芭樂是否有多個實體倉庫」，若未來有多倉需求需加 `WarehouseId` 維度
- [ ] 效期商品的自動下架/促銷：目前只記錄有效期，沒有自動化流程
- [ ] Catalog 呼叫本服務失敗時，前台商品頁該顯示「查詢中」還是「暫時隱藏庫存」，降級行為未定義
- [ ] Saga 補償失敗的最終處理與告警機制：`POST /internal/v1/wms/reservations/{id}/release`（結帳失敗時的補償呼叫）本身若失敗，`StockReservation.Released` 會卡在未釋放狀態，目前沒有重試/告警設計——與 [17-service-order.md](17-service-order.md) §6、[16-service-promotions.md](16-service-promotions.md) §5 是同一類尚待統一解決的問題，不建議本服務自行另立一套
