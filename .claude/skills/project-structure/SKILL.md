---
name: project-structure
description: "查詢完整的 repo 結構：6 個 repo（shyecms-api/shyecms-admin/ecommerce-services/ecommerce-storefront/ecommerce-admin/ecommerce-launch）各自的資料夾樹狀圖與部署單位。"
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
- **§7 待決議事項 4 項已全數解決**（v0.6–v0.8），不要以為還有未決項目：私有套件登錄選型定案 **GitHub Packages**（同時扛 NuGet 與 npm，不自架 Verdaccio/BaGet）；跨 repo CI/CD SOP 定案為 5 個有原始碼的 repo 各自用 GitHub Actions（push `main` 跑 build+test，推語意化版本 git tag 時額外建置映像檔推送 GitHub Packages）；`ecommerce-launch`（含各客戶 `ecommerce-launch-<客戶代稱>`）的版本標籤更新**現階段刻意維持人工**、不做自動化 PR bot（理由：不是每個客戶都該立刻升級，人工步驟本身就是確認關卡）；`services/*/Dockerfile` 已統一為同構 multi-stage build 並通過全服務 `docker compose up` 啟動實測。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
