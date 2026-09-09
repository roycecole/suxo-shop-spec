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
- §6 待決議事項：`PastDue` SOP **已解決**（定案採分階段流程：逾期第 1 天自動提醒信→第 7 天人工聯繫→第 14 天視情況暫停功能→第 30 天轉終止評估，見 v0.4）；GMV 超額抽成計算依據**仍待業主決策**，但已補上查證的業界參考區間（B2C 電商 SaaS GMV 抽成 take-rate 常見 1–3%，如 Mirakl 約 2%+固定年費，見 v0.5）——這只是外部參考基準，不是規格代為挑選的具體比例，不要誤讀成已有定案數字。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
