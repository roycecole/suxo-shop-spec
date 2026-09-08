# 11 - Identity Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)、[09-api-specification.md](09-api-specification.md) 拆分獨立，回應「微服務拆成多個規格」需求 |

## 1. 職責

會員/賣家帳號、認證、JWT 簽發、地址簿。是整個平台唯一的身分來源，其他服務只信任由本服務簽發的 JWT，不各自管理帳密。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| User | Id、Email、PasswordHash（可為 null，訪客無密碼帳號）、Role（Buyer/Seller/SellerStaff/PlatformSupportStaff）、Status |
| ExternalLogin | User 對應的第三方登入（LINE/Google），Provider + ProviderUserId |
| Address | 收件/帳單地址，多筆對應一個 User，含 IsDefault |
| PlatformSupportStaff 帳號 | Role = `PlatformSupportStaff`，見 §4 |

## 3. 爸芭樂案例

爸芭樂買家註冊、地址簿（收芭樂的地址，需可填寫低溫宅配備註）。

## 4. `PlatformSupportStaff` 角色

拾夜科技支援人員的角色定義在此服務，用於系統異常調查，**不透過 ShyeCMS 下發**，每個客戶環境各自手動建立、各自獨立帳密。完整設計（權限範圍、稽核要求、明確排除）見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §4。此角色可存取的診斷端點分散在 Order/Payment/WMS 等服務裡，不在本服務自己的 API 清單內。

## 5. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `POST /api/v1/identity/register` | 會員註冊 | 公開 |
| `POST /api/v1/identity/login` | 登入取得 JWT | 公開 |
| `POST /api/v1/identity/guest-checkout-profile` | 訪客結帳建立無密碼資料（見 [07-storefront-requirements.md](07-storefront-requirements.md) §1） | 公開 |
| `GET /api/v1/identity/me` | 取得目前登入者資料 | 需登入 |
| `POST /api/v1/identity/external-login/{provider}` | LINE/Google 登入（**保留**，見 [07-storefront-requirements.md](07-storefront-requirements.md) §2） | 公開 |
| `DELETE /api/v1/identity/account` | 會員自助刪除帳號 | 需登入 |
| `GET /api/v1/identity/addresses` | 地址簿 CRUD | 需登入 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 6. 待決議事項
- [ ] Refresh Token 與撤銷機制（目前只發 Access Token，過期後需重新登入）
- [ ] LINE / Google OAuth 實際串接時程
