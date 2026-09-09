---
name: shyecms-data-model
description: "查詢 ShyeCMS 自己的資料庫實體：Client、StaffUser、SubscriptionPlan、ClientSubscription、FeatureFlag、ClientFeatureEntitlement、ClientDeployment、AuditLog。"
---

# 02 - ShyeCMS 資料模型 (Data Model)

來源文件：`docs/02-data-model.md`

## 這份文件負責回答
ShyeCMS（不是客戶電商平台）的核心實體與欄位建議，含 §0 ERD。

## 涵蓋章節
- 0. ERD
- 1. 客戶與員工
- 2. 部署清冊
- 3. 訂閱與計費
- 4. 功能開關
- 5. 與 v1 既有欄位的對照
- 6. 待決議事項

## 使用注意
- `ClientDeployment` 只是內部盤點紀錄，不是可連線的端點；`ClientFeatureEntitlement` 是商業合約紀錄，不是技術強制——修改欄位時不要加回連線用途的欄位（如 API 端點、服務憑證），那些已在 v0.2 被明確移除。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
