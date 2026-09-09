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
- 3. 各服務 API 文件索引
- 4. 待決議事項

## 使用注意
- 版本號是**各服務各自累加**，不是全平台統一版號——同一部署內 Order 在 v2、CMS 還在 v1 是正常現象，不是不一致的 bug。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
