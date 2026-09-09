# 16 - Promotions Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表，落實 [28-i18n.md](28-i18n.md) §3 列出但本文件尚未實作的多語系需求 |
| v0.3 | 2026-09-08 | ordinarycas | 新增 §5 待決議事項——本文件先前是唯一沒有此章節的服務規格，屬於既有疏漏 |
| v0.4 | 2026-09-08 | ordinarycas | 新增 §4 併發保護機制，套用 [13-service-wms.md](13-service-wms.md) §4 已定案的原子條件更新模式，解決 [10-gap-analysis.md](10-gap-analysis.md) §11 已列的優惠券使用次數併發缺口；§5（原 §4）API 大綱補上編輯/刪除優惠券端點，回應賣家後台優惠券管理需求（見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1） |

## 1. 職責

優惠券的建立、驗證與套用。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| Coupon | Code、DiscountType（FixedAmount/Percentage）、Amount、MinimumSpend、UsageLimit/UsageLimitPerUser、StartAt/ExpiryAt、適用範圍（分類/商品限定） |
| Translation | EntityType（"Coupon"）、EntityId、LocaleCode、FieldName（如優惠券顯示文案）、Value——結構沿用 [28-i18n.md](28-i18n.md) §3 的共用模式 |

## 3. 爸芭樂案例

如「產季優惠」「滿千免運」等折扣券。

## 4. 併發保護機制

優惠券使用次數（`Coupon.UsedCount`）比照 [13-service-wms.md](13-service-wms.md) §4 已定案的**原子條件更新**模式，避免併發結帳導致超用：

```sql
UPDATE Coupons SET UsedCount = UsedCount + 1 WHERE Id = @CouponId AND UsedCount < UsageLimit
```

影響列數為 0 即代表已達使用上限，`/internal/v1/promotions/validate` 應回傳結帳失敗（優惠券已達使用上限），觸發 Saga 走「優惠券失敗」分支（見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §7）。`UsageLimitPerUser` 的每人限制需額外查詢該使用者的歷史使用次數，屬於原子更新之外的追加檢查，仍應在同一個資料庫交易內完成，避免 TOCTOU 競態。

## 5. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/vendor/coupons` | 賣家查看/管理優惠券 | 賣家 |
| `POST /api/v1/vendor/coupons` | 建立優惠券 | 賣家 |
| `PUT /api/v1/vendor/coupons/{id}` | 編輯優惠券（折扣內容、期限、適用範圍） | 賣家 |
| `DELETE /api/v1/vendor/coupons/{id}` | 刪除/停用優惠券 | 賣家 |
| `POST /internal/v1/promotions/validate` | 結帳 Saga 內部呼叫：驗證優惠券並計算折扣、使用次數 +1（原子更新，見 §4） | 內部（僅 Order Service） |
| `POST /internal/v1/promotions/{code}/revert` | Saga 補償：還原優惠券使用次數 | 內部 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 6. 待決議事項
- [ ] 優惠券是否可疊加使用（一張訂單同時套用多張優惠券），目前資料模型未定義互斥/疊加規則
- [ ] Saga 補償失敗時（`/internal/v1/promotions/{code}/revert` 本身失敗，如還原使用次數時資料庫異常）的處理方式——[17-service-order.md](17-service-order.md) §6 已列出同類問題，但只在 Order 自己的文件提及；本服務身為 Saga 參與者同樣會遇到，應個別確認
- [ ] `Coupon.Code` 的唯一性範圍：全站唯一，還是允許不同賣家各自使用相同代碼（若未來開放多賣家入駐，見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2）
