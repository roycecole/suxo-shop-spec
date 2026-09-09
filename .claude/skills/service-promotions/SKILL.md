---
name: service-promotions
description: "查詢電商平台 Promotions Service（優惠券）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 16 - Promotions Service

來源文件：`docs/16-service-promotions.md`

## 這份文件負責回答
優惠券。

## 涵蓋章節
- 1. 職責
- 2. 資料模型（含 `CouponUsageLog` 診斷用實體，見下方使用注意）
- 3. 爸芭樂案例
- 4. 併發保護機制
- 5. API 大綱（含 `PlatformSupportStaff` 診斷端點，見下方使用注意）
- 6. 待決議事項（已全數解決）

## 使用注意
- **`CouponUsageLog` 實體（§2）與對應診斷端點（§5，2026-09-10 新增）**：`CouponUsageLog`（欄位 `CouponId`/`OrderId`/`Action`：`Used`/`Reverted`/`CreatedAt`）比照 [13-service-wms.md](13-service-wms.md) `StockLedger` 模式補上優惠券使用/還原歷程，解決先前只有 `Coupon.UsedCount` 計數器、沒有歷史軌跡可查的缺口；`GET /internal/v1/promotions/support/{code}/usage-log`（內部 + `PlatformSupportStaff`）供唯讀查詢，排查併發或補償異常。
- §6 待決議事項 3 項皆已解決：優惠券**不可疊加使用**、Saga 補償失敗統一設計（見 [17-service-order.md](17-service-order.md) §4.1）、`Coupon.Code` 唯一性範圍定案為 **`VendorId` + `Code` 賣家範圍內唯一**（非全站唯一）。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
