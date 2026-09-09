---
name: shyecms-frontend-requirements
description: "查詢 shyecms-admin 的前端頁面清單、拾夜科技員工操作流程、角色權限矩陣——ShyeCMS 資料模型（02）與生命週期流程（03）的 UI/UX 落地規格。"
---

# 31 - ShyeCMS 前端需求 (ShyeCMS Frontend Requirements)

來源文件：`docs/31-shyecms-frontend-requirements.md`

## 這份文件負責回答
`shyecms-admin`（ShyeCMS 前端 React SPA）實際有哪些頁面、拾夜科技員工在畫面上怎麼操作——是 [02-data-model.md](../../../docs/02-data-model.md)（資料模型）與 [03-client-lifecycle.md](../../../docs/03-client-lifecycle.md)（生命週期流程）的 UI/UX 落地規格。內容屬 ShyeCMS（`00`–`05`），但編號延續在 `30` 之後（見文件開頭說明，`06`–`30` 已被電商平台文件佔用）。

## 涵蓋章節
- 0. 定位聲明
- 1. 使用者與登入
- 2. 站台地圖（Mermaid）
- 3. 頁面清單總覽
- 4. 客戶管理（`clients/`）
- 5. 訂閱與計費（`subscriptions/`）
- 6. 功能授權（`entitlements/`）
- 7. 部署盤點（`deployments/`）
- 8. 稽核紀錄（`audit-log/`）
- 9. 儀表板（首頁）
- 10. 內部人員管理（`staff/`，新增模組）
- 11. 角色權限矩陣
- 12. API 大綱（對照 `shyecms-api`）
- 13. RWD / PWA
- 14. 待決議事項

## 使用注意
- 功能授權（§6）與客戶狀態變更（§4.4）頁面的**強制警語文案**是本文件的核心要求——這兩處是整份規格集最容易被誤解為「技術強制」的操作，UI 必須主動提醒「僅更新 ShyeCMS 紀錄，不會同步到客戶環境」，不能只依賴文件層級的說明（呼應決策 C，見 [01-architecture.md](../../../docs/01-architecture.md)）。
- §11 角色權限矩陣是方向性建議，非最終定案；§1 認證機制（JWT）已定案，但 2FA 是否強制仍是 [29-shared-service-conventions.md](../../../docs/29-shared-service-conventions.md) §5 的待決議，兩者不要混為一談。
- `shyecms-admin` 是拾夜科技單一內部實例，不是逐客戶部署的白牌產品——修改本文件時不要引入「多租戶」或「客戶自訂」類的設計，那是電商平台前後台（`ecommerce-storefront`/`ecommerce-admin`）的模式。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到 [26-project-structure.md](../../../docs/26-project-structure.md) §2.2 的 `shyecms-admin/features/` 樹狀圖，需同步調整資料夾清單。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
