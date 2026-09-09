# 30 - 待決議事項總表 (Open Decisions Register)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「待決議事項」需求：彙整全部 29 份文件累積的 77 項待決議，並依風險/急迫性挑出前 10 項優先處理 |
| v0.2 | 2026-09-08 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) 第七輪跨文件複查發現 §1、§6 引用的「[13-service-wms.md](13-service-wms.md) §6 Saga 補償失敗」項目實際上源文件從未寫過；已回頭補上該文件的待決議項（見其 v0.2 異動紀錄），本次同步新增 §4 對應列，統計數字改為 78 項 |
| v0.3 | 2026-09-08 | ordinarycas | §4 移除「Order / Correlation ID 貫穿追蹤機制尚未設計」列——[17-service-order.md](17-service-order.md) §6 已標記解決（見其 v0.3），此為第八輪複查後的同步；[28-i18n.md](28-i18n.md) 章節編號調整（原 §9 待決議事項改為 §8），§1、§5 的引用同步更新；統計數字改為 77 項 |
| v0.4 | 2026-09-09 | ordinarycas | 同步 [25-service-gateway.md](25-service-gateway.md) v0.3：反向代理引擎選型（YARP vs 自建）於 2026-09-09 定案採用 YARP——該項先前未曾列入任何待決議清單，本次於來源文件補列並即標記已解決，故 §4 不新增列，僅更新統計為 77 項未解決 + 3 項已解決；前 10 名清單不受影響 |
| v0.5 | 2026-09-09 | ordinarycas | repo 更名 `ecommerce-deploy`→`ecommerce-launch`；同步本輪 5 項新解決事項並移出對應列——ShyeCMS 前端規格空白（[31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)）、共用套件資安修補傳播機制（[29](29-shared-service-conventions.md) §4.1）、Saga 補償失敗處理（[17](17-service-order.md) §4.1，§4 移除 WMS/Promotions/Order 三列、§6 移除對應合併決策列）、熱銷排行/付款分布圖表選型（[22](22-service-analytics.md) §4）、英日文海外客群範圍（[28-i18n.md](28-i18n.md) §7/§8）；§1 前 10 名因此縮減為前 5 名（原第 6–10 名依序遞補，未額外補新項目）；統計更新為 71 項未解決 + 8 項已解決（修正：先前記為 70 項是手動心算漏算 §5 一行，v0.6 已用腳本逐行核對訂正） |
| v0.6 | 2026-09-09 | ordinarycas | 同步 [11-service-identity.md](11-service-identity.md) v0.3：Refresh Token 與撤銷機制已設計（§5.1），§4 移除對應列；改用腳本逐行核對 §2–§5 列數（而非手動心算），統計訂正為 70 項未解決 + 9 項已解決 |
| v0.7 | 2026-09-09 | ordinarycas | 同步 [10-gap-analysis.md](10-gap-analysis.md) §14 第九輪複查（新增消費者會員登入後的連鎖影響）：§3 新增訪客升級為會員機制列、§4 新增 Identity §5.1 適用範圍列與 Order Payment 失敗分支列，共 3 項新增（Gateway 路由不一致屬純格式錯誤已直接修正，不列入待決議）；§1 新增第 1 名（帳號安全新發現），改為前 6 名；統計更新為 73 項未解決 + 9 項已解決 |
| v0.8 | 2026-09-09 | ordinarycas | 使用者確認「訪客升級會員的自動關聯要等信箱驗證通過」：§3 移除對應列，§1 移除第 1 名、還原為前 5 名；統計改用近期解決清單條列，訂正為 72 項未解決 + 10 項已解決 |
| v0.9 | 2026-09-09 | ordinarycas | 同步 [17-service-order.md](17-service-order.md) v0.6、[07-storefront-requirements.md](07-storefront-requirements.md) v0.8：§4 原「Order / 逾時未付款自動取消、訂單編號策略需另訂」1 列拆成 2 列（訂單編號現況已補上文件，見源文件 §2.1）；首頁 3D 互動效果與 Identity Refresh Token 儲存方式的文件補寫不影響本表（皆非既有待決議項）；改用腳本逐行核對 §2–§5 列數，統計訂正為 73 項未解決 + 10 項已解決 |
| v0.10 | 2026-09-09 | ordinarycas | 使用者要求「將 73 項未解決列出來實作」，開始逐項處理：本輪解決 4 項並移出對應列——Saga 循序圖 Payment 失敗分支（[17](17-service-order.md)、[06](06-ecommerce-platform-architecture.md)）、Identity §5.1 適用範圍澄清（[11](11-service-identity.md)）、PostgreSQL 共用 instance 定案（[06](06-ecommerce-platform-architecture.md)）、Dockerfile 已撰寫（[26](26-project-structure.md)）；改用腳本逐行核對，統計訂正為 69 項未解決 + 14 項已解決 |
| v0.11 | 2026-09-10 | ordinarycas | 繼續逐項處理待決議事項：本輪解決 11 項——`StoreSettings` 歸屬定案 Vendor Service（連帶解決 §6 重複追蹤列）、Catalog 搜尋效能（pg_trgm，已於 `ecommerce-services` 實作+實測）、Gateway 4 項（速率限制/Webhook/公開建單/聚合文件，連帶解決 §6 重複追蹤列與 [09](09-api-specification.md) 的重複項）、優惠券不可疊加、`Coupon.Code` 賣家範圍唯一（皆核對既有實作定案）、評價審核機制、訪客購物車清理排程；§1 優先清單因 `StoreSettings` 解決縮減為前 4 名；統計訂正為 57 項未解決 + 25 項已解決 |
| v0.12 | 2026-09-10 | ordinarycas | 處理 §2 ShyeCMS 剩餘 7 項：3 項可技術/流程判斷已解決（`PastDue` SOP、開通部署現階段人工、`Suspended` 訊息文案），4 項屬合約/商業模式判斷（濫用舉證、GMV 計費相關 2 項、終止合作資料政策），不強行代為決定，改為標記「需要業主決策」並補上具體待答子問題，讓開放狀態本身更可執行；統計訂正為 54 項未解決 + 28 項已解決 |
| v0.13 | 2026-09-10 | ordinarycas | 修正 v0.11 遺漏——Catalog 商品搜尋效能已當輪解決但忘記移出對應列，本輪補上移除；處理 §3/§4 剩餘項目共 19 列：17 項可技術/設計判斷已解決（部分為「設計已補齊、實際串接/實作仍待沙箱環境或未來排入」的誠實記錄，非完全落地——WMS 3 項、Vendor 多賣家管理員角色、Order 逾時付款、Payment 退款/對帳設計、Media 影片縮圖設計/CDN、CMS Page Builder、Shipping 溫控物流、Analytics 查詢效能/匯出設計、Notification 3 項），Payment 沙箱實測與 Shipping 超商門市選擇標記「需要外部資源」維持開放，LINE/Google OAuth 標記「需要業主決策」並去重（§4 Identity 列併入 §3 對應列，純粹合併非解決）；改用腳本逐行核對，統計訂正為 34 項未解決 + 45 項已解決（已解決數為累計手動追蹤，逐項列名核對而非心算，避免重蹈 v0.5 的計數錯誤） |
| v0.14 | 2026-09-10 | ordinarycas | §3 收尾：解決 9 項（DB 連線安全性、Supabase 連線數改走 pooler、CI/CD 建置機器、訪客結帳簡訊防詐、PlatformSupportStaff 診斷端點盤點——順手在 Promotions/Notification 補上原本遺漏的診斷端點、即時通知、會員資訊遮罩、Grouped 商品 WooCommerce 語意——過程中發現並訂正 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) 一處實際的分隔符號錯誤、匯出連結時效）；統計說明改為明確點出剩餘項目裡多少是「需要業主決策/外部資源」而非規格能單方面解決；統計訂正為 25 項未解決 + 54 項已解決 |
| v0.15 | 2026-09-10 | ordinarycas | §5 跨服務/基礎設施本輪**全數解決並清空**（16 項：私有套件選型、CI/CD SOP、斷點/對比 Token、賣家後台 WCAG AA、Lighthouse CI、PWA 快取、ShyeCMS PWA、i18n 4 項、2FA、log 集中收集、CSP 盤點、服務 JWT 快取）；過程中比對 [10-gap-analysis.md](10-gap-analysis.md) 發現該文件多處嚴重過時（如聲稱 `docker-compose.yml` 骨架未撰寫，實際上早已是本輪一路在用的真實可跑版本），已一併完整重新核對該文件 §1–§4/§6/§8/§9/§14 共 23 項並同步；追加解決原本卡在 §1 第 1 名的「單一 VPS 備份/災難復原策略」——重新評估後發現先前歸類為「需要業主決策」過於保守，實際有明確技術方案（見 [06](06-ecommerce-platform-architecture.md) §6.5），§1 因此從前 2 名再縮減為**前 1 名**；統計訂正為 9 項未解決 + 70 項已解決，且明確標註這 9 項全部卡在業主決策/外部資源/刻意留白，沒有一項是規格工作本身還做得動的 |

