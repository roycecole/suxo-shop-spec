---
name: service-notification
description: "查詢電商平台 Notification Service（LINE 官方帳號整合、通知派送）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 23 - Notification Service

來源文件：`docs/23-service-notification.md`

## 這份文件負責回答
LINE 官方帳號整合、通知派送。

## 涵蓋章節
- 1. 職責
- 2. 技術方式
- 3. 資料模型
- 4. 容錯與安全
- 5. 爸芭樂案例
- 6. API 大綱
- 7. 待決議事項

## 使用注意
- §7 待決議事項 3 項本次會期皆已解決：**LINE OA 綁定定案整個站台統一一組**（非賣家各自綁定——站台與賣家目前一對一，無多品牌識別情境）；**LINE Messaging API 額度已查證官方公開方案**（免費溝通方案 200 則/月、輕用量方案 5,000 則/月免費、標準方案月費可達 30,000+ 則），爸芭樂量級估算多數落在輕用量方案內，正式上線前仍需依實際訂單量核對台灣地區確切費率；**Email 併入本服務，簡訊現階段不做**——同時解除 [11-service-identity.md](11-service-identity.md) §5.1 驗證信/密碼重設信原本卡住的缺口，供應商建議 Resend 或 AWS SES 類交易型 API，正式選型待比價。
- §6 `POST /api/v1/vendor/line-oa/bind` 雖仍掛在賣家後台（認證欄仍標「賣家」），**語意上是站台設定而非賣家個人設定**（v0.4 已訂正端點說明文字以呼應上方站台統一定案，原文字曾是舊的「賣家各自綁定」語意、未同步更新）；同版新增 `GET /internal/v1/notifications/support/failed-log` 供 `PlatformSupportStaff` 查詢推播失敗診斷。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
