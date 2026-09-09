---
name: pwa-and-accessibility
description: "查詢前台/後台的 RWD 響應式設計與 PWA 可安裝規範，以及僅前台適用的 WCAG 2.1 AA 無障礙規範。"
---

# 27 - RWD / PWA / 無障礙規範 (Responsive, PWA & Accessibility)

來源文件：`docs/27-pwa-and-accessibility.md`

## 這份文件負責回答
RWD、PWA（可安裝）兩者前台後台皆須支援；無障礙規範明確排除後台，僅前台適用。

## 涵蓋章節
- 1. RWD（響應式設計）
- 2. PWA（可安裝的漸進式網路應用程式）
- 3. 無障礙規範（僅前台，依需求明確排除後台）
- 4. 待決議事項（5 項已全數解決，見下方使用注意）

## 使用注意
- **§4 待決議事項 5 項已於 2026-09-10（v0.2）全數解決**——不要誤以為這仍是待補的開放領域。斷點定案 `640px`/`768px`/`1024px`（§1，業界慣用級距，非等設計系統文件產出）；色彩對比已由 `ecommerce-storefront` 的 `scripts/check-contrast.mjs` 實測驗證通過 WCAG AA（4.5:1 一般文字／3:1 大字與 UI 元件），不必再等設計系統文件核對。
- 賣家後台 WCAG AA、`shyecms-admin` 的 RWD/PWA 兩項皆已**正式定案為不需要**（非本輪暫定，未來有具體需求再另外提出，不主動預判）；PWA 快取版本策略定案為 `CACHE_NAME` 附加應用程式版本號＋SW `activate` 事件清除舊快取＋`skipWaiting()`/`clients.claim()` 的標準 pattern；Lighthouse/axe-core 自動化稽核待 [26-project-structure.md](26-project-structure.md) §7 已定案的 CI/CD SOP 建置完成後順勢納入，現階段以 `check-contrast.mjs` 手動腳本頂著。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
