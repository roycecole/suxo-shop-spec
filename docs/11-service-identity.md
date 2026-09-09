# 11 - Identity Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)、[09-api-specification.md](09-api-specification.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 定義 `DELETE /api/v1/identity/account` 刪除後的關聯資料處理方式（採匿名化保留，非級聯刪除），解決 [10-gap-analysis.md](10-gap-analysis.md) §12 已列的缺口 |
| v0.3 | 2026-09-09 | ordinarycas | 回應「新增消費者會員登入，先保留 Google、Line 登入」需求：新增 §5.1 消費者會員（Email+密碼）註冊/信箱驗證/忘記密碼/Refresh Token 完整流程設計；§2 新增 `RefreshToken`/`AccountActionToken` 實體與 `User.EmailVerifiedAt` 欄位；§5 API 大綱補上 7 個新端點；§6 Refresh Token 待決議項標記已解決；LINE/Google 維持既有保留狀態不變，本輪不涉及 |
| v0.4 | 2026-09-09 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) §14 第九輪複查發現：§6 新增 2 項待決議——§5.1 適用範圍是否涵蓋 Seller/SellerStaff 待書面澄清、訪客升級為會員的自動關聯時機與信箱驗證的交互未定義（帳號冒領風險） |
| v0.5 | 2026-09-09 | ordinarycas | 使用者確認「訪客升級會員的自動關聯要等信箱驗證通過」：§5.1 新增「訪客升級為會員」設計——沿用同一 `User.Id`（訂單本來就指向它，不需搬移資料），密碼暫存於新增的 `AccountActionToken.PendingPasswordHash`，驗證通過才寫入 `User.PasswordHash`；`register` 端點依 Email 是否已是無密碼訪客帳號分兩種行為；§2 `AccountActionToken` 補上該欄位；§6 對應待決議項標記已解決 |
| v0.6 | 2026-09-09 | ordinarycas | §5.1 Refresh Token 儲存位置一段更新為「已依此實作」（原為建議）：`ecommerce-services` 完成 8 個端點真實作、`ecommerce-storefront` 完成 httpOnly Cookie BFF 前台實作，均已通過瀏覽器/docker-compose 實測，回應「把消費者會員登入串起來」與「把已經做出來但規格沒寫的東西補回文件」需求 |
| v0.7 | 2026-09-09 | ordinarycas | §5.1 開頭新增「適用範圍澄清」段落，明確 7/8 端點對 Buyer/Seller/SellerStaff 共用、僅 `register` 消費者限定；§6 對應待決議項標記已解決，回應「將待決議事項列出來實作」需求 |
| v0.8 | 2026-09-10 | ordinarycas | §6 LINE/Google OAuth 串接時程標記為需要業主決策（排程/資源分配問題，非技術決策），回應「將待決議事項列出來實作」需求 |
| v0.9 | 2026-09-10 | ordinarycas | §6 LINE/Google OAuth 串接時程補上排序建議（必要路徑 Email+密碼已完整可用，非阻擋性缺口，建議排在其他業主決策議題之後），仍未代為排定具體時程，回應「繼續補完 9 項未解決」需求 |

## 1. 職責

會員/賣家帳號、認證、JWT 簽發、地址簿。是整個平台唯一的身分來源，其他服務只信任由本服務簽發的 JWT，不各自管理帳密。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| User | Id、Email、PasswordHash（可為 null，訪客無密碼帳號）、EmailVerifiedAt（nullable，見 §5.1）、Role（Buyer/Seller/SellerStaff/PlatformSupportStaff）、Status（Active/Deleted，見 §5 帳號刪除） |
| ExternalLogin | User 對應的第三方登入（LINE/Google），Provider + ProviderUserId |
| Address | 收件/帳單地址，多筆對應一個 User，含 IsDefault |
| RefreshToken | Id、UserId、TokenHash（僅存雜湊，比照密碼雜湊原則不存明文）、ExpiresAt、RevokedAt（nullable）、CreatedAt，見 §5.1 |
| AccountActionToken | Id、UserId、Purpose（`EmailVerification`/`PasswordReset`）、TokenHash、ExpiresAt、UsedAt（nullable）、`PendingPasswordHash`（nullable，僅訪客升級為會員時使用，見 §5.1）、CreatedAt |
| PlatformSupportStaff 帳號 | Role = `PlatformSupportStaff`，見 §4 |

## 3. 爸芭樂案例

爸芭樂買家註冊、地址簿（收芭樂的地址，需可填寫低溫宅配備註）。

