---
name: service-vendor
description: "查詢電商平台 Vendor Service（商店資料、子帳號、抽成設定）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 14 - Vendor Service

來源文件：`docs/14-service-vendor.md`

## 這份文件負責回答
商店資料、子帳號、抽成設定。

## 涵蓋章節
- 1. 職責
- 2. 資料模型（含 `StoreSettings` 歸屬本服務的定案，見下方使用注意）
- 3. 爸芭樂案例
- 4. API 大綱
- 5. 待決議事項（已全數解決，見下方使用注意）

## 使用注意
- **`StoreSettings` 歸屬已定案為 Vendor Service（不歸 CMS Service）**（§2、§5）：三個功能開關（`CouponModuleEnabled`/`CodPaymentEnabled`/`ReviewsVisible`）屬賣家商業規則配置，跟本服務既有的抽成費率/子帳號同屬「賣家如何營運自己商店」範疇；且 `ecommerce-services` 的 `SuxoShop.Vendor.Domain` 從骨架階段就已建在本服務底下並有真實 `GET`/`PUT /api/v1/vendor/settings` 端點，CMS Service 從未實作過。[20-service-cms.md](20-service-cms.md) §5 對應項同步標記已解決——查詢 StoreSettings 歸屬問題時不要假設還在 CMS/Vendor 之間搖擺。
- §5 待決議事項 2 項皆已解決：多賣家平台管理員角色/審核流程/費率調整 API **明確延後**到「是否開放多賣家入駐」這個更上位商業決策拍板後才設計，非本輪範圍。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
