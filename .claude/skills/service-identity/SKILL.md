---
name: service-identity
description: "查詢電商平台 Identity Service（會員/賣家帳號、認證、JWT、地址簿）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 11 - Identity Service

來源文件：`docs/11-service-identity.md`

## 這份文件負責回答
會員/賣家帳號、認證、JWT、地址簿。

## 涵蓋章節
- 1. 職責
- 2. 資料模型
- 3. 爸芭樂案例
- 4. `PlatformSupportStaff` 角色
- 5. API 大綱（含 5.1 消費者會員註冊/登入/密碼重設流程——Email+密碼必要路徑已完整設計並實作，見下方使用注意）
- 6. 待決議事項（僅剩 LINE/Google OAuth 串接時程未解決，已補上排序建議，見下方使用注意）

## 使用注意
- **§5.1（消費者會員註冊/登入/密碼重設流程）已依此完整實作**（非僅設計）：`ecommerce-services` 完成 8 個端點真實作、`ecommerce-storefront` 完成 httpOnly Cookie BFF 前台實作，均通過瀏覽器/docker-compose 實測。除 `register`（固定建立 `Role=Buyer`）為消費者專屬外，其餘 7 個端點（login/refresh-token/logout/verify-email/resend-verification/forgot-password/reset-password）是 Identity Service 對 Buyer/Seller/SellerStaff **共用**的機制，不是只服務買家。LINE/Google 第三方登入維持**保留**狀態不變，本節不涉及。
- §6 待決議事項 4 項中 3 項已解決（Refresh Token、§5.1 適用範圍澄清、訪客升級會員的信箱驗證時機），**僅剩 LINE/Google OAuth 實際串接時程**——這是業主排程/資源分配問題非技術缺口，已補上排序建議（Email+密碼必要路徑已完整可用非阻擋性缺口；建議排在 ShyeCMS 商業條款議題之後、更接近實際客戶上線前再排入），**不是**代為排定的具體日期。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
