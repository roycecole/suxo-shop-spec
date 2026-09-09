# 30 - 待決議事項總表 (Open Decisions Register)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「待決議事項」需求：彙整全部 29 份文件累積的 77 項待決議，並依風險/急迫性挑出前 10 項優先處理 |
| v0.2 | 2026-09-08 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) 第七輪跨文件複查發現 §1、§6 引用的「[13-service-wms.md](13-service-wms.md) §6 Saga 補償失敗」項目實際上源文件從未寫過；已回頭補上該文件的待決議項（見其 v0.2 異動紀錄），本次同步新增 §4 對應列，統計數字改為 78 項 |
| v0.3 | 2026-09-08 | ordinarycas | §4 移除「Order / Correlation ID 貫穿追蹤機制尚未設計」列——[17-service-order.md](17-service-order.md) §6 已標記解決（見其 v0.3），此為第八輪複查後的同步；[28-i18n.md](28-i18n.md) 章節編號調整（原 §9 待決議事項改為 §8），§1、§5 的引用同步更新；統計數字改為 77 項 |
| v0.4 | 2026-09-09 | ordinarycas | 同步 [25-service-gateway.md](25-service-gateway.md) v0.3：反向代理引擎選型（YARP vs 自建）於 2026-09-09 定案採用 YARP——該項先前未曾列入任何待決議清單，本次於來源文件補列並即標記已解決，故 §4 不新增列，僅更新統計為 77 項未解決 + 3 項已解決；前 10 名清單不受影響 |

## 使用說明

**這份文件是索引，不是唯一真相來源**：每個項目的完整脈絡（為什麼會有這個問題、牽涉哪些既有決策）留在原文件裡，這裡只列一句話摘要 + 連結。修改某個待決議事項時，**改原文件**，不要只改這裡——這份索引之後需要重新掃描各文件同步更新，否則會變成第二份需要維護的清單，反而增加混亂。這正是 [10-gap-analysis.md](10-gap-analysis.md) 每一輪都要重新核對既有項目是否已解決、編號是否衝突的同一個教訓：分析/索引文件要跟實際規格狀態定期核對，不能只靠人工記憶。

統計：**77 項未解決** + 3 項已解決（[12-service-catalog.md](12-service-catalog.md) 的 Markdown XSS 防護、[17-service-order.md](17-service-order.md) 的 Correlation ID 貫穿追蹤，皆已於 [29-shared-service-conventions.md](29-shared-service-conventions.md) 定案；[25-service-gateway.md](25-service-gateway.md) 的反向代理引擎選型已於 2026-09-09 定案採用 YARP，見其 §4.1——已解決項目不列入本表）。

## 1. 前 10 項建議優先處理（依風險/急迫性排序，非文件順序）

