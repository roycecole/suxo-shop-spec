# 09 - 微服務 API 文件規範與版本控管 (API Specification)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「微服務各功能 API 文件，含 CMS、會員系統、WMS、訂單系統，API 需要版本控管」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §8 Catalog Service 匯出功能 API，回應「後台可產生 WooCommerce 匯入文件」需求 |
| v0.3 | 2026-09-08 | ordinarycas | 回應「微服務拆成多個規格」需求：各服務的 API 大綱移至 [11](11-service-identity.md)–[25](25-service-gateway.md) 各自的文件，本文件只保留跨服務通用的版本控管策略與文件格式規範 |
| v0.4 | 2026-09-08 | ordinarycas | §2 補充健康檢查端點的統一聲明；新增 §3 清單端點分頁慣例（原 §3/§4 遞移為 §4/§5），解決 [10-gap-analysis.md](10-gap-analysis.md) §10/§11 已列的兩項系統性缺口 |
| v0.5 | 2026-09-09 | ordinarycas | §4 解決 2 項待決議：Gateway 聚合文件呈現方式、內部 API 是否對外揭露文件，回應「將待決議事項列出來實作」需求 |
| v0.6 | 2026-09-11 | ordinarycas | 回應「15 個服務的 Problem Details／列舉序列化各自為政，需統一並落地」需求：新增 §2.1（Problem Details 實作機制——盤點發現改動前實際只有 Payment 真正生效，Vendor 呼叫過 `AddProblemDetails()` 但缺 `app.UseExceptionHandler()` 從未攔截過例外；本輪經 `SuxoShop.Shared.Conventions` 的 `AddSuxoShopProblemDetails()`/`AddSuxoShopApiConventions()` 統一補齊 15 服務，並補上 `traceId`/`correlationId`/`service` 三個 extension 欄位）、§2.2（列舉值 JSON 序列化慣例：canonical camelCase + 5 項已知例外技術債，逐一附上 `ecommerce-admin` 前台既有依賴的查證依據，待前台改版後才能移除） |

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
- 錯誤格式統一採 **Problem Details (RFC 7807)**，微服務拆分後每個服務都要遵守同一格式，避免串接方需要為不同服務寫不同的錯誤處理邏輯。實作機制與未預期例外的處理見 §2.1。
- 列舉（enum）欄位的 JSON 序列化格式見 §2.2——這是 §2.1 之外另一個「15 個服務原本各自為政」的落差，本輪（2026-09-11）一併收斂。
- 每個服務都必須提供 `/health/live`、`/health/ready`（見 [29-shared-service-conventions.md](29-shared-service-conventions.md) §1.2），這是**所有服務共通適用**的規則。各服務自己的 API 大綱表格**一律不重複列出**這兩個端點——看到某服務文件裡沒寫健康檢查端點，不代表該服務沒做，一律以本規則為準。

### 2.1 錯誤回應格式（Problem Details）——實作機制

上方「統一採 Problem Details」原本只是格式宣告，本輪（2026-09-11）盤點 `ecommerce-services` 實際程式碼發現：15 個服務中只有 2 個（Vendor、Payment）呼叫過 `AddProblemDetails()`，其餘 13 個完全沒有——且 Vendor 那次呼叫**從未真正生效過**（少了 `app.UseExceptionHandler()` 這個實際攔截例外的中介軟體，`AddProblemDetails()` 單獨呼叫只是註冊了 `IProblemDetailsService`，沒人用到它）。換句話說，改動前實際上只有 Payment 一個服務的未預期例外會得到 Problem Details 形狀的回應，其餘 14 個服務逃出全部 `catch` 區塊的未預期例外，會落到 ASP.NET Core 預設的原始例外回應（Development 環境含完整堆疊追蹤、正式環境是空白 500）。

**現況（本輪已落地）**：`SuxoShop.Shared.Conventions` 函式庫新增 `AddSuxoShopProblemDetails()` 擴充方法（`ApiConventionsExtensions` 類別），15 個服務的 `Program.cs` 一律呼叫此方法（或包含它的 `AddSuxoShopApiConventions()`），**外加** `app.UseExceptionHandler()`——兩者缺一不可，這正是 Vendor 先前踩過的坑。統一後的 Problem Details 回應除了標準的 `type`/`title`/`status` 欄位，額外補上三個 extension 欄位：

| 欄位 | 來源 | 用途 |
|---|---|---|
| `traceId` | `HttpContext.TraceIdentifier` | 對照該服務自己的 log |
| `correlationId` | `ICorrelationIdAccessor`（[29-shared-service-conventions.md](29-shared-service-conventions.md) §1.1，未註冊時優雅省略，不丟例外） | 對照跨服務請求鏈中**其他服務**的 log——結帳 Saga 這類跨服務呼叫鏈失敗時特別有用 |
| `service` | `SuxoShopServiceIdentity`（[29-shared-service-conventions.md](29-shared-service-conventions.md) §1.3 結構化 log 用的同一個服務識別，未註冊時優雅省略） | 呼叫端一次打中多個服務時，能分辨這個錯誤回應是哪一個服務產生的 |

