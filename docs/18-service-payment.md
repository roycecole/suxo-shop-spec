# 18 - Payment Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |

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

## 3. 資料模型

| 實體 | 說明 |
|---|---|
| Payment | OrderId、Method（CreditCard/LinePay/ATM/CVS/COD）、Provider、Status（Pending/Success/Failed/Refunded）、TransactionId、Amount/PaidAt |
| PaymentProviderSettings | 逐廠商的啟用狀態、加密後的商店代號/金鑰 |
| PaymentCallbackLog | 所有回調（含驗章失敗者）都寫入，**永不刪除**，是對帳爭議的唯一證據 |

## 4. 回調安全機制（三道防線）

| 防線 | 說明 |
|---|---|
| 驗章 | 各廠商演算法驗證簽章，失敗一律不更新訂單 |
| 金額比對 | 回調金額與原訂單差距超過 1 元即拒絕，防止竄改 |
| 重複識別 | 已付款的訂單重複回調會被識別為「已處理」而不重複記帳 |

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
| `GET /internal/v1/payments/support/{orderId}/callback-log` | 供 `PlatformSupportStaff` 查看回調紀錄，排查金流異常 | 內部 + PlatformSupportStaff |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 8. 待決議事項
- [ ] 三家廠商的沙箱實測（驗章邏輯需單元測試涵蓋，但未對接真實測試環境）
- [ ] 退款金流串接：目前狀態機有 Refunded，但沒有實際呼叫金流商退款 API
- [ ] 對帳排程：回調成功即標記 Paid，若當下資料庫寫入失敗，金流商已收款而系統未記錄，需補「對帳排程」主動向金流商查詢核對
