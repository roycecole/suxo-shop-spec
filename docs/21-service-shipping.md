# 21 - Shipping Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表，落實 [28-i18n.md](28-i18n.md) §3 列出但本文件尚未實作的多語系需求 |
| v0.3 | 2026-09-08 | ordinarycas | §4 API 大綱補齊運費區域/物流方式的查詢/編輯/刪除端點（原本只有建立），回應賣家後台運費規則管理需求（見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1、[10-gap-analysis.md](10-gap-analysis.md) §10） |
| v0.4 | 2026-09-10 | ordinarycas | §5 解決 2 項待決議：溫控物流視為一般宅配子選項（新增 RequiresColdChain 欄位）、超商取貨門市選擇標記為需要外部資源（電子地圖 API 官方合作），回應「將待決議事項列出來實作」需求 |

## 1. 職責

物流方式與運費試算。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| ShippingZone | 地區（如本島/離島） |
| ShippingMethod | 宅配 / 超商取貨，RateRule（jsonb，如滿額免運、每件加價） |
| Translation | EntityType（"ShippingMethod"）、EntityId、LocaleCode、FieldName（顯示名稱）、Value——結構沿用 [28-i18n.md](28-i18n.md) §3 的共用模式 |

## 3. 爸芭樂案例

常溫/冷藏宅配——生鮮水果通常需要溫控物流選項，一般超商取貨（常溫）可能不適用於容易碰傷的芭樂。

## 4. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/shipping/methods` | 結帳頁查詢可用物流方式與運費 | 公開 |
| `GET /api/v1/vendor/shipping/zones` | 賣家列出自己設定的運費區域與各區域下的物流方式 | 賣家 |
| `POST /api/v1/vendor/shipping/zones` | 賣家新增運費區域 | 賣家 |
| `PUT /api/v1/vendor/shipping/zones/{id}` | 編輯運費區域 | 賣家 |
| `DELETE /api/v1/vendor/shipping/zones/{id}` | 刪除運費區域（含底下的物流方式） | 賣家 |
| `POST /api/v1/vendor/shipping/zones/{zoneId}/methods` | 於指定區域新增物流方式（宅配/超商取貨）與 `RateRule` | 賣家 |
| `PUT /api/v1/vendor/shipping/methods/{id}` | 編輯物流方式的運費規則 | 賣家 |
| `DELETE /api/v1/vendor/shipping/methods/{id}` | 刪除物流方式 | 賣家 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 5. 待決議事項
- [x] ~~生鮮水果是否需要溫控物流的特殊處理邏輯，或視為一般宅配的子選項即可~~——**已解決：視為一般宅配的子選項，不建立獨立的溫控處理邏輯**。做法：`ShippingMethod` 新增 `RequiresColdChain`（布林）欄位，賣家設定運費方式時可標記某個宅配選項為「冷藏配送」，`RateRule`（既有的 jsonb 欄位）自行納入冷藏加價；前台顯示上以一般宅配選項呈現（如「黑貓宅急便－冷藏」），買家選擇時如同選一般宅配方式。不做的原因：溫控物流的「特殊處理邏輯」（如出貨時效限制、特殊包材追蹤）屬於賣家與物流商之間的實際履約細節，這個服務只負責收費規則與方式呈現，不涉入賣家怎麼真的把貨品保冷送達——這件事本來就是子選項的價格/名稱差異，不需要在資料模型或程式碼裡開一條特殊分支
- [ ] **無法由本規格庫解決（需要外部資源）**：超商取貨門市選擇（需串接電子地圖 API）——需要向 7-11／全家等超商申請官方物流 API 合作資格（通常需要正式簽約與串接文件，非公開自助申請），目前僅支援超商代碼付款（買家自行到店輸入代碼，不需要地圖選店），這是取代「地圖選店」的可行替代方案，維持現狀直到取得官方 API 合作資格
