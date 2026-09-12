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
| v0.10 | 2026-09-10 | ordinarycas | §5 訪客結帳防詐機制待決議項已解決：現階段不強制簡訊驗證，採分層防詐（速率限制+Email 驗證），回應「將待決議事項列出來實作」需求 |
| v0.11 | 2026-09-12 | ordinarycas | §3.1 補上「首頁尚未接上 CMS Service」缺口的具體交叉引用——先前只在本節提到「屬於既有已知缺口」但沒有指向任何集中追蹤處，[10-gap-analysis.md](10-gap-analysis.md) 新增 §15 集中盤點這個缺口與相關斷點（`AboutUs`/`Custom` 版型連前台路由都不存在、CMS 發佈後通知前台重新產生的機制實際上沒有可失效的快取對象），回應「集中追蹤首頁 CMS 串接斷開處」需求 |
| v0.12 | 2026-09-12 | ordinarycas | §3.1 整節重寫：`ecommerce-storefront` 第七輪把首頁互動 3D 芭樂從舊版雙態展示 hero（單一 `role="button"` 容器、`aria-pressed` 切換）升級為「結構標註圖」（`components/guava-callout-diagram.tsx`，舊檔 `guava-hero.tsx` 已併入移除），本節先前仍描述已不存在的舊元件行為，未反映現況；重寫為 4 個獨立熱點（各自 `aria-expanded`/`aria-controls`，查證來源的營養事實）、全程展開的文字化等價列表、WebGL 建立失敗時退回靜態 SVG 剖面圖的現況，回應本輪規格同步稽核發現的落後缺口 |
| v0.13 | 2026-09-12 | ordinarycas | 新增 §3.2：記錄 `ecommerce-storefront` 本輪同時新增、先前完全未寫入規格的揭示型導覽選單（`components/site-header.tsx`，WAI-ARIA disclosure navigation pattern），回應本輪規格同步稽核發現的落後缺口 |

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

### 3.1 首頁固定視覺效果：互動式 3D 芭樂「結構標註圖」（現況記錄，第七輪重寫）

`ecommerce-storefront` 首頁的互動式 3D 芭樂模型（呼應賣家品牌「爸芭樂」與熱銷商品「紅心芭樂」）第七輪（2026-09-12）由雙態展示 hero 升級為「結構標註圖」——沿用同一套 three.js 幾何/剖面手法，疊加一層熱點標註，讓使用者互動了解各部位的營養事實，不只是「切開看好玩」的展示效果。**本節先前描述的是已刪除的舊元件行為（單一 `role="button"` 容器、切開狀態以 `aria-pressed` 表達），未反映現況**，以下整節依現況重寫：

