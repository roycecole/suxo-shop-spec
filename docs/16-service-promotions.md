# 16 - Promotions Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表，落實 [28-i18n.md](28-i18n.md) §3 列出但本文件尚未實作的多語系需求 |
| v0.3 | 2026-09-08 | ordinarycas | 新增 §5 待決議事項——本文件先前是唯一沒有此章節的服務規格，屬於既有疏漏 |
| v0.4 | 2026-09-08 | ordinarycas | 新增 §4 併發保護機制，套用 [13-service-wms.md](13-service-wms.md) §4 已定案的原子條件更新模式，解決 [10-gap-analysis.md](10-gap-analysis.md) §11 已列的優惠券使用次數併發缺口；§5（原 §4）API 大綱補上編輯/刪除優惠券端點，回應賣家後台優惠券管理需求（見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1） |
| v0.5 | 2026-09-09 | ordinarycas | §6 Saga 補償失敗待決議項標記已解決，統一設計見 [17-service-order.md](17-service-order.md) §4.1 |
| v0.6 | 2026-09-09 | ordinarycas | §2 補上 Coupon.VendorId 欄位；§6 解決 2 項待決議：優惠券不可疊加使用、Code 唯一性範圍為賣家範圍內唯一（皆核對 `ecommerce-services` 既有實作後定案），回應「將待決議事項列出來實作」需求 |

## 1. 職責

優惠券的建立、驗證與套用。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| Coupon | VendorId（優惠券歸屬某個賣家）、Code（**VendorId + Code 唯一**，見 §6）、DiscountType（FixedAmount/Percentage）、Amount、MinimumSpend、UsageLimit/UsageLimitPerUser、StartAt/ExpiryAt、適用範圍（分類/商品限定） |
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
- [x] ~~優惠券是否可疊加使用（一張訂單同時套用多張優惠券），目前資料模型未定義互斥/疊加規則~~——**已解決：不可疊加**。`ecommerce-services` 的結帳 Saga 契約（`CheckoutOrderCommand.CouponCode`）本來就是單一字串欄位，不是清單，一張訂單只能套用一組優惠券——與其回頭改資料模型與 Saga 契約去支援疊加，不如承認現況即為定案。日後若要開放疊加，屬於新功能而非本項待決議的範圍
- [x] ~~Saga 補償失敗時（`/internal/v1/promotions/{code}/revert` 本身失敗，如還原使用次數時資料庫異常）的處理方式~~——**已解決**：統一設計見 [17-service-order.md](17-service-order.md) §4.1（`SagaCompensationFailure` 實體＋指數退避重試＋人工介入端點），`Coupon.UsedCount` 未還原即為該設計所稱的「卡住狀態」，由 Order Service 端追蹤重試與升級，本服務不需另立一套
- [x] ~~`Coupon.Code` 的唯一性範圍：全站唯一，還是允許不同賣家各自使用相同代碼~~——**已解決：VendorId + Code 唯一（賣家範圍內唯一，非全站唯一）**。`ecommerce-services` 已這樣實作（`Coupon` 實體的唯一索引），理由：多賣家入駐後（[05-scope-and-open-items.md](05-scope-and-open-items.md) §2）每個賣家應能獨立管理自己的行銷代碼，不需要跟platform上其他不相干的賣家協調避免代碼撞名（例如兩個賣家都想用 `WELCOME10` 當歡迎折扣碼）；優惠券驗證/使用本來就是在特定賣家的商品範圍內進行（見 §4、[17-service-order.md](17-service-order.md) §4 結帳 Saga 依 SubOrder 拆分賣家），不存在「同一個代碼、不知道歸哪個賣家」的歧義。若日後改採全站唯一，只需調整索引，程式碼已預留彈性