| # | 項目 | 為什麼優先 | 來源 |
|---|---|---|---|
| 1 | ShyeCMS 前端頁面/操作流程規格完全空白 | 唯一一個「連怎麼動工都答不出來」的缺口 | [26](26-project-structure.md) §7 |
| 2 | 共用函式庫（`SuxoShop.Shared.*`）的資安修補傳播機制缺失 | 版本化套件設計讓服務可以「不升級」，但資安修補不該適用這個彈性，直接關係到剛建立的資安基準是否真的有保護力 | [10-gap-analysis.md](10-gap-analysis.md) §9 |
| 3 | Saga 補償失敗的最終處理與告警機制 | 全新失敗模式，WMS/Promotions/Order 三個服務都會遇到，目前完全沒有設計 | [17](17-service-order.md) §6、[16](16-service-promotions.md) §5、[13](13-service-wms.md) §6 |
| 4 | 熱銷排行/付款分布沒有可用的圖表函式庫 | Lightweight Charts 明確排除這兩種類型，賣家後台兩個既有功能實際卡住無法動工 | [22](22-service-analytics.md) §6 |
| 5 | 英文/日文版是否要服務海外客群 | 答案會決定幣別計價與金流是否需要整個重新設計，是目前影響範圍最大的未知數 | [28-i18n.md](28-i18n.md) §8 |
| 6 | 單一 VPS 的備份/災難復原策略空白 | 架構決策把風險集中到一台主機，上線前必須有答案，不是能無限期擱置的項目 | [06](06-ecommerce-platform-architecture.md) §10 |
| 7 | 六個 repo 的 CI/CD 與跨 repo 版本協調 SOP | repo 拆分後最直接的維運後果，沒有這個 SOP，`ecommerce-deploy` 的版本標籤永遠只能手動猜 | [26](26-project-structure.md) §7 |
| 8 | `StoreSettings` 歸屬 Vendor 還是 CMS Service | 影響 API 開發分工，卡住實作排程的小決策 | [14](14-service-vendor.md) §4、[20](20-service-cms.md) §5 |
| 9 | 三家金流廠商的沙箱實測 | 上線前必做，且是規格無法解決的項目（需要廠商測試環境憑證） | [18](18-service-payment.md) §8 |
| 10 | 高權限帳號（SuperAdmin/PlatformSupportStaff）2FA | 這兩個角色能碰到所有客戶或所有訂單資料，權限範圍最大但目前驗證強度未提升 | [29](29-shared-service-conventions.md) §5 |

## 2. ShyeCMS（[00](00-overview.md)–[05](05-scope-and-open-items.md)）

| 項目 | 來源 |
|---|---|
| 濫用/超用糾紛的舉證機制（合約條款？人工稽核頻率？） | [01-architecture.md](01-architecture.md) §5 |
| GMV 計費若要恢復可行性，需要客戶方同意的資料申報機制設計 | [01-architecture.md](01-architecture.md) §5 |
| `ClientSubscription.Status = PastDue` 時的標準作業流程 | [02-data-model.md](02-data-model.md) §6 |
| GMV 超額抽成的計算依據完全空白 | [02-data-model.md](02-data-model.md) §6 |
| 開通部署（Provisioning）是否要自動化 | [03-client-lifecycle.md](03-client-lifecycle.md) §7 |
| `Suspended` 狀態下客戶站台該顯示什麼訊息給買家/賣家 | [03-client-lifecycle.md](03-client-lifecycle.md) §7 |
| 客戶終止合作的資料交還/刪除政策 | [03-client-lifecycle.md](03-client-lifecycle.md) §7 |

## 3. 電商平台總覽與前後台需求（[06](06-ecommerce-platform-architecture.md)–[09](09-api-specification.md)）

| 項目 | 來源 |
|---|---|
| 服務數量在單一 VPS 部署下的資源消耗未經實測校正 | [06](06-ecommerce-platform-architecture.md) §10 |
| PostgreSQL 共用 instance 多 schema，還是每服務獨立 instance | [06](06-ecommerce-platform-architecture.md) §10 |
| 外部/內部 DB 模式的網路延遲與安全性未規劃 | [06](06-ecommerce-platform-architecture.md) §10 |
| Supabase 免費/低階方案的連線數上限是否足夠 | [06](06-ecommerce-platform-architecture.md) §10 |
| CI/CD 建置機器的規格未定 | [06](06-ecommerce-platform-architecture.md) §10 |
| LINE / Google OAuth 實際串接時程 | [07](07-storefront-requirements.md) §5 |
| 訪客結帳是否需要簡訊驗證等防詐機制 | [07](07-storefront-requirements.md) §5 |
| `PlatformSupportStaff` 白名單診斷操作清單需逐服務盤點 | [08](08-vendor-admin-requirements.md) §6 |
| `PlatformSupportStaff` 存取是否需要即時通知客戶端 | [08](08-vendor-admin-requirements.md) §6 |
| 唯讀範圍是否需要對會員 Email/電話遮罩顯示 | [08](08-vendor-admin-requirements.md) §6 |
| 稅務欄位匯出固定值是否足夠 | [08](08-vendor-admin-requirements.md) §6 |
| Grouped 商品的 WooCommerce 語意核對 | [08](08-vendor-admin-requirements.md) §6 |
| 匯出檔案下載連結的時效與存取權限 | [08](08-vendor-admin-requirements.md) §6 |
| Gateway 聚合文件的實際呈現方式 | [09](09-api-specification.md) §4 |
| 內部 API 是否對外揭露文件 | [09](09-api-specification.md) §4 |

