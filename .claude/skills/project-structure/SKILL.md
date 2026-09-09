---
name: project-structure
description: "查詢完整的 repo 結構：6 個 repo（shyecms-api/shyecms-admin/ecommerce-services/ecommerce-storefront/ecommerce-admin/ecommerce-deploy）各自的資料夾樹狀圖與部署單位。"
---

# 26 - 專案結構 (Project Structure)

來源文件：`docs/26-project-structure.md`

## 這份文件負責回答
回答「總共會有哪些 repo、各自資料夾長什麼樣子」；也記錄了為何放棄單一 .sln、共用邏輯改用版本化套件（NuGet/npm）而非專案參照。

## 涵蓋章節
- 1. 總覽：六個 Repo
- 2. ShyeCMS：2 個 Repo
- 3. 電商平台：4 個 Repo
- 4. 共用邏輯的處理方式（因應 repo 拆分調整）
- 5.「15 個服務放一個 .sln 會不會影響維運」——重新檢視後的結論
- 6. 專案數量總計（因 repo 拆分更新）
- 7. 待決議事項

## 使用注意
- 建立新服務或調整 shared 函式庫時，要照 §4 的版本化套件慣例，不要用 ProjectReference 或 monorepo 路徑直接參照其他服務/repo。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
