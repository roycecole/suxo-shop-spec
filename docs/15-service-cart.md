# 15 - Cart Service

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 從 [06-ecommerce-platform-architecture.md](06-ecommerce-platform-architecture.md) 拆分獨立，回應「微服務拆成多個規格」需求 |

## 1. 職責

購物車，含會員與訪客兩種身分。

## 2. 資料模型

| 實體 | 說明 |
|---|---|
| Cart | 會員為 UserId，訪客為 Cookie/SessionId |
| CartItem | ProductId/VariationId、Quantity |

## 3. 爸芭樂案例

訪客免登入即可加入購物車，見 [07-storefront-requirements.md](07-storefront-requirements.md) §1。結帳時由 Order Service 呼叫本服務取得購物車內容（唯讀），見 [17-service-order.md](17-service-order.md) 的 Saga 流程。

## 4. API 大綱

| Method & Path | 說明 | 認證 |
|---|---|---|
| `GET /api/v1/cart` | 取得目前購物車（依 Cookie/JWT 識別） | 公開（含訪客） |
| `POST /api/v1/cart/items` | 加入商品 | 公開（含訪客） |
| `PUT /api/v1/cart/items/{id}` | 修改數量 | 公開（含訪客） |
| `DELETE /api/v1/cart/items/{id}` | 移除商品 | 公開（含訪客） |
| `GET /internal/v1/cart/{cartId}` | 結帳 Saga 內部呼叫：取得購物車內容 | 內部（僅 Order Service） |

版本控管與文件格式沿用 [09-api-specification.md](09-api-specification.md) 的通用規範。

## 5. 待決議事項
- [ ] 訪客購物車的自動清理排程（多久未更新視為過期）