## 4. 15 個微服務（[11](11-service-identity.md)–[25](25-service-gateway.md)）

| 服務 | 項目 |
|---|---|
| Identity | Refresh Token 與撤銷機制未設計（[11](11-service-identity.md) §6） |
| Identity | LINE / Google OAuth 實際串接時程（[11](11-service-identity.md) §6） |
| Catalog | 商品搜尋效能（`LIKE` 無法用索引）（[12](12-service-catalog.md) §6） |
| Catalog | 稅務欄位是否正式納入 Product（[12](12-service-catalog.md) §6） |
| WMS | 多倉支援未區分（[13](13-service-wms.md) §6） |
| WMS | 效期商品自動下架/促銷無流程（[13](13-service-wms.md) §6） |
| WMS | Catalog 呼叫失敗時的前台降級行為未定義（[13](13-service-wms.md) §6） |
| WMS | Saga 補償失敗（`release` 本身失敗）的最終處理與告警機制（[13](13-service-wms.md) §6） |
| Vendor | `StoreSettings` 歸屬未定案（[14](14-service-vendor.md) §4） |
| Vendor | 多賣家入駐需要補平台管理員角色（[14](14-service-vendor.md) §4） |
| Cart | 訪客購物車自動清理排程未定（[15](15-service-cart.md) §5） |
| Promotions | 優惠券是否可疊加使用（[16](16-service-promotions.md) §5） |
| Promotions | Saga 補償失敗（revert 本身失敗）的處理方式（[16](16-service-promotions.md) §5） |
| Promotions | `Coupon.Code` 唯一性範圍（[16](16-service-promotions.md) §5） |
| Order | Saga 補償失敗的最終處理與告警機制（[17](17-service-order.md) §6） |
| Order | 逾時未付款自動取消、訂單編號策略需另訂（[17](17-service-order.md) §6） |
| Payment | 三家廠商沙箱實測（[18](18-service-payment.md) §8） |
| Payment | 退款金流串接未實作（[18](18-service-payment.md) §8） |
| Payment | 對帳排程未實作（[18](18-service-payment.md) §8） |
| Media | 影片縮圖（需 ffmpeg）（[19](19-service-media.md) §7） |
| Media | CDN 加速是否需要（[19](19-service-media.md) §7） |
| CMS | Page Builder 實作方式（自建 vs 簡化版）（[20](20-service-cms.md) §5） |
| CMS | `StoreSettings` 歸屬未定案（[20](20-service-cms.md) §5） |
| Shipping | 生鮮水果溫控物流邏輯（[21](21-service-shipping.md) §5） |
| Shipping | 超商取貨門市選擇未串接（[21](21-service-shipping.md) §5） |
| Analytics | 報表查詢效能（即時彙總 vs 物化檢視）（[22](22-service-analytics.md) §6） |
| Analytics | 報表匯出（CSV/Excel）未實作（[22](22-service-analytics.md) §6） |
| Analytics | 熱銷排行/付款分布圖表函式庫選型（[22](22-service-analytics.md) §6） |
| Notification | LINE 官方帳號綁定歸屬（賣家各自 vs 站台統一）（[23](23-service-notification.md) §7） |
| Notification | LINE Messaging API 額度與費用評估（[23](23-service-notification.md) §7） |
| Notification | Email/簡訊是否併入本服務（[23](23-service-notification.md) §7） |
| Reviews | 評價審核機制（人工 vs 預設顯示）（[24](24-service-reviews.md) §5） |
| Gateway | 速率限制實際執行方式（記憶體 vs Redis）（[25](25-service-gateway.md) §7） |
| Gateway | 是否需要 Webhook 主動推播（[25](25-service-gateway.md) §7） |
| Gateway | 是否開放建立訂單的公開 API（[25](25-service-gateway.md) §7） |
| Gateway | Gateway 聚合文件呈現方式（[25](25-service-gateway.md) §7） |

