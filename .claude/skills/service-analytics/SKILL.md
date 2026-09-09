---
name: service-analytics
description: "查詢電商平台 Analytics Service（報表/數據分析（唯讀））的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 22 - Analytics Service

來源文件：`docs/22-service-analytics.md`

## 這份文件負責回答
報表/數據分析（唯讀）。

## 涵蓋章節
- 1. 職責
- 2. 資料來源
- 3. 爸芭樂案例
- 4. 前端圖表函式庫
- 5. API 大綱
- 6. 待決議事項

## 使用注意
- §6 待決議事項 3 項皆已解決：**報表查詢效能**定案對已拉取的投影資料即時彙總即可、不建物化檢視（本服務本來就是輪詢/批次拉取其他服務資料建置投影，見 §1，效能疑慮不大，未來個別查詢真有效能問題再加物化檢視）；**報表匯出**設計已補齊 `GET /api/v1/vendor/analytics/export?format=csv&from=...&to=...`（CSV 優先於 Excel，比照 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §5.5 WooCommerce 匯出的背景 Worker + 下載連結模式），但 `ecommerce-services` 尚未實作此端點，屬「設計已定、實作未動」；**圖表函式庫選型**已在 §4 定案：時間序列用 TradingView Lightweight Charts，熱銷排行/付款分布用 Chart.js（`react-chartjs-2`）。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
