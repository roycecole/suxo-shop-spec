---
name: service-shipping
description: "查詢電商平台 Shipping Service（物流方式與運費試算）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 21 - Shipping Service

來源文件：`docs/21-service-shipping.md`

## 這份文件負責回答
物流方式與運費試算。

## 涵蓋章節
- 1. 職責
- 2. 資料模型
- 3. 爸芭樂案例
- 4. API 大綱
- 5. 待決議事項

## 使用注意
- §5 待決議事項僅剩 1 項真正未解決：**超商取貨門市選擇**（需串接電子地圖 API），卡在需向 7-11/全家申請官方物流 API 合作資格（外部資源，非公開自助申請），現以超商代碼付款（買家自行輸入代碼）作為替代方案維持現狀。v0.5 已補上取得合作資格後的 4 步驟就緒清單（申請 API 憑證 → 新增 `ShippingMethod` 子選項與門市搜尋 UI → 門市代號是否需擴充 `Order`/`SubOrder` 資料模型待確認 → 7-11(ibon) 與全家(FamiPort) API 格式不同需分別串接）——**這是就緒清單，不是已解決**，查詢時不要誤判為已完成。
- **溫控物流已定案**：不建立獨立處理邏輯，視為一般宅配子選項——`ShippingMethod` 新增 `RequiresColdChain`（布林）欄位，冷藏加價併入既有 `RateRule`（jsonb），前台以一般宅配選項呈現（如「黑貓宅急便－冷藏」）。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
