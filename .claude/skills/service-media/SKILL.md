---
name: service-media
description: "查詢電商平台 Media Service（檔案上傳、儲存後端、縮圖、配額）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 19 - Media Service

來源文件：`docs/19-service-media.md`

## 這份文件負責回答
檔案上傳、儲存後端、縮圖、配額。

## 涵蓋章節
- 1. 職責
- 2. 支援的儲存後端
- 3. 資料模型
- 4. 安全處理
- 5. 爸芭樂案例
- 6. API 大綱
- 7. 待決議事項

## 使用注意
- §7 待決議事項 2 項本次會期皆已處理，非仍待討論：**影片縮圖**設計已補齊（上傳時呼叫系統安裝的 `ffmpeg` 擷取第 1 秒畫面：`ffmpeg -i input.mp4 -ss 00:00:01 -vframes 1 thumbnail.jpg`，Dockerfile 需另外 `apt-get install ffmpeg`，因目前共用的 `mcr.microsoft.com/dotnet/aspnet:10.0` base image 不含 ffmpeg），但 `ecommerce-services` 尚未實作（`VendorMediaController` 目前圖片/影片走同一套流程，未區分），屬「設計已定、實作未動」；**CDN 加速定案現階段不需要**（單一 VPS、每客戶獨立部署、非高流量，直接由應用伺服器回應媒體檔案即可），不是懸而未決。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
