# 07 - 前台需求 (Storefront Requirements)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分出前台細節需求，回應「免登入下單、保留 LINE/Google 登入」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §6：RWD/PWA 與無障礙規範要求，指向 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md) |
| v0.3 | 2026-09-08 | ordinarycas | §1 補充說明：本節「硬性需求」與 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) 的賣家自訂 `StoreSettings` 開關的界線，解決前一輪複查發現兩者矛盾（08 已移除對應開關，見其 v0.5） |
| v0.4 | 2026-09-08 | ordinarycas | §3 頁面清單補上結帳頁對應 Promotions Service、商品詳情頁對應 Reviews Service，解決 [10-gap-analysis.md](10-gap-analysis.md) §12 已列的漏列問題 |
| v0.5 | 2026-09-09 | ordinarycas | 回應「新增消費者會員登入，先保留 Google、Line 登入」需求：§3 補上登入/註冊、忘記密碼、會員中心三個頁面對應；§2 Email+密碼列補上 [11-service-identity.md](11-service-identity.md) §5.1 詳細流程的交叉引用。LINE/Google 維持既有保留狀態不變，未異動 |
| v0.6 | 2026-09-09 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) §14 第九輪複查發現：§1「訪客升級為會員」機制與 [11-service-identity.md](11-service-identity.md) §5.1 新增的信箱驗證流程存在帳號冒領風險，本文件本身不解決該問題（涉及 Identity Service 設計，非前台需求範疇），僅於 §5 新增對應待決議項記錄 |
| v0.7 | 2026-09-09 | ordinarycas | 使用者確認「訪客升級會員的自動關聯要等信箱驗證通過」：§1 更新訪客升級為會員的說明（沿用同一 `User.Id`，需驗證才能登入），§5 對應待決議項標記已解決，機制細節見 [11-service-identity.md](11-service-identity.md) §5.1 |
| v0.8 | 2026-09-09 | ordinarycas | §3 首頁列新增 §3.1 交叉引用；新增 §3.1 記錄 `ecommerce-storefront` 已實作的首頁互動 3D 芭樂效果（原本只活在程式碼與元件註解裡），回應「把已經做出來但規格沒寫的東西補回文件」需求 |
| v0.9 | 2026-09-10 | ordinarycas | §5 LINE/Google OAuth 串接時程標記為需要業主決策，回應「將待決議事項列出來實作」需求 |

> 本文件是 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5、§8 的細節展開，針對「爸芭樂」微服務平台具體化。前身規格曾有更完整的前台需求（搜尋篩選、商品評價、收藏追蹤等），已隨舊版規格一併移除，見 [00-overview.md](00-overview.md) §8。

## 1. 核心原則：免登入即可下單

