# 22 - Analytics Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | 新增 §4：圖表函式庫確定用 TradingView Lightweight Charts，回應「圖表使用 tradingview lightweight-charts」需求 |
| v0.3 | 2026-09-08 | ordinarycas | §1 修正「訂閱事件」措辭——與 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §2「不引入訊息佇列」的決策矛盾，改為明確的定期輪詢/批次拉取，解決 [10-gap-analysis.md](10-gap-analysis.md) §11 已列的缺口 |
| v0.4 | 2026-09-09 | ordinarycas | §4 補上熱銷排行/付款分布的圖表函式庫選型：Chart.js（`react-chartjs-2`），與 Lightweight Charts 職責互補；§6 對應待決議項標記已解決 |

## 1. 職責

報表/數據分析，唯讀服務。無自有寫入表，定期輪詢/批次拉取其他服務的 REST API 建置投影——本平台不引入訊息佇列（見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §2），本服務不例外，不走事件訂閱/pub-sub 機制。

## 2. 資料來源

| 來源服務 | 用途 |
|---|---|
| Order Service | 銷售趨勢、GMV |
| Catalog Service | 熱銷商品排行 |
| Payment Service | 付款方式分布 |

## 3. 爸芭樂案例

銷售趨勢、熱銷品種排行（如珍珠芭樂 vs 帝王芭樂銷量比較）。

用量統計（GMV、商品數等）留在本服務內供賣家自己檢視，**不會、也沒有機制回傳給拾夜科技**——這是 ShyeCMS 決策 D 的直接落實，見 [01-architecture.md](01-architecture.md)。

## 4. 前端圖表函式庫

時間序列類報表（銷售趨勢、GMV 走勢）採 **[TradingView Lightweight Charts](https://github.com/tradingview/lightweight-charts)**（Apache-2.0 授權，可商用）：體積小（約 45KB gzip）、效能佳，原生支援 Line/Area/Candlestick/Histogram 等時間序列圖表類型，直接對應銷售趨勢圖的需求。

**適用範圍的限制**：Lightweight Charts 是**時間序列/金融圖表**專用函式庫，**不適合**用來畫熱銷商品排行（類別型長條圖）或付款方式分布（圓餅圖）這類非時間軸資料，需另選一套通用圖表方案。

**熱銷排行/付款分布採 [Chart.js](https://www.chartjs.org/)（透過 [react-chartjs-2](https://react-chartjs-2.js.org/) 包裝）**：MIT 授權、生態成熟、原生支援 Bar/Pie/Doughnut 等類別型圖表，體積輕量（核心約 60KB gzip，可依實際用到的圖表類型 tree-shaking），與 Lightweight Charts 職責互補（一個管時間序列，一個管類別/比例型資料），不強求兩種圖表類型共用同一套函式庫。這兩種報表都只出現在**賣家後台**（Vite SPA，[08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) §1「銷售數據」），純 CSR 情境不需考慮 SSR 相容性，`react-chartjs-2` 的 React 元件封裝可直接套用。

| 報表 | 圖表類型 | 函式庫 |
|---|---|---|
| 銷售趨勢、GMV 走勢 | Line/Area（時間序列） | TradingView Lightweight Charts |
| 熱銷商品排行 | 長條圖（類別排名） | Chart.js（`react-chartjs-2`） |
| 付款方式分布 | 圓餅/環圖 | Chart.js（`react-chartjs-2`） |

## 5. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/vendor/analytics/sales` | 銷售摘要/趨勢（時間序列，供 Lightweight Charts 使用） | 賣家 |
| `GET /api/v1/vendor/analytics/top-products` | 熱銷商品排行 | 賣家 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 6. 待決議事項
- [ ] 報表查詢效能：直接對交易表即時彙總，資料量成長後需評估預先彙總表或物化檢視
- [ ] 報表匯出（CSV/Excel）供會計對帳
- [x] ~~熱銷排行/付款分布的圖表函式庫選型（見 §4，Lightweight Charts 不適用）~~——**已解決**：採 Chart.js（`react-chartjs-2`），見 §4