## 使用說明

**這份文件是索引，不是唯一真相來源**：每個項目的完整脈絡（為什麼會有這個問題、牽涉哪些既有決策）留在原文件裡，這裡只列一句話摘要 + 連結。修改某個待決議事項時，**改原文件**，不要只改這裡——這份索引之後需要重新掃描各文件同步更新，否則會變成第二份需要維護的清單，反而增加混亂。這正是 [10-gap-analysis.md](10-gap-analysis.md) 每一輪都要重新核對既有項目是否已解決、編號是否衝突的同一個教訓：分析/索引文件要跟實際規格狀態定期核對，不能只靠人工記憶。

統計：**9 項未解決** + 70 項已解決（未解決數逐行核對 §2–§5 表格列數所得，非估算；已解決數為累計手動追蹤，逐項列名核對而非心算，已解決項目不列入本表）。**§5 跨服務/基礎設施本輪全數解決，剩餘 9 項全部集中在 §2–§4**：§2 ShyeCMS 4 項、§3 2 項、§4 2 項皆標記「需要業主決策」或「需要外部資源」，§4 剩下的 Order 訂單編號 1 項是刻意維持開放（現況已有文件記載的暫定方案，只是還沒正式升級為永久設計，非被卡住）——換句話說，**這 9 項沒有一項是規格/設計工作本身還做得動的**，全部卡在業主決策、外部資源，或本來就刻意留白。本輪（§5 收尾）解決 16 項：私有套件選型（GitHub Packages）、CI/CD SOP 與 `ecommerce-launch` 版本標籤流程、斷點寬度/色彩對比 Token、賣家後台 WCAG AA、Lighthouse/axe-core、PWA 快取策略、ShyeCMS PWA、i18n 4 項（翻譯提供方式、SEO 優先度、LINE 多語系、繁中強制填寫）、2FA（TOTP）、log 集中收集（Grafana Loki）、CSP 規則盤點、服務 JWT 快取策略。過程中同步發現並訂正 [10-gap-analysis.md](10-gap-analysis.md) 多處與實際規格/程式碼脫節的舊記錄（該文件 v0.20 有完整說明），一併定案了原本標記在 §1 前 1 名的備份/災難復原策略（[06](06-ecommerce-platform-architecture.md) §6.5）——先前把它歸類為「需要業主決策」是判斷過於保守，實際上有明確的技術方案可以解決，見 §1 上方說明。近期解決（前一輪 9 項）：DB 連線安全性/Supabase 連線數/CI/CD 建置機器、訪客結帳簡訊防詐、PlatformSupportStaff 診斷端點盤點、即時通知、會員資訊遮罩、Grouped 商品 WooCommerce 語意（發現並訂正分隔符號錯誤）、匯出連結時效。

