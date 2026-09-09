---
name: shyecms-architecture
description: "確認 ShyeCMS 與客戶電商平台之間的邊界：為什麼兩者零技術連接、功能授權如何靠人工落地，而不是自動化查詢/推送。"
---

# 01 - ShyeCMS 定位與邊界 (Scope Boundary)

來源文件：`docs/01-architecture.md`

## 這份文件負責回答
說明 v0.1 的即時連線設計為何被推翻，以及現行「零連接」邊界的具體後果與取捨。

## 涵蓋章節
- 1. 為什麼原本的設計被推翻
- 2. 現在的關係圖
- 3. 「功能授權」在沒有連接的前提下如何運作
- 4. 這個決策放棄了什麼（誠實記錄取捨）
- 5. 待決議事項

## 使用注意
- 任何提案讓 ShyeCMS 呼叫客戶平台 API、或客戶平台反查 ShyeCMS 功能開關的想法，都違反此文件鎖定的決策，需要先跟使用者確認才能修改。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
