# 16 - Promotions Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表，落實 [28-i18n.md](28-i18n.md) §3 列出但本文件尚未實作的多語系需求 |
| v0.3 | 2026-09-08 | ordinarycas | 新增 §5 待決議事項——本文件先前是唯一沒有此章節的服務規格，屬於既有疏漏 |

## 1. 職責

優惠券的建立、驗證與套用。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| Coupon | Code、DiscountType（FixedAmount/Percentage）、Amount、MinimumSpend、UsageLimit/UsageLimitPerUser、StartAt/ExpiryAt、適用範圍（分類/商品限定） |
| Translation | EntityType（"Coupon"）、EntityId、LocaleCode、FieldName（如優惠券顯示文案）、Value——結構沿用 [28-i18n.md](28-i18n.md) §3 的共用模式 |

## 3. 爸芭樂案例

如「產季優惠」「滿千免運」等折扣券。

## 4. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/vendor/coupons` | 賣家查看/管理優惠券 | 賣家 |
| `POST /api/v1/vendor/coupons` | 建立優惠券 | 賣家 |
| `POST /internal/v1/promotions/validate` | 結帳 Saga 內部呼叫：驗證優惠券並計算折扣、使用次數 +1 | 內部（僅 Order Service） |
| `POST /internal/v1/promotions/{code}/revert` | Saga 補償：還原優惠券使用次數 | 內部 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 5. 待決議事項
- [ ] 優惠券是否可疊加使用（一張訂單同時套用多張優惠券），目前資料模型未定義互斥/疊加規則
- [ ] Saga 補償失敗時（`/internal/v1/promotions/{code}/revert` 本身失敗，如還原使用次數時資料庫異常）的處理方式——[17-service-order.md](17-service-order.md) §6 已列出同類問題，但只在 Order 自己的文件提及；本服務身為 Saga 參與者同樣會遇到，應個別確認
- [ ] `Coupon.Code` 的唯一性範圍：全站唯一，還是允許不同賣家各自使用相同代碼（若未來開放多賣家入駐，見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2）
