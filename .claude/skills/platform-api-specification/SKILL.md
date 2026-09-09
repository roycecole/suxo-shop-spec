---
name: platform-api-specification
description: "查詢跨服務 API 版本控管策略（/api/v{n}，各服務各自累加版本號）與 API 文件格式規範（OpenAPI 3.0、Problem Details）。"
---

# 09 - 微服務 API 文件規範與版本控管 (API Specification)

來源文件：`docs/09-api-specification.md`

## 這份文件負責回答
所有微服務都要遵守的 API 版本控管與文件格式共通規則；各服務自己的端點清單見各自的 service-* 文件。

## 涵蓋章節
- 1. 版本控管策略
- 2. API 文件格式規範
- 3. 清單端點分頁慣例
- 4. 各服務 API 文件索引
- 5. 待決議事項

## 使用注意
- 版本號是**各服務各自累加**，不是全平台統一版號——同一部署內 Order 在 v2、CMS 還在 v1 是正常現象，不是不一致的 bug。
- §3（清單端點分頁慣例）統一所有清單類 `GET` 端點的 `page`/`pageSize`（預設 20，上限 100）/`sort` 參數與分頁 envelope（`items`/`page`/`pageSize`/`totalCount`）——各服務文件的 API 大綱看到清單端點時預設就適用本節，不會逐一重複列出這三個參數，不代表沒做分頁。
- §5 待決議事項 2 項已於 2026-09-09（v0.5）全數解決：Gateway 聚合文件呈現方式定案**單一 Swagger UI + 服務切換選單**；`/internal/v1/...` 內部 API 定案**不另外對外揭露文件**，維持程式碼 XML doc 註解＋本規格庫作為內部 Wiki 的現狀。查詢時不要以為還有未決項目。
- [25-service-gateway.md](25-service-gateway.md) 最近定案的反向代理引擎（YARP）是 Gateway 服務內部的路由實作選型，與本文件定義的 API 版本控管/文件格式規範是不同層次的關注點，本文件內容未受影響、無需交叉引用。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
