# 19 - Media Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |
| v0.2 | 2026-09-08 | ordinarycas | `MediaAsset` 補上 `VendorId` 欄位——先前沒有歸屬欄位，無法對照 `VendorStorageQuota` 算出「這個賣家用了多少配額」（見 [10-gap-analysis.md](10-gap-analysis.md) §10） |
| v0.3 | 2026-09-10 | ordinarycas | §7 解決 2 項待決議：影片縮圖補上 ffmpeg 技術設計（尚未實作）、CDN 加速定案現階段不需要，回應「將待決議事項列出來實作」需求 |

## 1. 職責

檔案上傳、儲存後端切換、縮圖產生、賣家上傳配額。

## 2. 支援的儲存後端

| 方式 | 適用情境 | 影片 | 注意事項 |
|---|---|---|---|
| 本機磁碟 | 單機部署（符合本平台單一 VPS 拓樸） | ✓ | 多副本部署時各副本磁碟不共用，本平台單一 VPS 拓樸下無此疑慮 |
| FTP / FTPS | 客戶已有虛擬主機或自架伺服器 | ✓ | 需另有 Web 伺服器對應到上傳目錄 |
| S3 相容物件儲存 | 正式營運建議方案，相容 AWS S3 / MinIO / R2 / B2 | ✓ | 需搭配 Data Protection 金鑰持久化，避免容器重建後憑證失效 |
| Imgur | 客戶沒有儲存空間、想最快上線 | ✗ | 只支援圖片；免費額度有限；檔案存第三方，Imgur 有權移除內容，不建議正式營運長期依賴 |

**一次只啟用一種**，切換儲存方式不會自動搬移既有檔案，需個別執行搬移工具。

## 3. 資料模型

| 實體 | 說明 |
|---|---|
| MediaAsset | `VendorId`（歸屬賣家，用於配額計算）、Url、Provider、ThumbnailBytes、SortOrder |
| StorageProviderSettings | 目前啟用的儲存後端與其憑證（加密存放） |
| VendorStorageQuota | 逐賣家可覆寫的上傳配額，全站有預設值；目前用量＝依 `VendorId` 加總該賣家所有 `MediaAsset` 的檔案大小 |

## 4. 安全處理

| 項目 | 做法 |
|---|---|
| 檔名 | 一律改為隨機 GUID + 副檔名，避免路徑穿越與覆蓋既有檔案 |
| 格式白名單 | 圖片 JPEG/PNG/WebP/GIF/AVIF；影片 MP4/WebM/MOV |
| 大小上限 | 圖片 10MB、影片 200MB |
| 縮圖 | 產生多尺寸 WebP，降低列表頁頻寬 |

## 5. 爸芭樂案例

商品照片、產地照，芭樂新鮮度展示建議搭配多角度縮圖。

## 6. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `POST /api/v1/vendor/media` | 上傳檔案 | 賣家 |
| `DELETE /api/v1/vendor/media/{id}` | 刪除檔案（回收配額） | 賣家 |
| `GET /api/v1/vendor/media/quota` | 查詢目前配額用量 | 賣家 |
| `POST /internal/v1/media/thumbnail-jobs` | 縮圖背景 Worker 內部佇列 | 內部 |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 7. 待決議事項
- [x] ~~影片縮圖（需 ffmpeg，會增加映像檔體積與 CPU 需求）~~——**部分解決（設計已補齊，尚未實作）**：上傳影片時，於 Infrastructure 層呼叫系統安裝的 `ffmpeg`（`ffmpeg -i input.mp4 -ss 00:00:01 -vframes 1 thumbnail.jpg`，擷取第 1 秒畫面），產生的縮圖依既有圖片儲存流程處理（含 §4 的安全檢查）；Dockerfile 需在 runtime image 額外 `apt-get install ffmpeg`（目前 15 服務共用的 `mcr.microsoft.com/dotnet/aspnet:10.0` base image 不含 ffmpeg）。**仍待實作**：`ecommerce-services` 目前對任何檔案類型皆走同一套上傳流程（`VendorMediaController` 未區分圖片/影片），這是本規格庫目前唯一尚未動手實作的部分，記錄設計避免又成為只活在腦中的缺口
- [x] ~~CDN 加速是否需要~~——**已解決：現階段不需要**。理由：本平台鎖定單一 VPS、每客戶獨立部署、非高流量（見 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md)），CDN 主要解決的是「地理distributed 使用者」與「大流量」兩個問題，與現況（單一客戶、單一地區的台灣買家）不符；先以應用伺服器直接回應媒體檔案，待客戶流量或地理分布真的需要時再評估導入
