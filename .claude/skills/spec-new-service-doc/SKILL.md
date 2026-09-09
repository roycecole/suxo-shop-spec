---
name: spec-new-service-doc
description: "新增一個電商平台微服務規格文件（依 11–25 既有服務文件的既定範本），並同步更新所有會列出服務清單的索引文件。"
---

# 新增微服務規格文件

依循 `docs/11-service-identity.md`–`docs/25-service-gateway.md` 既有的服務文件範本，新增一份微服務規格。

## 檔案範本結構

新檔案命名為 `docs/{編號}-service-{service-name}.md`（編號延續 30 之後，或視情況與使用者確認插入位置），內容至少包含：

```
# {編號} - {Service Name}

## 異動紀錄
（標準四欄表格，第一列 v0.1 初版建立）

## 1. 職責
一句話說明這個服務負責什麼、不負責什麼（跟哪個服務的邊界要講清楚，尤其如果容易跟既有服務混淆，例如新服務若碰觸庫存/金流等敏感邊界，要對照 service-wms/service-payment 說明差異）。

## 2. 資料模型
新服務自己的實體/欄位表格。

## 3. 爸芭樂案例
用爸芭樂案例具體化這個服務的用途（比照既有服務文件慣例）。

## 4. API 大綱
遵守 platform-api-specification（09-api-specification.md）的版本控管與文件格式規範。

## 5. 待決議事項
- [ ] ...
```

（部分服務文件在「職責」與「API 大綱」之間插入專屬章節，如 payment 的回調安全機制、wms 的庫存扣減機制——視新服務性質決定是否需要。）

## 必須遵守

1. **一律套用 shared-service-conventions（29-shared-service-conventions.md）**：健康檢查端點、Correlation ID、結構化 log、Markdown 消毒（若涉及富文本欄位）、服務間認證，都不要在新服務文件裡重新定義，只在需要偏離慣例時特別註明。
2. 服務間只能透過內部 REST 呼叫或 Saga/通知呼叫，不可假設能直接查詢其他服務的資料表。
3. 每服務獨立 PostgreSQL schema（見 platform-architecture §6.4）。

## 新增後必須同步的索引（容易漏掉，逐一確認）

- `docs/00-overview.md` §6 文件索引表
- `docs/06-ecommerce-platform-architecture.md` §3 架構圖（Mermaid）、§4 服務邊界劃分表格
- `docs/09-api-specification.md` §3 各服務 API 文件索引
- `docs/26-project-structure.md` §3.1 `ecommerce-services/services/` 樹狀圖、§6 專案數量總計
- `docs/30-open-decisions-register.md` §4「15 個微服務」分組（若新服務一開始就有待決議事項）
- 若使用者要求同步建立對應的 Claude Skill，比照 `.claude/skills/service-*` 既有結構新增一個。

## 完成後

依 spec-add-changelog-entry 在新文件與所有被同步的索引文件裡各自新增一筆異動紀錄。