## 4. `PlatformSupportStaff` 角色

拾夜科技支援人員的角色定義在此服務，用於系統異常調查，**不透過 ShyeCMS 下發**，每個客戶環境各自手動建立、各自獨立帳密。完整設計（權限範圍、稽核要求、明確排除）見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §4。此角色可存取的診斷端點分散在 Order/Payment/WMS 等服務裡，不在本服務自己的 API 清單內。

## 5. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `POST /api/v1/identity/register` | 會員註冊，固定建立 `Role=Buyer`（見 §5.1） | 公開 |
| `POST /api/v1/identity/login` | 登入，同時核發 Access Token + Refresh Token（見 §5.1） | 公開 |
| `POST /api/v1/identity/refresh-token` | 以 Refresh Token 換發新 Access Token（Rotation，見 §5.1） | 公開（憑 Refresh Token） |
| `POST /api/v1/identity/logout` | 撤銷目前這組 Refresh Token（登出當前裝置） | 需登入 |
| `POST /api/v1/identity/verify-email` | 依驗證 Token 完成信箱驗證（見 §5.1） | 公開（憑 Token） |
| `POST /api/v1/identity/resend-verification` | 重新寄送信箱驗證信 | 需登入 |
| `POST /api/v1/identity/forgot-password` | 忘記密碼，寄送重設連結（見 §5.1） | 公開 |
| `POST /api/v1/identity/reset-password` | 依重設 Token 設定新密碼（見 §5.1） | 公開（憑 Token） |
| `POST /api/v1/identity/guest-checkout-profile` | 訪客結帳建立無密碼資料（見 [07-storefront-requirements.md](07-storefront-requirements.md) §1） | 公開 |
| `GET /api/v1/identity/me` | 取得目前登入者資料 | 需登入 |
| `POST /api/v1/identity/external-login/{provider}` | LINE/Google 登入（**保留**，見 [07-storefront-requirements.md](07-storefront-requirements.md) §2） | 公開 |
| `DELETE /api/v1/identity/account` | 會員自助刪除帳號（匿名化，見下方說明） | 需登入 |
| `GET /api/v1/identity/addresses` | 地址簿 CRUD | 需登入 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

### 5.1 消費者會員註冊/登入/密碼重設流程（新增）

本節具體化「消費者會員登入」的完整流程，聚焦 Email + 密碼這條**必要**路徑（見 [07-storefront-requirements.md](07-storefront-requirements.md) §2）；LINE/Google 第三方登入維持**保留**狀態不變，本節不涉及。

**適用範圍澄清（原文字聚焦消費者，容易誤讀成只服務買家）**：下方 8 個端點裡，只有 `register`（固定建立 `Role=Buyer`）是消費者專屬——賣家帳號不透過這個端點建立（見下方「註冊」段落最後一句）。**其餘 7 個端點（`login`/`refresh-token`/`logout`/`verify-email`/`resend-verification`/`forgot-password`/`reset-password`）是 Identity Service 對所有 `Role` 共用的機制**：`Seller`/`SellerStaff` 帳號同樣是 `PasswordHash` 登入、同樣核發 Access + Refresh Token、同樣能重設密碼——`ecommerce-services` 的實作本來就沒有另外為賣家設計一套登入機制，`login` 端點依 `User.Role` 決定要不要額外呼叫 Vendor Service 解析 `vendorId`（`Seller`/`SellerStaff` 才有），僅此而已，`ecommerce-services` 的 `LoginCommandHandler` 已是這樣實作。往後若要新增賣家專屬的登入限制（如強制 2FA、IP 白名單），改的是這 7 個端點的行為，不是另外新增一套「賣家登入」端點。

**註冊**：`POST /api/v1/identity/register` 提交 Email + 密碼時，先依 Email 查詢是否已存在 `User`，分兩種情況：

| 情況 | 判定 | 行為 |
|---|---|---|
| 全新 Email | 查無對應 `User` | 立即建立 `Role=Buyer`、`PasswordHash` 已設定的 `User`，**立即可登入**（沿用下方「信箱驗證（非阻擋式）」） |
| **既有訪客帳號**（曾訪客結帳過） | 已存在 `User` 且 `PasswordHash IS NULL`（[07-storefront-requirements.md](07-storefront-requirements.md) §1 訪客結帳建立的無密碼帳號） | **不立即設定 `PasswordHash`**，見下方「訪客升級為會員」 |

