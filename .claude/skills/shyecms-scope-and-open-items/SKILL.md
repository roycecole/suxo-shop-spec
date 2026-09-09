---
name: shyecms-scope-and-open-items
description: "查詢 ShyeCMS 這輪明確排除的項目清單，以及僅限 ShyeCMS 範圍（00–05）的待決議事項彙整。"
---

# 05 - 本輪範圍與待決議清單 (Scope & Open Items)

來源文件：`docs/05-scope-and-open-items.md`

## 這份文件負責回答
列出本輪刻意不做的項目與原因，避免日後誤判為遺漏；並整理 ShyeCMS 範圍的待決議清單。

## 涵蓋章節
- 1. 已鎖定、不再開放討論的決策
- 2. 本輪明確排除項目
- 3. 待決議事項彙整（僅 ShyeCMS 範圍）
- 4. 建議下一步

## 使用注意
- 本文件的待決議彙整**只涵蓋 ShyeCMS（00–05）**，電商平台/微服務側的待決議事項不在這裡，見 open-decisions-register。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
