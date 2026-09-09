---
name: shyecms-client-lifecycle
description: "查詢客戶（如爸芭樂）從建檔、開通部署、營運中、暫停到終止合作的完整生命週期流程（純商業/人工作業，非系統整合）。"
---

# 03 - 客戶生命週期與維運流程 (Client Lifecycle)

來源文件：`docs/03-client-lifecycle.md`

## 這份文件負責回答
五個階段（Prospect → Provisioning → Active → Suspended → Terminated）各自的作業內容。

## 涵蓋章節
- 1. 生命週期總覽
- 2. 階段一：建檔（Prospect）
- 3. 階段二：開通部署（Provisioning）
- 4. 階段三：營運中（Active）
- 5. 階段四：暫停（Suspended）
- 6. 階段五：終止合作（Terminated）
- 7. 待決議事項

## 使用注意
- §7 待決議事項目前 2 項已解決：開通部署（Provisioning）維持人工 SOP、不自動化；`Suspended` 狀態下買家/賣家看到的文案已定案（買家見「本商店暫停服務中」提示、賣家後台橫幅提示聯繫拾夜科技客服）。**第 3 項「客戶終止合作的資料交還/刪除政策」尚未解決**——雖然已補上具體草案（會員/訂單資料開放終止生效日前 30 天匯出、資料庫備份保留 90 天後銷毀而非匿名化保留），仍標記需要業主/法務最終核可，查詢終止合作資料政策時不要當作已定案回答。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
