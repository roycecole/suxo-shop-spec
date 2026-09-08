# 27 - RWD / PWA / 無障礙規範 (Responsive, PWA & Accessibility)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「畫面支援 RWD/PWA，前台支援網站無障礙規範」需求 |

> 無障礙規範原文（[數位發展部網站無障礙規範 110.07 版](https://accessibility.moda.gov.tw/Accessible/Guide/68)）本輪無法即時讀取（該站對本工具回應 403，瀏覽器擴充功能也未連接），本文件依此規範公開已知的基準（**WCAG 2.1 AA**）撰寫。**正式開發前應由人工核對官網最新版本，確認基準沒有變動。**

## 1. RWD（響應式設計）

前台（Next.js）與賣家後台（Vite SPA）**皆須支援**響應式設計，適配手機/平板/桌面三種尺寸。

| 裝置 | 寬度（暫定，待 [10-gap-analysis.md](10-gap-analysis.md) §5 的設計系統文件定案） |
|---|---|
| Mobile | < 640px |
| Tablet | 640px–1024px |
| Desktop | > 1024px |

實作原則：CSS Grid/Flexbox + 流體單位（`%`、`rem`、`clamp()`），避免固定 px 寬度在小螢幕破版；圖片需搭配 `srcset`/`sizes` 或 Next.js `<Image>` 依裝置提供適當尺寸。

## 2. PWA（可安裝的漸進式網路應用程式）

前台與後台**皆須**做成 PWA，讓手機/平板使用者可以「加入主畫面」，以類 App 的體驗使用。

### 2.1 技術實作

| 專案 | 工具 |
|---|---|
| 前台（Next.js） | 維護中的 PWA 外掛（如 `@ducanh2912/next-pwa`），建置時產生 Service Worker，相容於 SSG/ISR 輸出 |
| 後台（Vite SPA） | `vite-plugin-pwa` |

兩者都需要：`manifest.json`（App 名稱、圖示、`theme_color`、`background_color`）、多尺寸圖示（至少 192×192、512×512）、HTTPS（白牌部署本來就要求 HTTPS，見既有非功能需求基準）。

### 2.2 白牌客製化

`manifest.json` 的 App 名稱、圖示、主題色**依客戶各自的品牌設定**產生（如「爸芭樂」的 App 名稱與圖示需為芭樂品牌，不是通用預設值）——這與既有的品牌客製化模式（Logo/Banner/主色，見 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md)）一致，PWA 的品牌資訊應該從同一份商店設定讀取，不另外維護第二份。

### 2.3 快取策略

| 內容類型 | 策略 | 理由 |
|---|---|---|
| 靜態資源（JS/CSS/字型/圖示） | Cache First | 版本化檔名，變更會產生新檔案，可放心長期快取 |
| 商品頁 HTML（SSG/ISR 產出） | **Network First**，短 TTL 或不快取 HTML 本身 | 避免 Service Worker 快取住 ISR 已更新前的舊內容，與 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5 的 `/api/revalidate` 主動更新機制衝突 |
| API 回應（商品資訊、庫存） | Network First，失敗時退回快取 | 庫存/價格屬即時性資料，優先拿新資料 |
| 購物車/結帳/訂單（後台亦同：訂單/庫存操作） | **不快取，不支援離線寫入** | 這些是交易性操作，離線送出會造成資料不一致，斷網時應明確告知「需要網路連線」而非悄悄佇列稍後送出 |

### 2.4 明確排除
- **不支援離線交易**：購物車結帳、賣家出貨、庫存調整等寫入操作，斷網時一律阻擋並提示，不做離線佇列/背景同步（避免衍生庫存/訂單一致性問題，超出本輪範圍）。
- 推播通知（Web Push）不在本輪範圍內，即使 PWA 技術上可支援，与 [23-service-notification.md](23-service-notification.md) 的 LINE 通知是兩條獨立管道，不合併。

## 3. 無障礙規範（僅前台，依需求明確排除後台）

前台須符合 **WCAG 2.1 Level AA**，對應台灣「網站無障礙規範」110.07 版基準。**後台（賣家後台）本輪不強制**——後台是受過訓練的少數員工使用的內部工具，與面向不特定大眾的前台性質不同，見 §4 待決議。

### 3.1 四大原則對照

| WCAG 原則 | 具體要求 |
|---|---|
| 可感知（Perceivable） | 圖片需有 `alt` 文字（商品圖需描述商品，裝飾圖 `alt=""`）；文字與背景對比至少 **4.5:1**（一般文字）/ **3:1**（大字/粗體按鈕文字）——與既有色彩對比要求一致，一旦設計系統文件（[10-gap-analysis.md](10-gap-analysis.md) §5）定案色票，需逐一驗證是否達標；文字可縮放至 200% 不喪失功能或內容 |
| 可操作（Operable） | 全站可純鍵盤操作（Tab 順序合理、無鍵盤陷阱）；可見的 focus 樣式（不可用 `outline: none` 卻不提供替代樣式）；提供跳過導覽連結（Skip to content）；動畫可透過 `prefers-reduced-motion` 關閉（已於 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) §5.1 決議） |
| 可理解（Understandable） | 表單欄位需有對應 `<label>`；錯誤訊息需明確指出哪個欄位錯誤與如何修正；`<html lang="zh-Hant">`；導覽結構在各頁面間保持一致 |
| 穩健（Robust） | 語意化 HTML（`<nav>`、`<main>`、`<button>` 而非 `<div onclick>`）；適當使用 ARIA landmark，但優先用原生語意標籤而非額外疊加 ARIA；確保與主流螢幕報讀軟體（如 NVDA、VoiceOver）相容 |

### 3.2 驗證方式

- 自動化工具：axe-core / Lighthouke Accessibility 稽核，建議整合進前端建置流程（目前非功能需求的測試策略尚未涵蓋，見 [10-gap-analysis.md](10-gap-analysis.md)）。
- 人工測試：至少一次鍵盤操作全流程測試（瀏覽→加入購物車→結帳）與螢幕報讀軟體測試，自動化工具無法完全取代人工驗證。

## 4. 待決議事項
- [ ] 精確的斷點寬度、色彩對比 Token 待 [10-gap-analysis.md](10-gap-analysis.md) §5 建議的設計系統文件（編號待補，`27` 已被本文件使用）定案後回頭核對
- [ ] 賣家後台是否也要納入 WCAG AA（本輪依需求明確排除，但若賣家中有身心障礙使用者，可能需要重新評估）
- [ ] Lighthouse/axe-core 自動化稽核是否要納入 CI（見既有 CI/CD 缺口，[10-gap-analysis.md](10-gap-analysis.md) §3）
- [ ] PWA 快取版本更新策略（Service Worker 更新時如何提示使用者重新整理，避免舊版前端呼叫新版 API 造成不相容）
- [ ] 本文件範圍限定電商平台的前台+後台（[26-project-structure.md](26-project-structure.md) 的電商平台 repo）；ShyeCMS 自己的 `shyecms-admin`（拾夜科技員工內部使用）是否也要 RWD/PWA 尚未確認，暫定不需要（內部工具、使用場景固定在辦公室電腦），若有明確需求應另外提出
