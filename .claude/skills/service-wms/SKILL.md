---
name: service-wms
description: "查詢電商平台 WMS Service（倉儲管理）（倉儲/庫存：實際庫存量、批次與有效期、原子扣減）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 13 - WMS Service（倉儲管理）

來源文件：`docs/13-service-wms.md`

## 這份文件負責回答
倉儲/庫存：實際庫存量、批次與有效期、原子扣減。

## 涵蓋章節
- 1. 職責
- 2. 資料模型
- 3. 爸芭樂案例
- 4. 庫存扣減機制
- 5. API 大綱
- 6. 待決議事項（已全數解決，見下方使用注意）

## 使用注意
- §6 待決議事項 4 項已全數解決——2026-09-10 一次解決以下 3 項（Saga 補償失敗項已於 09-09 先解決，見異動紀錄 v0.3）：
  - **多倉支援現階段明確排除**：單一賣家自營，`WarehouseId` 屬無實例假設情境的預留維度，會讓 `Inventory`/`StockBatch`/`StockReservation` 三實體與既有查詢平白多一個維度；日後真有需求再加 nullable 欄位即可。
  - **效期商品新增每日自動處理背景排程**：到期前 **3 天內**標記 `StockBatch.IsNearExpiry=true`（供即期品徽章顯示、Promotions Service 可選擇性套用折扣，不強制）；**已過期**批次剩餘數量從可售庫存扣除（不計入 `Inventory.StockQuantity`）並寫入 `StockLedger`。刻意**不**自動下架整個 `Product`（那是 Catalog Service 職權，WMS 不越權）。
  - **「Catalog 呼叫本服務失敗時前台如何降級」的前提本身不成立**：`ecommerce-storefront` 的 `stock-badge.tsx` 直接呼叫本服務 `GET /api/v1/wms/products/{productId}/availability`，與 Catalog 的呼叫互相獨立；已有四態 UI（`loading`/`in-stock`/`out-of-stock`/`error`）並經 E2E 驗證。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