## 5. 跨服務/基礎設施（[26](26-project-structure.md)–[29](29-shared-service-conventions.md)）

| 項目 | 來源 |
|---|---|
| ShyeCMS 前端需求規格尚未撰寫 | [26](26-project-structure.md) §7 |
| 私有 NuGet feed 與 npm registry 服務選型未定 | [26](26-project-structure.md) §7 |
| 六個 repo 的 CI/CD 與跨 repo 版本協調 SOP 未定 | [26](26-project-structure.md) §7 |
| `ecommerce-deploy` 版本標籤更新流程（人工 vs 自動化）未定 | [26](26-project-structure.md) §7 |
| `services/*/Dockerfile` 實際內容未撰寫 | [26](26-project-structure.md) §7 |
| 精確斷點寬度、色彩對比 Token 待設計系統文件定案 | [27](27-pwa-and-accessibility.md) §4 |
| 賣家後台是否也要納入 WCAG AA | [27](27-pwa-and-accessibility.md) §4 |
| Lighthouse/axe-core 自動化稽核是否納入 CI | [27](27-pwa-and-accessibility.md) §4 |
| PWA 快取版本更新策略未定 | [27](27-pwa-and-accessibility.md) §4 |
| ShyeCMS 自己的前端是否也要 RWD/PWA（暫定不需要） | [27](27-pwa-and-accessibility.md) §4 |
| 翻譯內容由誰提供（人工 vs 機器翻譯） | [28-i18n.md](28-i18n.md) §8 |
| 日文/英文版 SEO 優先度是否與繁中相同 | [28-i18n.md](28-i18n.md) §8 |
| LINE 通知訊息是否需要多語系 | [28-i18n.md](28-i18n.md) §8 |
| 賣家後台是否強制至少填寫繁中版本 | [28-i18n.md](28-i18n.md) §8 |
| 英文/日文版是否服務海外客群（連動幣別/金流重新設計） | [28-i18n.md](28-i18n.md) §8 |
| 高權限帳號是否強制 2FA | [29](29-shared-service-conventions.md) §5 |
| 結構化 log 集中收集方案選型 | [29](29-shared-service-conventions.md) §5 |
| CSP 詳細規則需逐服務盤點 | [29](29-shared-service-conventions.md) §5 |
| 服務身分 JWT 的簽發頻率與快取策略 | [29](29-shared-service-conventions.md) §5 |

## 6. 重複出現超過一次的項目（值得合併決策，而非分開處理）

| 主題 | 出現位置 |
|---|---|
| `StoreSettings` 歸屬 | [14](14-service-vendor.md)、[20](20-service-cms.md)、[10-gap-analysis.md](10-gap-analysis.md) |
| Saga 補償失敗處理 | [17](17-service-order.md)、[16](16-service-promotions.md)、[13](13-service-wms.md) |
| LINE / Google OAuth 串接時程 | [07](07-storefront-requirements.md)、[11](11-service-identity.md) |
| Gateway 聚合文件呈現方式 | [09](09-api-specification.md)、[25](25-service-gateway.md) |
| 多賣家入駐後需要平台管理員角色 | [05](05-scope-and-open-items.md)、[14](14-service-vendor.md)、[16](16-service-promotions.md) |

**建議**：這 5 組與其分散在多份文件各自等待決定，不如各自安排一次性的決策會議一次解決——尤其是 Saga 補償失敗處理，三個服務的答案其實應該是同一套設計（統一的補償失敗告警機制），不是三個服務各自想各自的。
