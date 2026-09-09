# 21 - Shipping Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表，落實 [28-i18n.md](28-i18n.md) §3 列出但本文件尚未實作的多語系需求 |
| v0.3 | 2026-09-08 | ordinarycas | §4 API 大綱補齊運費區域/物流方式的查詢/編輯/刪除端點（原本只有建立），回應賣家後台運費規則管理需求（見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1、[10-gap-analysis.md](10-gap-analysis.md) §10） |

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
- [ ] 生鮮水果是否需要溫控物流的特殊處理邏輯，或視為一般宅配的子選項即可
- [ ] 超商取貨門市選擇（需串接電子地圖 API），目前僅支援超商代碼付款