密碼最低要求 8 碼且需同時包含英文字母與數字，雜湊機制沿用 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 既有規範。賣家帳號建立方式不在此端點範圍——「爸芭樂」暫定單一賣家自營（見 [05-scope-and-open-items.md](05-scope-and-open-items.md) §2），不開放消費者自助註冊成賣家。

**信箱驗證（非阻擋式，僅適用全新 Email 註冊）**：註冊成功後產生 `AccountActionToken`（`Purpose=EmailVerification`，24 小時有效）並寄出驗證連結。**未驗證信箱不影響任何購買行為**——延續 [07-storefront-requirements.md](07-storefront-requirements.md) §1「免登入下單」的精神，會員身分本來就不是下單前提，未驗證只影響會員專屬功能（訂單歷史整合、未來的收藏/會員優惠）是否完整可用，不阻擋帳號本身的使用。`POST /api/v1/identity/verify-email` 驗證通過後將 `User.EmailVerifiedAt` 設為目前時間；`POST /api/v1/identity/resend-verification` 供使用者重新索取。

**訪客升級為會員（解決既有待決議事項，唯一的阻擋式驗證情境）**：沿用 [07-storefront-requirements.md](07-storefront-requirements.md) §1「訪客結帳後可用同 Email 註冊，歷史訂單自動關聯到新帳號」的承諾，但**這裡的「自動關聯」不需要額外的資料搬移**——訪客結帳建立的 `User.Id` 本來就是所有歷史 `Order.BuyerId` 指向的對象（見 [07](07-storefront-requirements.md) §1、`guest-checkout-profile`），升級只是讓**同一個 `User.Id`** 從無密碼變成有密碼，訂單從頭到尾都沒有換過擁有者。真正需要設計的是「密碼何時生效」：

1. 提交升級請求時，**不直接寫入 `User.PasswordHash`**，而是把密碼雜湊暫存於 `AccountActionToken.PendingPasswordHash`（`Purpose=EmailVerification`），並寄出驗證信——在這之前，`User` 維持原本的無密碼狀態，**不能登入**。
2. `POST /api/v1/identity/verify-email` 驗證通過時，若該 Token 帶有 `PendingPasswordHash`，才**同時**把它寫入 `User.PasswordHash` 並設定 `EmailVerifiedAt`——驗證通過的那一刻，帳號才真正「升級」為可登入的會員，能看到歷史訂單。
3. 這就是**唯一正確的安全邊界**：只要攻擊者不是信箱的實際擁有者，就永遠收不到、也點不到驗證連結，`PendingPasswordHash` 永遠不會被寫入 `User.PasswordHash`，攻擊者提交的密碼形同無效——不會有任何人能靠著「知道某個 Email 曾經訪客結帳過」就冒領對應的訂單歷史。
4. 同一 `UserId` + `Purpose=EmailVerification` 若有多筆待處理 Token（如攻擊者與真正的信箱擁有者都各自送出過一次升級請求），**新產生的 Token 使前面所有未使用的同用途 Token 失效**（`UsedAt` 直接標記，不再可驗證），避免舊連結事後被誤點造成密碼被覆蓋成攻擊者當初提交的那一組。
5. `POST /api/v1/identity/register` 在此情境下的回應與全新註冊不同（不回傳可登入的 Token，僅提示「請至信箱完成驗證」）——這會讓攻擊者間接得知「這個 Email 已經買過東西」，但這比讓攻擊者直接拿到訂單歷史的風險低得多，本文件視為可接受的取捨，不做進一步遮蔽。

