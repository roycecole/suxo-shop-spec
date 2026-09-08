# 20 - CMS Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)、[09-api-specification.md](09-api-specification.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | `RichText` 區塊內容改為 Markdown 儲存，回應「後台內容編輯使用 Markdown」需求 |
| v0.3 | 2026-09-08 | ordinarycas | §2 補上 `Translation` 表，落實 [28-i18n.md](28-i18n.md) §3 列出但本文件尚未實作的多語系需求 |

## 1. 職責

首頁/形象頁版型管理。每個客戶站台首頁由多個可插拔區塊組成，賣家可調整區塊順序、內容與顯示/隱藏；未自訂時套用系統預設版型。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| PageLayout | PageType（Home/AboutUs/Custom）、IsDefault、Status（Draft/Published） |
| PageSection | Type（Banner/FeaturedCategories/ProductBlock/VendorSpotlight/RichText/Custom）、Config（jsonb）、SortOrder、IsVisible |
| Translation | EntityType（"PageSection"）、EntityId、LocaleCode、FieldName（`Config` 內需要翻譯的子欄位路徑，如 Banner 文案、RichText 內容）、Value——結構沿用 [28-i18n.md](28-i18n.md) §3 的共用模式 |

> `Type = RichText` 的區塊，`Config` 內的內容欄位以 **Markdown** 格式儲存（賣家後台用 Markdown 編輯器輸入），前台渲染時轉成 HTML 顯示（沿用 [29-shared-service-conventions.md](29-shared-service-conventions.md) §2 的共用 Markdown 處理管線，不自行實作）。
>
> `Config` 是半結構化 jsonb，並非每個子欄位都需要翻譯——只有文字內容（Banner 文案、RichText 本文）透過 `Translation` 查詢，結構性欄位（圖片 URL、排序數字、顯示分類 Id）不分語言、所有語言共用同一份。

## 3. 爸芭樂案例

品牌故事、產地介紹等形象內容區塊；首頁主打當季芭樂品種的 Banner 輪播。

## 4. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/cms/page-layouts/{pageType}` | 取得已發佈的頁面版型（前台 SSG 建置時呼叫） | 公開 |
| `PUT /api/v1/cms/page-layouts/{pageType}` | 更新版型內容（區塊順序、顯示/隱藏、Config） | 賣家 |
| `POST /api/v1/cms/page-layouts/{pageType}/publish` | 將草稿版型發佈 | 賣家 |
| `POST /internal/v1/cms/revalidate-webhook` | 版型變更時通知前台 Next.js 觸發 ISR 重新產生 | 內部（Gateway 或 CMS 自己觸發） |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 5. 待決議事項
- [ ] Page Builder 實作方式：自建拖拉式編輯器，還是先做「後台表單設定區塊參數」的簡化版
- [ ] `StoreSettings`（賣家自家功能開關）是否應歸屬本服務而非 Vendor Service（見 [14-service-vendor.md](14-service-vendor.md) §4）
