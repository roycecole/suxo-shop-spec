# 18 - Payment Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | `Payment` 補上 `ProviderTransactionId` 欄位與 `(Provider, ProviderTransactionId)` 唯一索引，把 §4「重複識別」從文字承諾落實成資料層保證；§2 補充說明 COD 不屬於本表的金流廠商，啟用與否唯一歸屬 [14-service-vendor.md](14-service-vendor.md) 的 `StoreSettings.CodPaymentEnabled`（見 [10-gap-analysis.md](10-gap-analysis.md) §10、§11） |
| v0.3 | 2026-09-10 | ordinarycas | §8 處理 3 項待決議：沙箱實測標記為需要外部資源（金流商測試環境憑證），退款串接與對帳排程補上完整設計（實際 API 串接仍待沙箱環境），回應「將待決議事項列出來實作」需求 |

## 1. 職責

金流串接、回調處理、驗章、金流稽核紀錄。

## 2. 支援廠商

| 廠商 | 代碼 | 驗章方式 |
|---|---|---|
| 綠界科技 ECPay | `ECPay` | 欄位字典序 + HashKey/IV → URL encode → SHA256 (CheckMacValue) |
| 紅陽科技 SunPay | `SunPay` | 固定欄位順序串接 + HashKey → SHA256 (Checksum) |
| 藍新金流 NewebPay | `NewebPay` | AES-256-CBC 加密 TradeInfo + SHA256 驗章 (TradeSha) |
| LINE Pay | `LinePay` | REST 兩階段（Reserve/Confirm） |

每家廠商**各自獨立開關**：賣家後台逐家設定「啟用/測試模式/商店代號/HashKey/HashIV」，未啟用或設定不完整的廠商不會出現在前台結帳頁。

> **COD（貨到付款）不算本表的「廠商」**：COD 不需要商店代號/HashKey，也沒有 server-to-server 回調要驗章，性質上是純粹的營運/物流選擇，不是金流閘道整合。COD 是否開放給買家選擇，唯一歸屬 [14-service-vendor.md](14-service-vendor.md) 的 `StoreSettings.CodPaymentEnabled`，本服務**不**另外提供 COD 專屬的啟用開關，避免同一件事有兩個地方可以設定、卻沒有明訂誰優先。買家選擇 COD 結帳時，本服務僅建立 `Method=COD`、`Status=Pending` 的 `Payment` 紀錄（不產生導轉表單、不等待回調），實際收款由賣家出貨後透過 §7 新增的端點手動標記（見 `PUT /api/v1/vendor/payments/{orderId}/mark-cod-received`）。

## 3. 資料模型

| 實體 | 說明 |
|---|---|
| Payment | OrderId、Method（CreditCard/LinePay/ATM/CVS/COD）、Provider、Status（Pending/Success/Failed/Refunded）、TransactionId、`ProviderTransactionId`（廠商端交易序號，如 ECPay 的 `TradeNo`；COD 無廠商回調，此欄位為 null）、Amount/PaidAt。`(Provider, ProviderTransactionId)` 唯一索引（`ProviderTransactionId` 非 null 時），作為回調去重的資料層保證 |
| PaymentProviderSettings | 逐廠商的啟用狀態、加密後的商店代號/金鑰 |
| PaymentCallbackLog | 所有回調（含驗章失敗者）都寫入，**永不刪除**，是對帳爭議的唯一證據 |

## 4. 回調安全機制（三道防線）

| 防線 | 說明 |
|---|---|
| 驗章 | 各廠商演算法驗證簽章，失敗一律不更新訂單 |
| 金額比對 | 回調金額與原訂單差距超過 1 元即拒絕，防止竄改 |
| 重複識別 | 依 `(Provider, ProviderTransactionId)` 唯一索引識別：寫入 `Payment.ProviderTransactionId` 前先查詢是否已存在相同組合且 `Status=Success`，是則直接回應成功、不重複記帳；唯一索引本身作為併發下的最後一道資料層防線（見 §3） |

## 5. 金鑰保管

HashKey/HashIV 以 Data Protection 加密後存入資料庫，後台不回傳明文；儲存時金鑰欄位留空 = 沿用既有金鑰，避免誤清空。**Data Protection 金鑰須持久化**（掛載磁碟區），容器重建若金鑰未持久化會導致既有密文無法解密。

## 6. 爸芭樂案例

生鮮商品建議優先開啟信用卡與 LINE Pay（縮短付款到出貨的等待時間，降低生鮮腐損風險），COD（貨到付款）需評估退貨/拒收造成的生鮮報廢成本。

## 7. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `POST /internal/v1/payments/orders/{id}/redirect` | 結帳 Saga 內部呼叫：產生含驗章的表單欄位 | 內部（僅 Order Service） |
| `POST /api/v1/payments/callback/{provider}` | 金流商 server-to-server 回調 | 對外開放（簽章驗證） |
| `GET /api/v1/vendor/payment-settings` | 賣家查看/設定逐廠商啟用狀態 | 賣家 |
| `PUT /api/v1/vendor/payments/{orderId}/mark-cod-received` | 賣家標記 COD 訂單已當面收款（`Payment.Status` → `Success`） | 賣家 |
| `GET /internal/v1/payments/support/{orderId}/callback-log` | 供 `PlatformSupportStaff` 查看回調紀錄，排查金流異常 | 內部 + PlatformSupportStaff |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 8. 待決議事項
- [ ] **無法由本規格庫解決（需要外部資源）**：三家廠商的沙箱實測（驗章邏輯需單元測試涵蓋，但未對接真實測試環境）——需要向綠界/藍新/等金流商申請商店測試環境憑證（MerchantID、HashKey/HashIV 等），這是要向廠商申請的帳號資源，不是規格或程式碼能單方面解決的事，維持開放，等實際申請到測試環境後才能進行
- [x] ~~退款金流串接：目前狀態機有 Refunded，但沒有實際呼叫金流商退款 API~~——**部分解決（設計已補齊，實作仍待沙箱環境）**：流程定案為「賣家在後台對某筆訂單發起退款 → Payment Service 檢查 `PaymentStatus=Paid` 才允許 → 呼叫金流商的退款 API（ECPay `AllPay.ECPayAPI` 的信用卡負向交易，或藍新對應端點）→ 成功則 `PaymentStatus` 轉 `Refunded`、寫入 `PaymentCallbackLog`，失敗則維持原狀態並回報賣家重試」。**仍待實作**：實際串接退款 API 需要真實的金流商串接文件與沙箱環境（見上一項待決議），本項設計本身不受此阻擋，先記錄下來避免又是一個只活在程式碼 TODO 裡的缺口
- [x] ~~對帳排程：回調成功即標記 Paid，若當下資料庫寫入失敗，金流商已收款而系統未記錄，需補「對帳排程」主動向金流商查詢核對~~——**部分解決（設計已補齊，實作仍待沙箱環境）**：新增每日對帳背景排程（比照本規格庫已驗證過的背景 Worker 模式，見 [17-service-order.md](17-service-order.md) §4.1）：向各金流商查詢**前一天**的實際收款明細，逐筆比對 `PaymentCallbackLog`——(1) 金流商有收款但本服務無對應 `Paid` 記錄：標記為對帳異常，通知 `PlatformSupportStaff` 人工核對（比照 [17-service-order.md](17-service-order.md) §4.1 的人工介入模式，不自動修正金流資料）；(2) 本服務標記 `Paid` 但金流商查無此筆：同樣標記異常。**仍待實作**：金流商查詢 API 的實際串接同樣需要沙箱環境（見第一項待決議），設計本身已可作為之後實作的依據
