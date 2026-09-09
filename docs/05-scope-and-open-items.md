# 05 - 本輪範圍與待決議清單 (Scope & Open Items)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立 |
| v0.2 | 2026-09-08 | ordinarycas | 因應決策 C、D 推翻：移除已不適用的「即時推送」「憑證輪替」等待決議項；新增「零連接」為已鎖定決策而非開放項目；新增與 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 相關的排除項目 |
| v0.3 | 2026-09-08 | ordinarycas | 與前身規格全面比對後，新增 2 項明確排除：平台管理員角色（暫定單一賣家自營）、本文件集專屬開發環境文件（先列待辦） |
| v0.4 | 2026-09-08 | ordinarycas | 釐清 §3 範圍：本節標題原稱「待決議事項彙整」但實際只涵蓋 ShyeCMS 文件（[01](01-architecture.md)–[03](03-client-lifecycle.md)），並未包含電商平台/微服務側累積的其他待決議事項，容易誤導成「全規格彙整」。改名並加註說明，完整跨文件清單見新增的 [30-open-decisions-register.md](30-open-decisions-register.md) |
| v0.5 | 2026-09-08 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) 第八輪複查發現本節比 [30-open-decisions-register.md](30-open-decisions-register.md) §2 少列 2 項（皆屬 ShyeCMS 範圍卻遺漏同步）：補上「GMV 超額抽成的計算依據完全空白」（[02-data-model.md](02-data-model.md) §6）與「開通部署（Provisioning）是否要自動化」（[03-client-lifecycle.md](03-client-lifecycle.md) §7） |

> 明確寫出「本輪刻意不做什麼」，避免日後誤以為是遺漏。**本文件範圍限定 ShyeCMS（[00](00-overview.md)–[05](05-scope-and-open-items.md)）**，電商平台/微服務側的排除項目與待決議事項見各自文件與 [30-open-decisions-register.md](30-open-decisions-register.md)。

## 1. 已鎖定、不再開放討論的決策

- **ShyeCMS 與客戶平台零連接**：不存在任何 API 呼叫、憑證、輪詢機制。這不是待決議項，是使用者明確推翻 v0.1 設計後鎖定的決策，見 [01-architecture.md](01-architecture.md)。
- **ShyeCMS 不取得客戶商品/售價/會員資料**：包含彙總數據也不取得（比 v0.1 決策 D「只取彙總」更進一步）。

## 2. 本輪明確排除項目

| 項目 | 說明 | 排除原因 |
|---|---|---|
| 形象網站 / 部落格產品線 | ShyeCMS 資料模型雖通用，功能開關清單與 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 都是針對電商設計 | 決策 A：本輪先聚焦電商案例（爸芭樂） |
| 客戶自助登入 ShyeCMS | 客戶（如爸芭樂）本身沒有 ShyeCMS 帳號 | 決策 B：純內部工具 |
| 功能授權的技術強制機制 | ShyeCMS 的 `ClientFeatureEntitlement` 只是合約紀錄，客戶環境的實際設定由人工落地，沒有任何自動驗證或防呆 | 決策 C：零連接前提下，技術強制需要另一套完全不同的機制（如離線 License Key），本輪不做 |
| GMV 超額抽成的計費/收款系統 | 沒有任何用量資料來源可用 | 決策 D：ShyeCMS 不取得客戶資料，此缺口本輪未被推進，維持空白 |
| 開通部署自動化（IaC 一鍵建環境） | ShyeCMS 只記錄客戶合約與部署盤點資訊 | 沿用既有人工 SOP（環境建置 → 資料庫初始化 → 品牌設定 → 金流物流串接測試 → 教育訓練 → 正式上線） |
| 客戶終止合作的會員/訂單資料交還/刪除流程 | [03-client-lifecycle.md](03-client-lifecycle.md) §6 僅記錄狀態變更；商品資料已可透過 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §5 WooCommerce 匯出自行取得 | 會員/訂單資料涉及個資法與合約條款，本輪僅解決商品資料部分 |
| 平台管理員（Admin）角色規格 | 目前僅有「賣家」（[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md)）與拾夜科技支援角色，沒有「客戶自己的平台管理員」（賣家審核、全站金流物流設定、客訴仲裁）對應規格 | 爸芭樂案例**暫定為單一賣家自營**，賣家審核/多賣家仲裁等場景不適用；若未來此平台需要開放多賣家入駐，需回頭補上對應 Admin 角色與規格 |
| 本文件集專屬的開發環境建置文件 | 尚未撰寫對應 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 的 15 服務、單一 VPS、三種 DB 連線模式的 Docker Compose 啟動指令、Migration 指令等 | 列為待辦，待實際進入建置階段再撰寫（避免規格早於實作、規格本身還會隨開發過程調整） |

## 3. 待決議事項彙整（僅 ShyeCMS 範圍）

- [ ] 若未來出現濫用/超用糾紛，要靠什麼機制舉證（合約條款？人工稽核頻率？）（見 [01-architecture.md](01-architecture.md) §5）
- [ ] `ClientSubscription.Status = PastDue` 時的標準作業流程（見 [02-data-model.md](02-data-model.md) §6）
- [ ] GMV 超額抽成的計算依據完全空白（見 [02-data-model.md](02-data-model.md) §6）
- [ ] 開通部署（Provisioning）是否要自動化（見 [03-client-lifecycle.md](03-client-lifecycle.md) §7）
- [ ] `Suspended` 狀態下客戶站台該顯示什麼訊息給該客戶的買家/賣家（見 [03-client-lifecycle.md](03-client-lifecycle.md) §7）
- [ ] 客戶終止合作的資料交還/刪除政策（見 [03-client-lifecycle.md](03-client-lifecycle.md) §7）
- [ ] GMV 計費若要恢復可行性，需要客戶方同意的資料申報機制設計（不透過連線取得，見 [01-architecture.md](01-architecture.md) §5）

## 4. 建議下一步

1. 先以「爸芭樂」案例走一次 [03-client-lifecycle.md](03-client-lifecycle.md) 的建檔→開通→營運流程，驗證「ShyeCMS 記錄 + 人工落地設定」這個工作流程實際操作起來是否順暢（純流程驗證，不涉及任何系統整合測試，因為本來就沒有整合）。
2. 電商平台本身（[06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)）可獨立於 ShyeCMS 開發與驗證，兩者沒有依賴關係，可平行進行。
3. Pilot 驗證通過後，再評估是否擴充形象網站/部落格產品線的功能清單。
