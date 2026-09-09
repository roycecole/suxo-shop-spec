---
name: spec-check-cross-references
description: "檢查 docs/ 內所有相對連結（[文字](檔名.md)）是否都指向實際存在的檔案，以及各處文件索引（00 §6、06 §4/§11、09 §3、26、30）彼此是否與目前的檔案清單一致。"
---

# 檢查文件交叉引用一致性

`docs/` 內大量使用相對 Markdown 連結（如 `[09-api-specification.md](09-api-specification.md)`）互相引用，且有多處各自維護的「文件索引/服務清單」，容易在新增/刪除/搬移文件後出現連結失效或清單漏更新。

## 檢查步驟

1. 列出 `docs/` 目前實際存在的所有檔案（`ls docs/*.md`）。
2. 搜尋所有 Markdown 連結（`grep -oE '\[[^]]+\]\([0-9]{2}-[a-z0-9-]+\.md[^)]*\)' docs/*.md`），確認每個被引用的檔名都在步驟 1 的清單裡；找出「引用了不存在的檔案」或「檔案存在但沒有任何文件引用它」兩種情況。
3. 逐一核對以下幾處「索引/清單」是否與步驟 1 的實際檔案清單一致（檔名、標題、一句話說明三者都要對得上，不只是檔名）：
   - `docs/00-overview.md` §6 文件索引
   - `docs/06-ecommerce-platform-architecture.md` §4 服務邊界劃分表、§11 相關文件
   - `docs/09-api-specification.md` §3 各服務 API 文件索引
   - `docs/26-project-structure.md`（repo/服務數量統計）
   - `docs/30-open-decisions-register.md`（分組是否涵蓋所有目前存在、且真的有待決議事項的文件）
4. 若發現不一致，修正對應索引文件，並依 spec-add-changelog-entry 記錄。

## 常見觸發時機

- 新增/刪除/搬移任何 `docs/*.md` 文件之後（尤其是用 spec-new-service-doc 新增服務文件之後）。
- 使用者要求「檢查文件連結」「確認索引有沒有跟上」時。
