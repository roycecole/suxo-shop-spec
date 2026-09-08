# 07 - 前台需求 (Storefront Requirements)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分出前台細節需求，回應「免登入下單、保留 LINE/Google 登入」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §6：RWD/PWA 與無障礙規範要求，指向 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md) |

> 本文件是 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5、§8 的細節展開，針對「爸芭樂」微服務平台具體化。前身規格曾有更完整的前台需求（搜尋篩選、商品評價、收藏追蹤等），已隨舊版規格一併移除，見 [00-overview.md](00-overview.md) §8。

## 1. 核心原則：免登入即可下單

**買家不需要註冊/登入就能瀏覽商品、加入購物車、完成結帳下單。** 這是硬性需求，不是可選功能。

| 情境 | 行為 |
|---|---|
| 訪客瀏覽商品、加入購物車 | 購物車以 Cookie/SessionId 識別，存於 Cart Service |
| 訪客結帳 | 僅需填寫收件資訊（姓名/電話/地址）與 Email，不強制建立帳號；對應 Identity Service 建立一筆無密碼的訪客購買紀錄 |
| 訪客查詢訂單 | 提供公開端點：訂單編號 + Email 查詢，避免探測攻擊需搭配速率限制 |
| 訪客升級為會員 | 訪客結帳後可用同 Email 註冊，系統將歷史訂單自動關聯到新帳號 |

## 2. 登入方式（可選，非必要）

登入不是下單的前提，而是**加值功能**（如查看訂單歷史更方便、收藏商品、會員專屬優惠）：

| 方式 | 狀態 |
|---|---|
| Email + 密碼 | 必要，Identity Service 基本功能 |
| LINE 登入 | **保留**：資料結構（`ExternalLogin`）與前端按鈕需備妥，OAuth 流程本輪不重新設計 |
| Google 登入 | **保留**：同上 |

> 「保留」的意義：介面已備妥，實際 OAuth 串接未完成，呼叫會回應明確的「尚未開放」狀態，不是靜默失敗。

## 3. 頁面清單與對應服務

| 頁面 | 對應服務 | 生成方式（見 [06](06-ecommerce-platform-architecture.md) §5） |
|---|---|---|
| 品牌形象首頁 | CMS Service | SSG |
| 商品清單/分類頁 | Catalog Service | ISR |
| 商品詳情頁 | Catalog Service（商品資訊）+ WMS Service（是否有現貨） | SSG（熱銷商品）/ ISR（其餘） |
| 購物車 | Cart Service | CSR |
| 結帳頁 | Order Service（建立訂單）、Payment Service（導轉金流）、Shipping Service（運費試算） | CSR |
| 訂單查詢頁（會員） | Order Service（依登入身分查詢） | CSR |
| 訂單查詢頁（訪客） | Order Service（訂單編號 + Email） | CSR |

## 4. 商品詳情頁的現貨顯示

商品詳情頁需要即時反映庫存狀態（生鮮商品缺貨/售完的顯示尤其重要），但商品資訊本身走靜態生成（SSG/ISR）。做法：
- 頁面主體（描述、圖片、價格）走 SSG/ISR，SEO 內容完整。
- 現貨/庫存狀態透過前端呼叫 WMS Service 的**唯讀查詢端點**，於頁面載入後以 CSR 方式覆蓋顯示（類似「載入中→現貨 12 件／已售完」），避免庫存數字被快取在靜態頁面裡而失真。

## 5. 待決議事項
- [ ] LINE / Google OAuth 實際串接時程（沿用 v1 既有缺口，未在本輪排入）
- [ ] 訪客結帳是否需要簡訊驗證等防詐機制（生鮮商品退貨成本高，惡意下單風險需評估）

## 6. RWD / PWA / 無障礙規範

前台須支援響應式設計（手機/平板/桌面）並做成可安裝的 PWA，且須符合 **WCAG 2.1 AA**（台灣網站無障礙規範 110.07 版基準）。完整規格見 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md)，本節不重複列出。
