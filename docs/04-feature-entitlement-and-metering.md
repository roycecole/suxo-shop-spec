# 04 - 功能開關與用量彙總機制規格（已停用設計）

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立：規劃 ShyeCMS 與客戶 Open API Gateway 之間的功能開關查詢（下拉）與用量彙總拉取（上拉）兩條介面 |
| v0.2 | 2026-09-08 | ordinarycas | **使用者推翻此設計**：確認 ShyeCMS 不與客戶平台連接、不取得客戶商品/售價/會員資料。本文件原本規劃的兩條介面（`GET /internal/v1/clients/{clientId}/entitlements`、`GET /internal/v1/usage-summary`）**全數作廢**，整份內容移除，僅保留本頁供追溯決策變化 |

## 為什麼保留這份文件

規格變化要留下記錄，不是直接刪除改寫——本文件保留檔名與異動紀錄，讓後續閱讀的人能追溯「這裡曾經規劃過即時連線機制，後來為什麼被推翻」，避免誤以為是遺漏或憑空消失。

## 現行設計

現行設計請見：
- [01-architecture.md](01-architecture.md) — ShyeCMS 與客戶平台零連接的邊界說明，以及功能授權如何以人工方式落地
- [02-data-model.md](02-data-model.md) — `ClientFeatureEntitlement` 現在的定位（商業紀錄，非技術查詢介面）

本文件不再包含任何介面規格、端點定義或資料流程圖。
