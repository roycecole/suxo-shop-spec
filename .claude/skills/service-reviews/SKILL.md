---
name: service-reviews
description: "查詢電商平台 Reviews Service（商品評價）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 24 - Reviews Service

來源文件：`docs/24-service-reviews.md`

## 這份文件負責回答
商品評價。

## 涵蓋章節
- 1. 職責
- 2. 資料模型
- 3. 爸芭樂案例
- 4. API 大綱
- 5. 待決議事項

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
