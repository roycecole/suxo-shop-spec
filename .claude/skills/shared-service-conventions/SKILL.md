---
name: shared-service-conventions
description: "查詢所有 15 個微服務都必須遵守的共通慣例與資安基準：Correlation ID、健康檢查端點、結構化 log、Markdown 消毒管線、服務間認證、資安規則。"
---

# 29 - 跨服務共通慣例與資安基準 (Shared Service Conventions & Security Baseline)

來源文件：`docs/29-shared-service-conventions.md`

## 這份文件負責回答
本文件定義過的慣例，各服務文件（service-identity ... service-gateway）不會重複定義，只在需要偏離慣例時特別註明——查任何服務的可觀測性/資安相關問題前先看這裡。

## 涵蓋章節
- 1. 可觀測性
- 2. Markdown 處理管線（統一實作，避免各服務各自為政）
- 3. 服務間認證（解決既有待決議事項）
- 4. 資安基準（所有服務適用）
- 5. 待決議事項

## 使用注意
- 新增或修改任一微服務的規格時，凡涉及 log 格式、健康檢查、內部呼叫認證、Markdown 輸出清理，一律先確認是否已被本文件規範，不要讓某個服務自己另外發明一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
