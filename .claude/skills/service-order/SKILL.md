---
name: service-order
description: "查詢電商平台 Order Service（訂單、子訂單、狀態機、結帳 Saga 協調者）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 17 - Order Service

來源文件：`docs/17-service-order.md`

## 這份文件負責回答
訂單、子訂單、狀態機、結帳 Saga 協調者。

## 涵蓋章節
- 1. 職責
- 2. 資料模型（含 2.1 訂單編號產生規則——已定案為永久設計，見下方使用注意）
- 3. 爸芭樂案例
- 4. 結帳 Saga（本平台唯一的跨服務交易協調流程，含 4.1 補償失敗的統一處理）
- 5. API 大綱
- 6. 待決議事項（目前僅剩「逾時未付款自動取消」以外皆已解決——該項本身也已定案，見下方）

## 使用注意
- **`OrderNumber` 格式已定案為永久設計**（`ORD{yyyyMMdd UTC}{8 碼大寫十六進位亂數}`，§2.1）：刻意不
  改成序號式編號（會洩漏營業量、需要額外集中計數基礎設施），已加防碰撞重試（`ecommerce-services` 的
  `CheckoutOrderCommandHandler.SaveOrderWithRetryAsync`，最多 3 次）。查詢/設計相關功能時不要假設
  這是暫定方案再提議重新設計。
- **`Order.Status` 有 `Failed`（Saga 服務呼叫失敗專用）與 `Cancelled`（買家逾時未付款/主動取消）兩個
  獨立狀態**，語意不同不要混用——§2、§4 Payment 失敗分支、§6 逾時未付款自動取消都用得到這個區分。
- **逾時未付款自動取消已定案**（§6）：僅線上付款適用（COD 不適用）、30 分鐘門檻、背景排程每 5 分鐘
  掃描、觸發既有補償鏈。
- Saga 循序圖（§4）目前畫出 4 種失敗分支（庫存不足/優惠券失敗/Vendor 查詢失敗/**Payment 建立失敗**），
  對應 [06-ecommerce-platform-architecture.md](../platform-architecture/SKILL.md) §7 的同一份圖——
  兩邊要保持同步，不要只改一邊。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
