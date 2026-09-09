---
name: service-catalog
description: "查詢電商平台 Catalog Service（商品主檔、分類、變體、標籤、定價（不含庫存））的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 12 - Catalog Service

來源文件：`docs/12-service-catalog.md`

## 這份文件負責回答
商品主檔、分類、變體、標籤、定價（不含庫存）。

## 涵蓋章節
- 1. 職責
- 2. 資料模型
- 3. 爸芭樂案例
- 4. WooCommerce 匯出功能
- 5. API 大綱
- 6. 待決議事項（已全數解決，見下方使用注意）

## 使用注意
- **§6 商品搜尋效能已定案採 `pg_trgm` 而非 tsvector 全文檢索**：`ecommerce-services` 已在 `Product.Name` 加上 `pg_trgm` GIN 索引（Migration `AddProductNameTrigramIndex`，`CREATE EXTENSION pg_trgm` + `gin_trgm_ops`，已實測 `ILIKE` 查詢計畫可命中索引）。刻意不用 tsvector：(1) 需維持現有子字串比對語意，tsvector 斷詞/詞幹化會改變「符合」定義；(2) `ListProductsQueryHandler` 用同步 LINQ 相容記憶體假 DbContext 測試替身，PostgreSQL 專屬的 `EF.Functions.ToTsVector` 無法被假實作轉譯。既有 `Name.Contains(keyword)` 查詢程式碼原樣保留，43 個既有測試全數通過無回歸——這是已落地的技術決策，不是待研究的選項。
- **稅務欄位（Tax status/class）已定案不新增**，維持匯出固定值（台灣零售稅內含慣例、無跨稅率商品線實際需求），[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §6 同步標記已解決。§6 目前 3 項待決議（搜尋效能、稅務欄位、XSS 防護）已全數解決。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
