# 23 - Notification Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | §4 補上具體重試參數（先前只寫「僅記錄並重試」），解決 [10-gap-analysis.md](10-gap-analysis.md) §12 已列的缺口 |

## 1. 職責

LINE 官方帳號整合（LINE Messaging API）、通知派送。讓賣家不需一直登入後台，也能即時掌握訂單。

## 2. 技術方式

| 方向 | 說明 |
|---|---|
| Push（推播） | 新訂單建立、訂單狀態變更（已出貨/已完成）時，主動推播訊息給已綁定的 LINE 官方帳號 |
| Webhook（接收） | LINE 平台將使用者傳送給官方帳號的訊息回呼到本服務，實作簡易指令（如「查訂單」「訂單 #12345」）查詢訂單狀態 |

## 3. 資料模型

| 實體 | 說明 |
|---|---|
| LineOaBinding | OwnerType（Vendor/Platform）、OwnerId、ChannelId、ChannelSecretEncrypted/ChannelAccessTokenEncrypted、Status |
| LineNotificationLog | Direction（Outbound/Inbound）、Status（Sent/Failed/Retrying）、Payload（訊息內容快照） |

## 4. 容錯與安全

- LINE API 呼叫失敗（逾時/額度限制）僅記錄並重試，**不可影響訂單本身的建立與核心流程**（容錯隔離原則）。
- **重試參數**：採指數退避，重試間隔 1 秒 → 5 秒 → 30 秒 → 2 分鐘，最多重試 **4 次**（含首次共 5 次嘗試）；全部失敗後 `LineNotificationLog.Status = Failed`，不再自動重試，僅留紀錄供賣家/`PlatformSupportStaff` 查看（訂單本身不受影響，買家/賣家可在後台自行查看訂單狀態，LINE 推播只是加值提醒）。
- Channel Secret / Channel Access Token 需加密儲存，Webhook 需驗證 LINE 簽章（`X-Line-Signature`）避免偽造請求。

## 5. 爸芭樂案例

新訂單/出貨時推播到爸芭樂綁定的 LINE 官方帳號。

## 6. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `POST /api/v1/vendor/line-oa/bind` | 賣家綁定自己的 LINE 官方帳號 | 賣家 |
| `DELETE /api/v1/vendor/line-oa/bind` | 解除綁定 | 賣家 |
| `POST /api/v1/line-oa/webhook` | LINE 平台 Webhook 接收端點 | 對外開放（簽章驗證） |
| `POST /internal/v1/notifications/order-events` | Order Service 非同步呼叫：新訂單/狀態變更通知 | 內部 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 7. 待決議事項
- [ ] LINE 官方帳號綁定歸屬：賣家各自綁定一組，或整個站台統一一組
- [ ] LINE Messaging API 額度與費用評估（免費額度有上限）
- [ ] Email/簡訊等其他通知管道是否併入本服務
