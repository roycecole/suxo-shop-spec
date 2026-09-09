---
name: platform-architecture
description: "查詢電商平台（以爸芭樂為案例）的整體架構：技術棧、15 服務邊界、部署拓樸、資料庫連線模式、結帳 Saga 流程。"
---

# 06 - 電商平台架構：以「爸芭樂」為案例 (Ecommerce Platform Architecture)

來源文件：`docs/06-ecommerce-platform-architecture.md`

## 這份文件負責回答
整套電商平台的頂層架構文件：技術棧總覽、架構圖、服務邊界劃分、前端 SSG/ISR 規格、部署拓樸與 DB 三種連線模式、結帳 Saga、客戶拿到的頁面/功能清單。

## 涵蓋章節
- 0. 定位聲明
- 1. 案例背景：爸芭樂
- 2. 技術棧總覽
- 3. 整體架構圖
- 4. 服務邊界劃分
- 5. 前端：React 靜態頁面生成規格
- 6. 部署拓樸與資料庫設計原則
- 7. 結帳流程（Saga）
- 8. 客戶拿到的頁面/功能清單（對應原始需求第 5 項）
- 9. 與 ShyeCMS 的關係（重申決策 C）
- 10. 待決議事項
- 11. 相關文件

## 使用注意
- 這是全新產品線，**直接以微服務起步**，不是模組化單體演進而來——不要建議「先做成單體再拆」之類的路徑，那與此文件的定位聲明（§0）矛盾。
- 新增或調整服務邊界時，§4 服務清單與 §3 架構圖要同步更新，並確認 09-api-specification.md §3、30-open-decisions-register.md §4 的索引也對得上。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