| 項目 | 說明 |
|---|---|
| 元件 | `components/guava-callout-diagram.tsx` + `guava-callout-diagram-loader.tsx`（`next/dynamic({ ssr:false })` 包裝層，因 Server Component 頁面不能直接寫 `ssr:false`）。前身 `guava-hero.tsx`（LatheGeometry 車身、拖曳旋轉＋慣性＋閒置自轉、點擊切換整顆/剖半的雙態展示 hero）已併入本檔並移除，避免首頁維護兩顆平行的 3D 芭樂實作 |
| 呈現方式 | 純 three.js 程序化建模（`LatheGeometry` 車出果身輪廓＋果蒂＋葉片），**不載入任何外部 3D 模型或貼圖檔**——不增加 SSG 產物體積，也沒有跨網域資源請求 |
| 互動：旋轉 | 拖曳（滑鼠／觸控皆可）旋轉；放開後保留慣性、逐漸衰減；閒置一段時間後緩慢自轉 |
| 互動：整顆↔剖半 | 獨立的 `<button aria-pressed>` 觸發（**不再是容器本身兼任這顆按鈕**——這是與舊版最主要的行為差異），剖面露出果肉與籽（呼應「紅心芭樂」商品名稱） |
| 互動：熱點標註（本輪新增） | **4 個查證過的部位**：整顆總覽／外皮（整顆狀態下可見）、果肉／籽（剖半狀態下可見）——刻意只做這 4 個有查證數字的部位，沒有替「果蒂與葉片」「中心果核腔」等無查證數字的部位杜撰營養事實。每個熱點是掛在對應 3D 錨點（`THREE.Object3D` anchor）、在 `requestAnimationFrame` 迴圈裡手動投影成 2D 螢幕座標的獨立 `<button data-hotspot-button aria-expanded aria-controls aria-label>`，可 Tab 到，點擊或 Enter 展開/收合對應的查證營養事實與資料來源（來源含 USDA FoodData Central、期刊論文，文字列表中附可點連結；展開時同步把下方文字化等價列表對應的項目捲入可視範圍） |
| 熱點的無障礙細節 | 非目前狀態（整顆/剖半）的熱點直接不 render（React 依 `cut` 狀態決定要掛載哪 2 個熱點），不會讓鍵盤使用者 Tab 到畫面上看不到的按鈕；背面（因拖曳旋轉而朝向鏡頭另一側、正面判定用法向量與視角的內積）的熱點則以 `hidden` + `tabIndex=-1` 隱藏（效能考量，避免每幀觸發 React re-render） |
| 文字化等價列表 | 頁面同時提供一份**全程展開、全程在 DOM 裡**的 `<dl>`，內容與 3D 熱點共用同一份字典——不需要先跟 3D 互動，就能取得全部 4 個部位的完整營養說明 |
| WebGL 不可用時的退回 | WebGL 建立失敗，或偵測到低階裝置訊號（`navigator.hardwareConcurrency <= 1`）時，直接退回**靜態 SVG 剖面圖**＋同一份文字列表，配色比照 three.js 材質色票，不嘗試硬撐一個半殘的 3D 版本 |
| `prefers-reduced-motion` | 關閉閒置自轉與整顆/剖半切換的彈跳動畫；使用者主動觸發的拖曳旋轉、切開/合起、熱點展開不受影響 |
| 文案與翻譯範圍 | 提示文案、按鈕與熱點的 `aria-label` 走三語字典（[28-i18n.md](28-i18n.md) §5 UI 文字），三語皆完整；**4 個熱點查證過的營養事實/來源內容本身刻意只在 zh-Hant 定義**，en/ja 依既有 fallback 規則沿用同一份中文內容（比照既有「事實類」內容的慣例），屬刻意留待日後另行查證/專業翻譯的範疇，不是本節新增的缺口 |

**與 CMS Service 的關係**：這是首頁模板裡**寫死**的固定視覺效果，不透過 CMS Service 的 `PageSection`/`Config` 機制管理，賣家後台無法關閉或替換它——與同一頁面上其餘走 CMS 管理的內容區塊（Banner／RichText 等）是不同性質的東西，屬於「爸芭樂」這個範例案例本身的品牌呈現，比照白牌客戶各自客製首頁模板的預期（見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)），不是每個白牌客戶都會有的通用平台功能。首頁其餘文字內容目前仍是佔位文案、尚未實際接上 CMS Service 查詢，這點屬於既有已知缺口（不是本節新增的設計）——完整盤點見 [10-gap-analysis.md](10-gap-analysis.md) §15：`AboutUs`/`Custom` 版型連前台路由都不存在（同一個根本缺口的延伸，不是各自獨立遺漏）、CMS 發佈後通知前台重新產生的機制實際上沒有可失效的快取對象、賣家後台的版型編輯器是完整可用的真實功能因此容易讓操作者誤判「發佈後買家端已生效」。

### 3.2 全站導覽：揭示型選單（Disclosure Navigation，本輪新增）

`ecommerce-storefront` 全站共用的 `<header>`（`components/site-header.tsx`，隱含 `banner` landmark）本輪把主要導覽（首頁/商品/購物車/訂單）從先前一排常駐的小連結，改為 **WAI-ARIA Authoring Practices 的揭示型選單（disclosure navigation pattern）**：一顆觸發按鈕 + 一片緊貼 header 下緣、展開時淡入並些微位移的面板。先前完全未寫入規格，本節為本輪新增：

