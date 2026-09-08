# 14 - Vendor Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |

## 1. 職責

賣家商店資料、子帳號、抽成設定。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| VendorProfile | StoreName/StoreSlug、Description、LogoUrl/BannerUrl、CommissionRate、Status |
| VendorStaff | 子帳號，關聯 User，Permissions（如僅出貨、僅看報表） |
| StoreSettings（歸屬**暫定**，見 §4） | 賣家自己管理的功能開關（`CouponModuleEnabled`、`CodPaymentEnabled`、`ReviewsVisible`、`GuestCheckoutEnabled` 等），與 ShyeCMS 的合約層級授權是兩回事，見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §2 |

## 3. 爸芭樂案例

爸芭樂店家自己的商店資料。**暫定為單一賣家自營**——多賣家審核/子帳號分權等場景在此案例下不適用，若平台本身即單一賣家自營，此服務仍保留供未來多賣家擴充，見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2。

## 4. 待決議事項
- [ ] `StoreSettings` 是否應歸屬本服務而非 CMS Service（見 [10-gap-analysis.md](10-gap-analysis.md) §2 已列的未定案項目）
- [ ] 若未來開放多賣家入駐，需要補上賣家審核流程與對應的平台管理員角色（見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2）
