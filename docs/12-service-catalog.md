# 12 - Catalog Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)、[09-api-specification.md](09-api-specification.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | `Description`/`ShortDescription` 儲存格式改為 Markdown，回應「後台內容編輯使用 Markdown」需求；連帶修正 WooCommerce 匯出的轉換規則 |
| v0.3 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表；§6 的 Markdown/XSS 待決議改為引用 [29-shared-service-conventions.md](29-shared-service-conventions.md) 已定案的共用管線 |
| v0.4 | 2026-09-08 | ordinarycas | §5 明確定義商品讀取端點回傳 `DescriptionHtml`（已轉換安全 HTML）而非原始 Markdown，解決 [10-gap-analysis.md](10-gap-analysis.md) §10 已列「前端可能繞過共用消毒管線」的缺口 |

## 1. 職責

商品主檔、分類、標籤、變體、定價。**不擁有實際庫存數字**——庫存權責在 [13-service-wms.md](13-service-wms.md)，Catalog 只負責商品資訊與價格展示，顯示用的可售庫存透過呼叫 WMS Service 取得。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| Product | Id、VendorId、Name/Slug、**Description/ShortDescription（Markdown 格式儲存）**、Type（Simple/Variable/Grouped）、Status、RegularPrice/SalePrice、SKU、Weight/Length/Width/Height |
| Category / ProductCategory | 階層分類（多對多），支援 Parent/Child |
| Tag / ProductTag | 標籤（多對多），無階層 |
| ProductAttribute / ProductAttributeValue | 全域屬性（如顏色、尺寸）及其可選值 |
| ProductVariation | 商品變體，對應一組屬性值組合，各自 SKU/Price（不含庫存，庫存查 WMS） |
| Translation | EntityType（"Product"/"Category"/"Tag"）、EntityId、LocaleCode、FieldName（如 Name、Description）、Value——結構沿用 [28-i18n.md](28-i18n.md) §3 的共用模式 |

> `StockQuantity`/`StockStatus` 已從本服務移除，改由 WMS Service 擁有，見 [13-service-wms.md](13-service-wms.md)。

## 3. 爸芭樂案例

芭樂品種（珍珠芭樂/帝王芭樂/天拔芭樂）、規格（如 5 斤裝/10 斤裝）、售價。

## 4. WooCommerce 匯出功能

賣家後台「匯出 WooCommerce 商品 CSV」功能的欄位對照、變體展開規則、非同步作業流程，完整規格見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §5。

**Markdown 轉換注意事項**：`Description`/`ShortDescription` 儲存為 Markdown，但 WooCommerce CSV 的 `Description`/`Short description` 欄位期待的是 **HTML**。匯出時（`POST /internal/v1/catalog/export/woocommerce-row`，見 §5）呼叫 [29-shared-service-conventions.md](29-shared-service-conventions.md) §2 的共用 Markdown 轉換函式庫，把 Markdown 轉成清理過的 HTML 再寫入 CSV，不能直接把 Markdown 原始文字塞進去，否則匯入 WooCommerce 後會顯示未轉譯的 Markdown 語法（如 `**` 符號）。匯出**只匯出繁體中文（預設語言）版本**，見 [28-i18n.md](28-i18n.md) §6。

## 5. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/products` | 商品清單（搜尋/篩選/排序，回傳 `ShortDescriptionHtml`，見下方說明） | 公開 |
| `GET /api/v1/products/{slug}` | 商品詳情（回傳 `DescriptionHtml`/`ShortDescriptionHtml`，見下方說明） | 公開 |
| `POST /api/v1/vendor/products` | 賣家新增商品 | 賣家 |
| `PUT /api/v1/vendor/products/{id}` | 賣家編輯商品 | 賣家 |
| `DELETE /api/v1/vendor/products/{id}` | 賣家刪除/封存商品（有訂單記錄者拒絕刪除，改上下架） | 賣家 |
| `POST /api/v1/vendor/products/export/woocommerce` | 建立 WooCommerce 匯出工作，立即回應 `202 Accepted` + `jobId` | 賣家 |
| `GET /api/v1/vendor/products/export/woocommerce/{jobId}` | 查詢匯出工作狀態，完成後回傳時效性下載連結 | 賣家 |
| `POST /internal/v1/catalog/export/woocommerce-row` | 背景 Worker 內部使用：組裝單一商品（含變體展開）對應的 CSV 列，內部呼叫 WMS（庫存/BackorderPolicy）與 Media（圖片排序） | 內部 |

**讀取端點的回應格式（解決 Markdown 輸出格式未定義的問題）**：`Description`/`ShortDescription` 儲存為 Markdown（見 §2），但公開讀取端點（`GET /api/v1/products`、`GET /api/v1/products/{slug}`）**一律回傳已轉換的安全 HTML**（`DescriptionHtml`/`ShortDescriptionHtml`），由 Catalog Service 在組裝回應時呼叫 [29-shared-service-conventions.md](29-shared-service-conventions.md) §2 的共用 Markdown 轉換函式庫產生，前台（Next.js）**直接渲染**這個欄位，不自行對 Markdown 做二次轉換或消毒。這是唯一正確的架構——共用轉換函式庫是 .NET 函式庫，前台的 JavaScript 執行環境本來就無法呼叫它，若讓前端各自處理 Markdown 渲染，等於逼前端另外實作一套消毒邏輯，繞開 29 §2 好不容易統一的清理規則。原始 Markdown 只在賣家編輯情境（`POST`/`PUT /api/v1/vendor/products` 的請求/回應本體）雙向傳遞，供編輯器載入/儲存用，不會出現在上述公開讀取端點的回應裡。轉換時機可由實作階段決定（每次讀取即時轉換，或寫入時預先轉換快取），不影響本節定案的介面契約。

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 6. 待決議事項
- [ ] 商品搜尋效能：關鍵字若以 `LIKE '%kw%'` 實作無法用索引，商品量成長後需改 PostgreSQL 全文檢索（tsvector + GIN 索引）
- [ ] 稅務欄位（Tax status/class）是否要正式納入 Product 欄位，或維持匯出時固定值（見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §6 待決議）
- [x] ~~前台渲染 Markdown 為 HTML 時的 XSS 防護~~——已定案採共用管線，見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §2
