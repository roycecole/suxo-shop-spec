---
name: shared-service-conventions
description: "查詢所有 15 個微服務都必須遵守的共通慣例與資安基準：Correlation ID、健康檢查端點、結構化 log、Markdown 消毒管線、服務間認證、資安規則。"
---

# 29 - 跨服務共通慣例與資安基準 (Shared Service Conventions & Security Baseline)

來源文件：`docs/29-shared-service-conventions.md`

## 這份文件負責回答
本文件定義過的慣例，各服務文件（service-identity ... service-gateway）不會重複定義，只在需要偏離慣例時特別註明——查任何服務的可觀測性/資安相關問題前先看這裡。

## 涵蓋章節
- 1. 可觀測性
- 2. Markdown 處理管線（統一實作，避免各服務各自為政）
- 3. 服務間認證（解決既有待決議事項）
- 4. 資安基準（所有服務適用，含 4.1 共用套件資安修補強制升級窗口——已定案，見下方使用注意）
- 5. 待決議事項（4 項已全數解決，見下方使用注意）

## 使用注意
- 新增或修改任一微服務的規格時，凡涉及 log 格式、健康檢查、內部呼叫認證、Markdown 輸出清理，一律先確認是否已被本文件規範，不要讓某個服務自己另外發明一套。
- **§4.1 是 2026-09-09（v0.3）新增的整節**：`SuxoShop.Shared.*` 版本化套件（[26-project-structure.md](26-project-structure.md) §4.2）平常各服務可自行決定升級時機，但**資安修補例外**——凡涉及已知或懷疑安全漏洞的套件更新（不論是否已取得正式 CVE 編號），所有直接依賴的服務須於**7 個日曆天內**完成升級並重新部署；Release Notes 須以 `[SECURITY]` 前綴標示，並搭配人工追蹤清單逐服務勾選完成。查詢共用套件升級相關問題時，不要套用一般版本更新「各自決定」的彈性。
- §5 待決議事項 4 項已於 2026-09-10（v0.4）全數解決，不要以為還有未決項目：高權限帳號（`SuperAdmin`/`PlatformSupportStaff`）2FA 定案**強制、採 TOTP**（非簡訊，理由與簡訊驗證現階段不做一致）；結構化 log 集中收集定案 **Grafana Loki + Promtail**（非 ELK/Seq，單機部署下更輕量）；CSP 規則逐服務盤點完成（storefront `form-action` 放行金流商網域、`img-src` 放行 Media 儲存後端、前端一律 `connect-src` 僅同源、`script-src`/`style-src` 不允許 `unsafe-inline`）；§3 服務身分 JWT 定案**不需要快取**（純本機 HMAC-SHA256 簽章，效期 5 分鐘）。

## 修改這份文件時
- 依 `spec-add-changelog-entry` Skill 的步驟新增異動紀錄（版號/日期/作者/說明），不要靜默修改內容。
- 解決或新增「待決議事項」清單項目時，依 `spec-resolve-open-item` Skill 的做法（劃刪除線＋註記，不要直接刪除該行）。
- 若此文件的異動影響到其他文件的索引或交叉引用，依 `spec-check-cross-references` Skill 檢查並同步。