**買家不需要註冊/登入就能瀏覽商品、加入購物車、完成結帳下單。** 這是**平台層級**的硬性需求，不是可選功能，也**不是**[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §2 那種賣家可在自己後台自行開關的營運選項——賣家能決定要不要開放貨到付款、要不要用優惠券模組，但不能關閉訪客結帳這個能力本身。

| 情境 | 行為 |
|---|---|
| 訪客瀏覽商品、加入購物車 | 購物車以 Cookie/SessionId 識別，存於 Cart Service |
| 訪客結帳 | 僅需填寫收件資訊（姓名/電話/地址）與 Email，不強制建立帳號；對應 Identity Service 建立一筆無密碼的訪客購買紀錄 |
| 訪客查詢訂單 | 提供公開端點：訂單編號 + Email 查詢，避免探測攻擊需搭配速率限制 |
| 訪客升級為會員 | 訪客結帳後可用同 Email 註冊即可升級——沿用同一個帳號（`User.Id` 不變，訂單本來就指向它），**需完成信箱驗證才能實際登入**，避免任何人光憑知道某個 Email 曾訪客結帳過就冒領其訂單歷史，機制細節見 [11-service-identity.md](11-service-identity.md) §5.1 |

## 2. 登入方式（可選，非必要）

登入不是下單的前提，而是**加值功能**（如查看訂單歷史更方便、收藏商品、會員專屬優惠）：

| 方式 | 狀態 |
|---|---|
| Email + 密碼 | 必要，Identity Service 基本功能——註冊/信箱驗證/忘記密碼/Refresh Token 完整流程見 [11-service-identity.md](11-service-identity.md) §5.1 |
| LINE 登入 | **保留**：資料結構（`ExternalLogin`）與前端按鈕需備妥，OAuth 流程本輪不重新設計 |
| Google 登入 | **保留**：同上 |

> 「保留」的意義：介面已備妥，實際 OAuth 串接未完成，呼叫會回應明確的「尚未開放」狀態，不是靜默失敗。

## 3. 頁面清單與對應服務

| 頁面 | 對應服務 | 生成方式（見 [06](06-ecommerce-platform-architecture.md) §5） |
|---|---|---|
| 品牌形象首頁（含互動 3D 芭樂，見 §3.1） | CMS Service | SSG |
| 商品清單/分類頁 | Catalog Service | ISR |
| 商品詳情頁 | Catalog Service（商品資訊）+ WMS Service（是否有現貨）+ Reviews Service（評價列表，受 `StoreSettings.ReviewsVisible` 控制） | SSG（熱銷商品）/ ISR（其餘） |
| 購物車 | Cart Service | CSR |
| 結帳頁 | Order Service（建立訂單）、Payment Service（導轉金流）、Shipping Service（運費試算）、Promotions Service（套用優惠券，見 [06](06-ecommerce-platform-architecture.md) §7 Saga） | CSR |
| 登入/註冊頁 | Identity Service（[11](11-service-identity.md) §5.1） | CSR |
| 忘記密碼/重設密碼頁 | Identity Service（[11](11-service-identity.md) §5.1） | CSR |
| 會員中心頁（個人資料、信箱驗證狀態、地址簿） | Identity Service | CSR |
| 訂單查詢頁（會員） | Order Service（依登入身分查詢） | CSR |
| 訂單查詢頁（訪客） | Order Service（訂單編號 + Email） | CSR |

### 3.1 首頁固定視覺效果：互動式 3D 芭樂（現況記錄）

`ecommerce-storefront` 首頁已實作一個互動式 3D 芭樂模型（呼應賣家品牌「爸芭樂」與熱銷商品「紅心芭樂」），先前只以元件內的說明註解存在，未回頭寫進本文件：

| 項目 | 說明 |
|---|---|
| 呈現方式 | 純 three.js 程序化建模（`LatheGeometry` 車出果身輪廓＋果蒂＋葉片），**不載入任何外部 3D 模型或貼圖檔**——不增加 SSG 產物體積，也沒有跨網域資源請求 |
| 互動：旋轉 | 拖曳（滑鼠／觸控皆可）旋轉；放開後保留慣性、逐漸衰減；閒置一段時間後緩慢自轉 |
| 互動：切開 | 點擊，或聚焦後按 Enter／空白鍵：整顆↔剖半切換，剖面露出果肉與籽（呼應「紅心芭樂」商品名稱） |
| 無障礙 | 容器為 `role="button"`、可鍵盤聚焦（`tabIndex`），切開狀態以 `aria-pressed` 表達，另有 `aria-label` 描述用途；提示文案與無障礙標籤走三語字典（[28-i18n.md](28-i18n.md) §5 UI 文字），不是 Translation 表的動態內容 |
| `prefers-reduced-motion` | 使用者開啟「減少動態效果」時，關閉閒置自轉與切換彈跳動畫，僅保留拖曳旋轉、切開/合起這類使用者直接觸發的操作 |

**與 CMS Service 的關係**：這是首頁模板裡**寫死**的固定視覺效果，不透過 CMS Service 的 `PageSection`/`Config` 機制管理，賣家後台無法關閉或替換它——與同一頁面上其餘走 CMS 管理的內容區塊（Banner／RichText 等）是不同性質的東西，屬於「爸芭樂」這個範例案例本身的品牌呈現，比照白牌客戶各自客製首頁模板的預期（見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)），不是每個白牌客戶都會有的通用平台功能。首頁其餘文字內容目前仍是佔位文案、尚未實際接上 CMS Service 查詢，這點屬於既有已知缺口，不是本節新增的設計。

## 4. 商品詳情頁的現貨顯示

商品詳情頁需要即時反映庫存狀態（生鮮商品缺貨/售完的顯示尤其重要），但商品資訊本身走靜態生成（SSG/ISR）。做法：
- 頁面主體（描述、圖片、價格）走 SSG/ISR，SEO 內容完整。
- 現貨/庫存狀態透過前端呼叫 WMS Service 的**唯讀查詢端點**，於頁面載入後以 CSR 方式覆蓋顯示（類似「載入中→現貨 12 件／已售完」），避免庫存數字被快取在靜態頁面裡而失真。

## 5. 待決議事項
- [ ] **需要業主決策（非技術判斷）**：LINE / Google OAuth 實際串接時程（沿用 v1 既有缺口）——排程/資源分配問題，非規格能回答的技術決策，維持開放，見 [11-service-identity.md](11-service-identity.md) §6 同項
- [ ] 訪客結帳是否需要簡訊驗證等防詐機制（生鮮商品退貨成本高，惡意下單風險需評估）
- [x] ~~訪客升級為會員的確切機制未定義~~——**已解決**（使用者 2026-09-09 確認：要等信箱驗證通過）：§1 已更新，機制細節見 [11-service-identity.md](11-service-identity.md) §5.1

## 6. RWD / PWA / 無障礙規範

前台須支援響應式設計（手機/平板/桌面）並做成可安裝的 PWA，且須符合 **WCAG 2.1 AA**（台灣網站無障礙規範 110.07 版基準）。完整規格見 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md)，本節不重複列出。