**忘記密碼**：`POST /api/v1/identity/forgot-password` 產生 `AccountActionToken`（`Purpose=PasswordReset`，1 小時有效）並寄出重設連結；**無論該 Email 是否存在對應帳號，一律回傳相同的成功訊息**，避免帳號列舉攻擊（呼應 [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 既有的暴力破解/列舉防護原則）。`POST /api/v1/identity/reset-password` 驗證 Token 有效且未使用/未過期後更新 `PasswordHash`，Token 標記為已使用，並**同時撤銷該使用者所有現有 Refresh Token**（密碼重設後強制所有裝置重新登入，屬安全常規）。

**Refresh Token（解決既有待決議事項）**：`login` 同時核發 Access Token（短效，如 30 分鐘）與 Refresh Token（長效，如 30 天，`RefreshToken.TokenHash` 僅存雜湊不存明文）。`POST /api/v1/identity/refresh-token` 換發新 Access Token 時採**輪替（Rotation）**：每次使用後舊 Refresh Token 立即失效、核發新的一組，降低 Token 遭竊後被長期濫用的風險。前台儲存位置：Access Token 存於記憶體，Refresh Token 以 httpOnly、Secure（正式環境）、SameSite=Lax Cookie 存放（避免 XSS 情境下被 JS 讀取），透過 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5 既有的 Route Handler 代理模式轉發，不直接暴露給前端 JS——`ecommerce-storefront` 已依此實作（`app/api/auth/*` BFF 路由＋`lib/auth-cookie.ts`），不再只是建議，見該 repo 自己的 README「會員登入」一節。

**殘留缺口（誠實記錄）**：驗證信/重設密碼信的**實際寄送管道**依賴 Email 發送能力，但這是 [23-service-notification.md](23-service-notification.md) 既有待決議「Email/簡訊是否併入本服務」尚未定案的部分（見 [30-open-decisions-register.md](30-open-decisions-register.md) §4）——本節只設計到「產生 Token、提供驗證/重設端點」，實際寄信管道選型不在本次範圍內，該項待決議定案前，這兩個信件動作在實作上會缺一塊。

**帳號刪除的關聯資料處理（採匿名化保留，非級聯刪除）**：`DELETE /api/v1/identity/account` **不會**刪除 `User` 這一列，而是：

1. `User.Status` 改為 `Deleted`；`Email`/`PasswordHash` 清空，`ExternalLogin`、`Address` 全數刪除（這些是純個資，沒有其他服務依賴其存在）。
2. `User.Id` **保留不變**——`Order.BuyerId`、`Review.BuyerId`、`VendorStaff` 等其他服務對此 User 的外鍵參照因此不會失效，歷史訂單/評價/子帳號紀錄的完整性不受影響，比照 [17-service-order.md](17-service-order.md) `OrderNameSnapshot`/`SKUSnapshot` 保留歷史快照的精神——差別是這裡保留的是「參照關係」本身，不是欄位快照。
3. 前台/後台顯示歷史訂單/評價的買家資訊時，遇到 `User.Status = Deleted` 一律顯示「已刪除的會員」，不嘗試讀取已清空的 `Email` 等欄位。
4. 若該 User 同時是 `Seller`/`SellerStaff`（賣家角色自助刪除帳號），比照相同做法：`VendorStaff` 記錄的 `UserId` 保留，但賣家後台的操作權限因 `Status = Deleted` 而失效，不需要額外刪除 `VendorStaff` 列。
5. 此設計**不提供**真正的實體刪除（Hard Delete）——若客戶因個資法要求必須完全抹除，需另外規劃資料保留期滿後的批次清除流程，本輪不處理。

## 6. 待決議事項
- [x] ~~Refresh Token 與撤銷機制（目前只發 Access Token，過期後需重新登入）~~——**已解決**：見 §5.1（Access + Refresh 雙 Token、輪替機制、`RefreshToken` 實體）
- [ ] **需要業主決策（非技術判斷，已補上排序建議）**：LINE / Google OAuth 實際串接時程——這是排程/資源分配問題（何時安排工程資源去做這個已經保留好介面、只差實際串接的功能），不是規格能回答的技術決策，具體時程仍待業主排定。**排序建議**：Email + 密碼登入這條**必要路徑**已完整可用（§5.1，`ecommerce-services`/`ecommerce-storefront` 皆已真實作並通過瀏覽器 E2E），LINE/Google 屬於「加值的第二種登入方式」而非阻擋性缺口——目前買家/賣家都能正常完成帳號相關操作，不會因為沒有第三方登入而卡住。建議排在 §2 ShyeCMS 那批「需要業主決策」的商業條款議題（濫用舉證、GMV 計費、資料保留政策）之後、且更接近實際客戶要正式上線前再排入，不需要搶在目前這批議題前面處理
- [x] ~~§5.1 的登入/Refresh Token/密碼重設機制文字上聚焦「消費者會員」，但機制本身是 Identity Service 對所有 `Role` 共用~~——**已解決**：§5.1 開頭新增「適用範圍澄清」段落，明確列出僅 `register` 為消費者限定、其餘 7 個端點對所有角色共用
- [x] ~~訪客升級為會員時，歷史訂單「自動關聯」是否要等信箱驗證通過才生效~~——**已解決**（使用者 2026-09-09 確認：要等驗證通過）：見 §5.1「訪客升級為會員」，沿用同一 `User.Id`（訂單本來就指向它，不需搬移），密碼暫存於 `AccountActionToken.PendingPasswordHash`，驗證通過那一刻才寫入 `User.PasswordHash`、帳號才能登入
