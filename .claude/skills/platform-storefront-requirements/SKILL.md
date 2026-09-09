---
name: platform-storefront-requirements
description: "查詢電商平台前台（買家端）需求：免登入下單原則、可選登入方式、頁面清單與對應服務、商品現貨顯示規則。"
---

# 07 - 前台需求 (Storefront Requirements)

來源文件：`docs/07-storefront-requirements.md`

## 這份文件負責回答
前台核心原則是免登入即可下單；列出頁面清單並對應到負責的微服務，另有現貨顯示的降級行為考量。

## 涵蓋章節
- 1. 核心原則：免登入即可下單
- 2. 登入方式（可選，非必要）
- 3. 頁面清單與對應服務
- 4. 商品詳情頁的現貨顯示
- 5. 待決議事項
- 6. RWD / PWA / 無障礙規範

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