| 項目 | 說明 |
|---|---|
| 觸發按鈕 | `<button aria-expanded aria-controls>`，`aria-expanded` 反映面板目前是否展開，`aria-controls` 指向面板的 `id`；按鈕文案在展開/收合兩態各自對應「選單」/「關閉選單」，三語字典化 |
| 面板內容 | 語意完整的 `<nav aria-label="…"><ul><a>`，**不套用 `role="menu"`/`menuitem`**——那是給選單列/應用程式選單用的角色，套在一般導覽連結上會讓螢幕報讀軟體的操作方式跟使用者預期不符，是 ARIA APG 明確不建議的用法，本元件刻意避免 |
| 收合時的隱藏機制 | 面板收合時套用 `inert` 屬性（而非 `hidden`）——收合時鍵盤/螢幕報讀皆摸不到、Tab 不會跳進去，行為等同 `hidden`，但因為只是 `inert` 而非 `display:none`，收合/展開之間的淡入＋些微位移過渡（CSS `opacity`/`transform`）才有起點可以動畫；`hidden` 屬性做不到這點（`display:none` 無法 transition） |
| Escape 收合 | 面板展開時按 `Escape`：收合面板並把焦點還給觸發按鈕，避免焦點遺失 |
| 點擊面板外收合 | 面板展開時，點擊面板與觸發按鈕以外的任何地方（`mousedown` 監聽）：收合面板 |
| 換頁後的狀態重置 | 換頁後面板不應該還留著上一頁展開的狀態——`pathname` 改變時在 render 期間直接重設 `isNavOpen`，不透過 `useEffect`（避免多一次 cascading render） |
| `prefers-reduced-motion` | 全站共用規則已把 `transition-duration` 壓到近乎 0——使用者開啟「減少動態效果」時，面板展開/收合會直接「瞬間」切換，功能（展開/收合/可操作性）不受影響 |

**適用範圍**：這是全站共用 `<header>` 元件，不是首頁專屬——套用在前台每一個頁面，與 §3.1 首頁固定視覺效果（僅首頁）是不同範疇的東西。

## 4. 商品詳情頁的現貨顯示

商品詳情頁需要即時反映庫存狀態（生鮮商品缺貨/售完的顯示尤其重要），但商品資訊本身走靜態生成（SSG/ISR）。做法：
- 頁面主體（描述、圖片、價格）走 SSG/ISR，SEO 內容完整。
- 現貨/庫存狀態透過前端呼叫 WMS Service 的**唯讀查詢端點**，於頁面載入後以 CSR 方式覆蓋顯示（類似「載入中→現貨 12 件／已售完」），避免庫存數字被快取在靜態頁面裡而失真。

## 5. 待決議事項
- [ ] **需要業主決策（非技術判斷）**：LINE / Google OAuth 實際串接時程（沿用 v1 既有缺口）——排程/資源分配問題，非規格能回答的技術決策，維持開放，見 [11-service-identity.md](11-service-identity.md) §6 同項
- [x] ~~訪客結帳是否需要簡訊驗證等防詐機制（生鮮商品退貨成本高，惡意下單風險需評估）~~——**已解決：現階段不強制簡訊驗證，採分層防詐**。理由：簡訊驗證需要簡訊閘道商合約與費用（見 [23-service-notification.md](23-service-notification.md) §7），且會在結帳流程多加一個步驟，對「免登入即可下單」這個平台核心原則（§1）造成摩擦，在還沒觀察到實際濫用情況前就加上去，屬於為假設風險預先犧牲轉換率。現階段已有的防詐層：(1) [29-shared-service-conventions.md](29-shared-service-conventions.md) §4 既有的匿名端點速率限制（依 IP 位址），本來就會限制短時間內大量結帳嘗試；(2) Email 驗證機制（[11-service-identity.md](11-service-identity.md) §5.1）本身已提供一定程度的真實性門檻。**若之後真的觀察到惡意下單問題**（COD 空單、假訂單佔用庫存等），再評估加簡訊驗證，屆時建議只針對高風險情境觸發（如同一 IP/Email 短時間內多筆訂單、COD 高單價訂單），而非全面強制，維持免登入下單的核心體驗
- [x] ~~訪客升級為會員的確切機制未定義~~——**已解決**（使用者 2026-09-09 確認：要等信箱驗證通過）：§1 已更新，機制細節見 [11-service-identity.md](11-service-identity.md) §5.1

## 6. RWD / PWA / 無障礙規範

前台須支援響應式設計（手機/平板/桌面）並做成可安裝的 PWA，且須符合 **WCAG 2.1 AA**（台灣網站無障礙規範 110.07 版基準）。完整規格見 [27-pwa-and-accessibility.md](27-pwa-and-accessibility.md)，本節不重複列出。
