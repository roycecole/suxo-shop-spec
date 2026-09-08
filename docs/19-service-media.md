# 19 - Media Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |

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
| MediaAsset | Url、Provider、ThumbnailBytes、SortOrder |
| StorageProviderSettings | 目前啟用的儲存後端與其憑證（加密存放） |
| VendorStorageQuota | 逐賣家可覆寫的上傳配額，全站有預設值 |

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
- [ ] 影片縮圖（需 ffmpeg，會增加映像檔體積與 CPU 需求）
- [ ] CDN 加速是否需要