## 1. 前 1 項建議優先處理（依風險/急迫性排序，非文件順序）

> 這份清單原本有 10 名，逐輪解決後陸續縮減。2026-09-10 最後兩輪：原第 1 名「單一 VPS 備份/災難復原策略」其實**可以**由技術方案解決（排程備份容器＋異地存放＋還原演練，非業主決策獨佔的問題，先前判斷過於保守），已定案見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §6.5，移出本表；原第 2 名「六個 repo 的 CI/CD SOP」與原第 4 名「高權限帳號 2FA」也已定案解決。下表只剩 1 項，且是**規格文件真的無法單方面解決**的項目（需要金流商核發的測試環境憑證，不存在「重新想一個技術方案」這條路）。

| # | 項目 | 為什麼優先 | 來源 |
|---|---|---|---|
| 1 | 三家金流廠商的沙箱實測，**需要外部資源**（廠商測試環境憑證） | 上線前必做，且是規格無法解決的項目——本表現在唯一的殘留項 | [18](18-service-payment.md) §8 |

## 2. ShyeCMS（[00](00-overview.md)–[05](05-scope-and-open-items.md)）

| 項目 | 來源 | 備註 |
|---|---|---|
| 濫用/超用糾紛的舉證機制（合約條款？人工稽核頻率？） | [01-architecture.md](01-architecture.md) §5 | **需要業主決策**，非技術判斷 |
| GMV 計費若要恢復可行性，需要客戶方同意的資料申報機制設計 | [01-architecture.md](01-architecture.md) §5 | **需要業主決策**，與下一項互相依賴 |
| GMV 超額抽成的計算依據完全空白 | [02-data-model.md](02-data-model.md) §6 | **需要業主決策**，具體抽成比例/計費模式 |
| 客戶終止合作的資料交還/刪除政策 | [03-client-lifecycle.md](03-client-lifecycle.md) §7 | **需要業主決策**，個資法/合約範疇；已補上 4 個具體待答子問題 |

