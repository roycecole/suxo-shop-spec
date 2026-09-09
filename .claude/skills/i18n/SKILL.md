---
name: i18n
description: "查詢多語系支援規格：繁中（預設）/英/日三語的 URL 路由策略、內容翻譯資料模型、fallback 策略，以及明確排除的幣別/日期/數字在地化。"
---

# 28 - 多語系支援 (Internationalization)

來源文件：`docs/28-i18n.md`

## 這份文件負責回答
電商平台前台+後台的多語系範圍與資料模型策略；ShyeCMS 不在多語系範圍內。

## 涵蓋章節
- 1. 範圍
- 2. URL 路由策略
- 3. 內容翻譯的資料模型策略（含 3.1 共用套件 `SuxoShop.Shared.Translation` 去留，已定案不採用，見下方使用注意）
- 4. Fallback 策略
- 5. UI 文字（非使用者輸入內容）
- 6. 與既有決策的交互影響
- 7. 明確排除：本文件不涵蓋幣別/日期/數字在地化
- 8. 待決議事項（5 項已全數解決，見下方使用注意）

## 使用注意
- §7 明確排除幣別/日期/數字在地化——不要因為要做多語系就順帶擴大到幣別轉換等未規劃的範圍。
- **§3.1 是 2026-09-09（v0.6）定案的整節**：[26-project-structure.md](26-project-structure.md) 原規劃的共用 NuGet 套件 `SuxoShop.Shared.Translation` **決定不採用**——`ecommerce-services` 的 Catalog/CMS/Promotions/Shipping 四服務早已各自獨立實作 `Translation` 表，套件本身從未被任何服務接線。套件程式碼保留在 repo 內降級為**參考範本**（新增第 5 個需要翻譯欄位的服務時可參考其欄位設計，但非強制依賴），既有 4 服務不會回頭遷移。查詢 `ecommerce-services` 的 shared 套件清單或多語系實作時，不要假設這個套件是現行依賴的一部分。
- 同一節也定案 **zh-Hant 版本不落在 `Translation` 表裡**，沿用各服務主表原欄位（如 `Product.Name`）；`Translation` 表只存 en/ja 等非預設語言的覆寫值。
- §8 待決議事項 5 項已全數解決（v0.5、v0.7），不要以為還有未決項目：翻譯內容定案**人工輸入**（非機器翻譯）；EN/JA SEO 優先度低於繁中（SSG 預產名額以繁中為主）；LINE 通知維持繁中單一語系；賣家後台上架須檢核 zh-Hant 必填欄位（未填擋下上架）；英文/日文版正式定案**不服務海外客群**，幣別維持 TWD 單一計價、金流架構不變。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
