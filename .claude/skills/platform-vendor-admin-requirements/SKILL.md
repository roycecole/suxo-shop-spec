---
name: platform-vendor-admin-requirements
description: "查詢賣家後台需求：上架/訂單/數據等核心功能、賣家子帳號、拾夜科技 PlatformSupportStaff 支援權限、WooCommerce 商品 CSV 匯出。"
---

# 08 - 賣家後台需求 (Vendor Admin Requirements)

來源文件：`docs/08-vendor-admin-requirements.md`

## 這份文件負責回答
賣家（爸芭樂店主）後台功能、賣家自己管理的功能開關（與 ShyeCMS 無關）、子帳號角色、拾夜科技支援人員的唯讀診斷權限、資料可攜出匯出功能。

## 涵蓋章節
- 1. 賣家（爸芭樂店主）核心功能
- 2. 自家系統功能開關（賣家自己管理，與 ShyeCMS 無關）
- 3. 賣家角色與子帳號
- 4. 拾夜科技資訊人員的支援權限（新增）
- 5. 資料可攜出：匯出 WooCommerce 商品 CSV
- 6. 待決議事項
- 7. RWD / PWA

## 使用注意
- §2 的「自家系統功能開關」是賣家自己在後台管理的開關，跟 shyecms-architecture 說的 ShyeCMS 合約紀錄是兩回事，不要混淆。
- 本輪暫定爸芭樂為單一賣家自營，沒有「客戶自己的平台管理員」角色（審核多賣家等）——見 shyecms-scope-and-open-items 的排除說明。
- **§5.3 Grouped 商品的 WooCommerce 匯出分隔符號已訂正（v0.8）**：先前文件誤植為逗號分隔，經查證 WooCommerce 官方 CSV 匯出格式後訂正為**直線符號 `|`**（如 `SKU1|SKU2|SKU3`，或 `id:100|id:101`）——這是下游實作必須採用的正確格式，用逗號會讓匯入 WooCommerce 端整串被當成一個 SKU 而失敗，是本輪查證後揪出的真實缺陷，不只是文字調整。
- §6 待決議事項 6 項已全數解決（v0.7、v0.8），不要以為還有未決項目：稅務欄位定案不新增、`PlatformSupportStaff` 白名單診斷端點盤點完成（順手補上 Promotions/Notification 兩個原本遺漏的端點）、即時通知定案不需要（AuditLog 已足夠）、會員 Email/電話遮罩顯示定案（`t***@example.com`／`09XX-XXX-123`）、匯出下載連結時效定案 **30 分鐘簽章網址**、Grouped 分隔符號如上已訂正。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