## 3. 電商平台總覽與前後台需求（[06](06-ecommerce-platform-architecture.md)–[09](09-api-specification.md)）

| 項目 | 來源 |
|---|---|
| 服務數量在單一 VPS 部署下的資源消耗，**需要實測**（非規格能解決） | [06](06-ecommerce-platform-architecture.md) §10 |
| LINE / Google OAuth 實際串接時程，**需要業主決策**（排程/資源分配） | [07](07-storefront-requirements.md) §5、[11](11-service-identity.md) §6 |

## 4. 15 個微服務（[11](11-service-identity.md)–[25](25-service-gateway.md)）

| 服務 | 項目 |
|---|---|
| Order | 訂單編號產生規則是否正式定案（現況已補上文件，見 [17](17-service-order.md) §2.1、§6） |
| Payment | 三家廠商沙箱實測，**需要外部資源**（金流商測試環境憑證）（[18](18-service-payment.md) §8） |
| Shipping | 超商取貨門市選擇未串接，**需要外部資源**（超商官方地圖 API 合作資格）（[21](21-service-shipping.md) §5） |

## 5. 跨服務/基礎設施（[26](26-project-structure.md)–[29](29-shared-service-conventions.md)）

**2026-09-10 本節原本 16 項已全數解決**（私有套件選型、CI/CD SOP、斷點/對比 Token、賣家後台 WCAG AA、Lighthouse CI、PWA 快取策略、ShyeCMS PWA、i18n 4 項、2FA、log 集中收集、CSP 盤點、服務 JWT 快取），詳見 [26-project-structure.md](26-project-structure.md) §7、[27-pwa-and-accessibility.md](27-pwa-and-accessibility.md) §4、[28-i18n.md](28-i18n.md) §8、[29-shared-service-conventions.md](29-shared-service-conventions.md) §5 各自的異動紀錄，本表無殘留項目。

## 6. 重複出現超過一次的項目（值得合併決策，而非分開處理）

| 主題 | 出現位置 |
|---|---|
| LINE / Google OAuth 串接時程 | [07](07-storefront-requirements.md)、[11](11-service-identity.md) |

**建議**：與其分散在多份文件各自等待決定，不如安排一次性的決策會議一次解決。（原「多賣家入駐平台管理員角色」已於 2026-09-10 定案延後、原「Saga 補償失敗處理」已於 2026-09-09 統一解決，見 [17-service-order.md](17-service-order.md) §4.1，皆已移出本表；`16-service-promotions.md` 從未實際提過這個主題，是本表過去的引用誤植，一併訂正。）
