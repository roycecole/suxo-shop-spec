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

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
