---
name: service-gateway
description: "查詢電商平台 Open API Gateway（對外入口、路由/聚合、金鑰驗證）的職責、資料模型、爸芭樂案例與 API 大綱。"
---

# 25 - Open API Gateway

來源文件：`docs/25-service-gateway.md`

## 這份文件負責回答
對外入口、路由/聚合、金鑰驗證。

## 涵蓋章節
- 1. 職責
- 2. 對內路由 vs 對外公開 API
- 3. 金鑰與權限
- 4. 對內路由設計原則（含 4.1 反向代理引擎選型——已定案採用 YARP，見下方使用注意）
- 5. 資料模型
- 6. API 文件
- 7. 待決議事項（5 項本次會期全數已解決，見下方使用注意）

## 使用注意
- **反向代理引擎已定案採用 `Yarp.ReverseProxy`（YARP）**（§4.1，2026-09-09），取代先前手刻 `HttpClient` 轉發的過渡實作。理由：微軟官方維護成熟度（串流、hop-by-hop 標頭、HTTP/2 等轉發細節）、路由表/叢集完全設定驅動可用環境變數覆寫下游位址、避免自建轉發隨聚合/重試/限流需求長成第二個框架、單一 VPS 單客戶規模下自建「省一個依賴」的優勢不敵維護成本。**落地現況**：Gateway 已改用 YARP，路由表涵蓋全部 14 個領域服務的公開 `/api/v1/*` 前綴；金鑰驗證（§3）、聚合、限流（§3.1）仍未實作，維持待辦——不要把「引擎選型定案」誤讀成「Gateway 功能已完工」。
- §7 待決議事項其餘 4 項同期解決：速率限制採**記憶體計數**（非 Redis，單一 VPS 單執行個體無需跨執行個體共享計數）；Webhook 主動推播**現階段不需要**（無實際外部系統提出整合需求）；公開 API **不開放**建立訂單（維持唯讀＋出貨更新，避免繞過 storefront 端的庫存/價格一致性把關）；聚合文件採**單一 Swagger UI + 服務切換選單**（`swagger-ui` 原生多 `urls` 設定，非每服務各自子網址）。

## 共通慣例
此服務受 `docs/29-shared-service-conventions.md`（shared-service-conventions Skill）規範的跨服務共通慣例約束：Correlation ID 傳遞、`/health/live` + `/health/ready`、結構化 JSON log、Markdown 輸出消毒（若適用）、服務間內部認證。本文件未特別註明偏離的部分，一律以該文件為準，不要重複定義或另立一套。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