**刻意不做的事**：不會把例外訊息／堆疊追蹤塞進回應本文（不論任何環境）——統一的只是回應「形狀」，不是把內部實作細節暴露給呼叫端；未預期例外仍照常被記錄為 Error 等級 log。既有的「服務自己已預期並手動轉譯」的錯誤處理（如 Payment 的 `NotImplementedExceptionHandler` 窄範圍攔截特定例外型別）不受影響、可與這個平台層級的安全網並存。

### 2.2 列舉值 JSON 序列化慣例

**Canonical 慣例（新欄位、以及本輪確認前台未依賴既有格式的服務一律適用）**：列舉序列化為 **camelCase 字串**（如 `"published"`、`"notAllowed"`），`[Flags]` 位元旗標列舉序列化為逗號分隔字串（如 `"catalogRead, ordersRead"`）——選擇理由：比底層數字可讀（第三方 Gateway API 金鑰持有者不需要另外維護一份數字對照表）、比 PascalCase 更貼近一般 JSON API 命名慣例（欄位名稱本身已是 camelCase）。實作機制：`SuxoShop.Shared.Conventions` 的 `AddSuxoShopCanonicalEnumJsonConversion()`（或包含它的 `AddSuxoShopApiConventions()`），透過 `JsonStringEnumConverter(JsonNamingPolicy.CamelCase)` 套用於 MVC JSON 選項。

**本輪盤點時的落差（改動前）**：15 個服務對列舉欄位的實際輸出同時存在 4 種不同結果——camelCase 字串（Catalog）、PascalCase 字串（Gateway、Payment，兩者皆用不帶命名策略的 `JsonStringEnumConverter`）、手動 `.ToString()` 繞過序列化器直接輸出 PascalCase 字串（Order 的 `OrderDetailDto`/`OrderDetailSubOrderDto`/`VendorOrderSummaryDto`、CMS 的 `VendorPageLayoutDto`/`VendorPageSectionDto`）、完全沒有轉換器直接輸出底層數字（WMS 的 `ProductAvailabilityDto.BackorderPolicy`、Vendor 的 `VendorStaffDto.Permissions`/`Status`，後者還是 `[Flags]` 旗標，數字更難被第三方 API 呼叫端正確解讀）。呼叫端原本完全無法只憑「這是一個列舉欄位」預期回應格式。

**已知例外（技術債，刻意保留、非遺漏）**：以下欄位目前**仍維持改動前的既有格式**，因為本輪 grep `ecommerce-admin` 原始碼確認前台有硬編碼依賴目前的確切格式（型別或大小寫），貿然切換會讓對應頁面的邏輯整個失效，需要先協調前台改版才能安全切換：

| 服務 | 欄位 | 目前格式 | 前台既有依賴（`ecommerce-admin`） |
|---|---|---|---|
| Payment | 全部列舉（`PaymentProvider` 等 6 個型別） | PascalCase 字串（不變） | `src/api/client.ts` 的 `PaymentProvider` 型別／`PaymentTab.tsx` 顯示邏輯硬編碼 PascalCase 字面值 |
| Order | `OrderDetailDto.Status`/`PaymentStatus`、`OrderDetailSubOrderDto.Status`、`VendorOrderSummaryDto.Status` | PascalCase 字串（不變，但已改回真正的列舉型別＋`[JsonConverter]` 屬性覆寫，不再手動 `.ToString()` 繞過序列化器） | `src/features/orders/OrdersPage.tsx` 對 `order.status` 做 `=== 'Shipped'`/`'Completed'` 精確比對、`src/api/client.ts` 的 `SubOrderStatus` 型別是 7 個 PascalCase 字面值的聯集 |
| CMS | `VendorPageLayoutDto.PageType`/`Status`、`VendorPageSectionDto.Type`（賣家後台端點，公開端點 `PageLayoutDto`/`PageSectionDto` 因無既有消費者已直接採用 canonical camelCase） | PascalCase 字串（不變，型別修正同上） | `src/features/cms/CmsPage.tsx` 對 `layout.status` 做 `=== 'Draft'` 精確比對、`section.type === 'RichText'`；`src/api/client.ts` 的 `CmsPageType`/`PageSectionType` 型別 |
| WMS | `ProductAvailabilityDto.BackorderPolicy`（`GET /api/v1/wms/products/{id}/availability`，前台商品頁 CSR 呼叫的公開端點） | 底層數字（不變） | `src/features/inventory/InventoryPage.tsx` 對 `availabilityItem.backorderPolicy` 做 `switch` 數字比對、`src/api/client.ts` 的 `BackorderPolicy` 型別是 `0 \| 1 \| 2` |
| Vendor | `VendorStaffDto.Permissions`（`[Flags]`）、`Status` | 底層數字（不變） | `src/features/settings/StaffTab.tsx` 直接對 `permissions` 做位元運算（`(permissions & flag) !== 0`）、`StaffForm.tsx`/`src/api/client.ts` 的 `VendorStaffStatus` 型別是 `0 \| 1` |

這 5 列是目前已知、已驗證的例外，不代表其餘服務的列舉欄位都已確認安全——本輪任務範圍明確只查證了上表列出的服務/欄位；其餘服務（Identity/Media/Notification/Shipping/Analytics/Reviews/Promotions）也有欄位目前是底層數字輸出，但**尚未逐一查證**前台依賴情形，見 [10-gap-analysis.md](10-gap-analysis.md) 對應項目。

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
