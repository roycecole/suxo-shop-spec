# 26 - 專案結構 (Project Structure)

## 異動紀錄
| 版本 | 日期 | 作者 | 說明 |
|---|---|---|---|
| v0.1 | 2026-09-08 | ordinarycas | 初版建立，回應「確認專案清單」需求，彙整 ShyeCMS 與電商平台各自的完整 repo/專案結構 |
| v0.2 | 2026-09-08 | ordinarycas | 回應「預期該專案的資料夾目錄結構」需求：補上兩個 repo 的實際資料夾樹狀圖、服務內部分層慣例、共用 .NET 函式庫落地位置；解決 `.sln` 與 `packages/api-client` 待決議事項 |
| v0.3 | 2026-09-08 | ordinarycas | **推翻 v0.2 的部分決策**：(1) React 前端一律獨立成自己的 repo，不跟後端放一起；(2) 電商平台的前台（買家）與後台（賣家）也彼此獨立成兩個 repo，避免其中一邊的原始碼/排版被另一邊的建置產物意外帶到；(3) 重新檢視「15 個服務共用一個 `.sln`」的維運風險並推翻，改為服務各自獨立、共用邏輯改用版本化套件而非專案參照。因應 repo 數量增加，新增 `ecommerce-deploy` 部署設定 repo 統整多個 repo 建置出的映像檔 |
| v0.4 | 2026-09-08 | ordinarycas | [10-gap-analysis.md](10-gap-analysis.md) 第八輪複查發現：§3.1 `services/identity/` 的範例底下複製貼上時忘了改，命名空間誤植為 `SuxoShop.Catalog.*`，已訂正為 `SuxoShop.Identity.*` |
| v0.5 | 2026-09-09 | ordinarycas | §3.3 `ecommerce-admin/features/` 補上 promotions/shipping/reviews 三個資料夾，同步 [08-vendor-admin-requirements.md](08-vendor-admin-requirements.md) v0.5 已新增的三個賣家後台功能（實作 repo 已依 08 建了 10 個 features 資料夾，本文件範本至此對齊） |
| v0.6 | 2026-09-09 | ordinarycas | repo 更名 `ecommerce-deploy`→`ecommerce-launch`，呼應實際 checkout 的資料夾命名；§2.2 `shyecms-admin/features/` 補上 `staff/` 模組並確認先前的猜測樹狀圖、§7 標記 ShyeCMS 前端需求規格待決議項已解決——皆同步新增的 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md) |

> 本文件回答「總共會有哪些 repo、各自資料夾長什麼樣子」。

## 1. 總覽：六個 Repo

| Repo | 內容 | 部署方式 |
|---|---|---|
| `shyecms-api` | ShyeCMS 後端（C# .NET 10 單體） | 拾夜科技單一實例 |
| `shyecms-admin` | ShyeCMS 前端（React SPA） | 拾夜科技單一實例 |
| `ecommerce-services` | 電商平台 15 個微服務 + 共用 .NET 函式庫 | 逐客戶部署（映像檔） |
| `ecommerce-storefront` | 電商平台前台（Next.js，買家使用） | 逐客戶部署（映像檔） |
| `ecommerce-admin` | 電商平台後台（Vite SPA，賣家使用） | 逐客戶部署（映像檔） |
| `ecommerce-launch` | 電商平台部署設定（docker-compose + 版本標籤，見 §4） | 逐客戶 clone 到各自 VPS |

**為什麼從原本的「2 個 repo」變成「6 個 repo」**：v0.1/v0.2 曾把前後端放在同一個 repo 裡（如 `ShyeCMS Repo` 內同時放 `shyecms-api` 和 `shyecms-admin`；電商平台的 `apps/storefront`、`apps/admin`、`services/*` 全部塞進一個 monorepo）。使用者提出兩點理由後推翻此設計：
1. **React 前端本來就該獨立**：前端團隊不需要 checkout 後端程式碼，反之亦然，職責邊界應該用 repo 邊界具體落實，而不是只靠資料夾分開。
2. **買家前台跟賣家後台更要分開**：兩者的使用者完全不同（消費者 vs 賣家），放在同一個 repo/同一次建置流程裡，多一分「後台程式碼或排版意外出現在前台建置產物」的風險（如共用的建置設定、環境變數、或未來有人不小心在 `apps/` 底下互相 import）。分成兩個 repo 讓這件事**結構上不可能發生**，不是靠開發者自律。

## 2. ShyeCMS：2 個 Repo

### 2.1 `shyecms-api`

```
shyecms-api/
├── ShyeCMS.sln                      服務本身只有 1 個單體，.sln 在此規模沒有 §5 的耦合疑慮
├── docker-compose.yml                本機開發用：API + Postgres
├── src/
│   ├── ShyeCMS.Api/                  Controllers、Program.cs、appsettings.json、JWT 驗證設定
│   ├── ShyeCMS.Application/          Use Case：客戶建檔、訂閱調整、功能授權變更、AuditLog 寫入
│   ├── ShyeCMS.Domain/               實體：Client、StaffUser、SubscriptionPlan、
│   │                                 ClientSubscription、FeatureFlag、
│   │                                 ClientFeatureEntitlement、ClientDeployment、AuditLog
│   └── ShyeCMS.Infrastructure/       EF Core DbContext、Migrations
└── tests/
    └── ShyeCMS.Tests/
```

### 2.2 `shyecms-admin`（獨立 repo，不與 `shyecms-api` 放在一起）

```
shyecms-admin/
├── src/
│   ├── features/
│   │   ├── clients/                  客戶檔案清單/建檔/狀態變更
│   │   ├── subscriptions/            訂閱方案指派與調整
│   │   ├── entitlements/             功能授權（合約紀錄）調整
│   │   ├── deployments/              客戶部署盤點紀錄
│   │   ├── audit-log/                StaffUser 操作稽核查詢
│   │   └── staff/                    內部人員（StaffUser）管理，見 31-shyecms-frontend-requirements.md §10
│   ├── components/
│   ├── api/                          呼叫 shyecms-api 的型別化 Client（透過環境變數指向 API 網址，不寫死）
│   └── main.tsx
├── Dockerfile
└── vite.config.ts
```

> **`features/*` 已依 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md) §3 確認**（先前是依現有資料模型反推的猜測，該文件已補上完整頁面/操作流程規格）——新增 `staff/` 模組對應內部人員管理（31 §10），其餘 5 個模組維持不變。

兩個 repo 之間透過 API 溝通（`shyecms-admin` 呼叫 `shyecms-api`），**不共用程式碼、不共用建置流程**。拾夜科技只需要自己內部知道兩個服務的網址對應關係即可，因為只有一份實例，不像電商平台需要 §4 那種「多 repo 映像檔組裝」的正式流程。

## 3. 電商平台：4 個 Repo

### 3.1 `ecommerce-services`（後端，15 個微服務）

```
ecommerce-services/
├── docker-compose.yml                本機開發用：15 服務 + Postgres，不含前端
├── services/
│   ├── identity/
│   │   ├── Dockerfile
│   │   ├── src/
│   │   │   ├── SuxoShop.Identity.Api/            （以 Identity 為例，其餘服務同構）
│   │   │   ├── SuxoShop.Identity.Application/
│   │   │   ├── SuxoShop.Identity.Domain/
│   │   │   └── SuxoShop.Identity.Infrastructure/
│   │   └── tests/
│   ├── catalog/            （結構同上）
│   ├── wms/
│   ├── vendor/
│   ├── cart/
│   ├── promotions/
│   ├── order/
│   ├── payment/
│   ├── media/
│   ├── cms/
│   ├── shipping/
│   ├── analytics/
│   ├── notification/
│   ├── reviews/
│   └── gateway/
└── shared/                           跨服務共用 .NET 邏輯，發布為版本化 NuGet 套件（見 §5.2）
    ├── SuxoShop.Shared.Conventions/
    ├── SuxoShop.Shared.Markdown/
    ├── SuxoShop.Shared.Security/
    └── SuxoShop.Shared.Translation/
```

單一微服務的內部四層分層慣例（Api/Application/Domain/Infrastructure）維持不變，見 §5.1。

### 3.2 `ecommerce-storefront`（前台，買家使用，獨立 repo）

```
ecommerce-storefront/
├── Dockerfile
├── app/
│   ├── [locale]/                     i18n 路徑前綴（28-i18n.md §2）
│   │   ├── page.tsx                   首頁
│   │   ├── products/
│   │   ├── cart/
│   │   ├── checkout/
│   │   └── orders/
│   └── api/
│       └── revalidate/                商品異動時觸發 ISR 更新（06 §5）
├── components/
├── lib/                               API 呼叫（透過 26 §5.3 的 api-client 套件）、Route Handler 代理
└── next.config.js
```

### 3.3 `ecommerce-admin`（後台，賣家使用，獨立 repo，與前台完全分開）

```
ecommerce-admin/
├── Dockerfile
├── src/
│   ├── features/
│   │   ├── products/                 上架/編輯，含 Markdown 編輯器（08 §1）
│   │   ├── inventory/                 WMS 操作（入庫、批次）
│   │   ├── orders/                    訂單處理
│   │   ├── analytics/                 銷售數據，含 Lightweight Charts（22 §4）
│   │   ├── cms/                       首頁版型，含 Markdown 編輯器
│   │   ├── settings/                  StoreSettings（08 §2）
│   │   ├── woocommerce-export/        WooCommerce CSV 匯出（08 §5）
│   │   ├── promotions/                優惠券管理（08 §1 v0.5 新增）
│   │   ├── shipping/                  運費規則設定（08 §1 v0.5 新增）
│   │   └── reviews/                   評價管理（08 §1 v0.5 新增）
│   ├── components/
│   ├── i18n/                          react-i18next 語系檔（28-i18n.md §5）
│   ├── api/                           呼叫 API 的型別化 Client（見 §5.3）
│   └── main.tsx
└── vite.config.ts
```

`ecommerce-storefront` 與 `ecommerce-admin` 除了都透過 Open API Gateway 呼叫後端之外，**沒有任何共用的原始碼目錄、沒有共用的建置設定**，唯一共用的是 §5.3 的 `api-client` 版本化套件（透過 npm registry 安裝，不是檔案系統上的共用資料夾）。

### 3.4 `ecommerce-launch`（新增：部署設定，實際 clone 到客戶 VPS 的是這個）

```
ecommerce-launch/
├── docker-compose.yml                引用其他三個 repo 各自建置好的映像檔（見範例）
├── .env.example
└── README.md                         SOP：如何在新客戶 VPS 上部署、如何更新版本
```

`docker-compose.yml` 示意（版本標籤由各 repo 的 CI/CD 發布時決定）：

```yaml
services:
  storefront:
    image: registry.shyetech.com/ecommerce/storefront:2.1.0
  admin:
    image: registry.shyetech.com/ecommerce/admin:1.4.0
  catalog:
    image: registry.shyetech.com/ecommerce/catalog:1.8.2
  # ...其餘 13 個服務同構
  gateway:
    image: registry.shyetech.com/ecommerce/gateway:1.2.0
  postgres:      # 僅 Docker 內建 DB 模式需要，見 06 §6.2
    image: postgres:17
```

**為什麼需要這第 4 個 repo**：把前台/後台/15 個服務拆成 3 個獨立 repo 後，**沒有任何一個 repo 天生知道「這一整套要怎麼一起啟動」**——`ecommerce-services` 只知道自己的 15 個服務，不會、也不該知道前端的存在。`ecommerce-launch` 是唯一橫跨全部的角色：只放組裝用的 compose 設定與版本標籤，不放任何一行應用程式邏輯。更新客戶版本＝改這個 repo 裡的映像檔標籤，`git pull` + `docker compose pull && docker compose up -d`，不需要在客戶 VPS 上 clone 三份原始碼。

**每個客戶各自一份，不是一個 repo 服務所有客戶**：`ecommerce-launch` 沿用其餘電商平台 repo「每個客戶各自一份拷貝」的白牌慣例，做法是把它當**範本 repo**，新客戶上線時複製成 `ecommerce-launch-<客戶代稱>`（如 `ecommerce-launch-babaguava`），各自維護自己的映像檔版本標籤與 `.env`。這樣客戶 A 停在 `catalog:1.8.2`、客戶 B 已升到 `1.9.0`，兩份 `ecommerce-launch-*` 互不影響——不會因為拆成多 repo 反而變成「所有客戶被迫同時升級」。

**機密資訊不進版控**：`.env.example` 只列出需要哪些變數（DB 連線字串、`Jwt:SigningKey` 等），**實際的 `.env`（含真實密鑰/密碼）不 commit 進 `ecommerce-launch-<客戶代稱>`**，比照 [03-client-lifecycle.md](03-client-lifecycle.md) §3 既有的「新客戶上線 SOP」（環境建置 → 資料庫初始化 → ...）由維運人員在客戶 VPS 上手動建立，`.gitignore` 排除 `.env`。

## 4. 共用邏輯的處理方式（因應 repo 拆分調整）

### 4.1 服務內部分層慣例（不變）

每個微服務仍維持 Api/Application/Domain/Infrastructure 四層，理由同前版：團隊共用心智模型，複雜度差異用「這一層程式碼多寡」呈現，不是用「有沒有這一層」呈現。

### 4.2 `SuxoShop.Shared.*`：改為版本化 NuGet 套件，不是專案參照（回應 §6 的 `.sln` 疑慮）

原本規劃 `shared/` 資料夾內的函式庫用 `ProjectReference` 讓 15 個服務直接參照原始碼。**現在改為：`ecommerce-services` 內的 `shared/` 專案各自發布成版本化的 NuGet 套件**（推到私有 NuGet feed），15 個服務用 `PackageReference` 依版本號安裝，不是直接參照原始碼。

**為什麼改**：直接對應 §6 的疑慮——`ProjectReference` 代表任何一個共用函式庫的變更，理論上會立刻影響「同一次建置」內的所有服務，模糊了「服務可以獨立升級」的邊界。改成版本化套件後，服務 A 想升級到 `SuxoShop.Shared.Markdown 2.0.0`、服務 B 還停留在 `1.3.0`，兩者互不影響，各自的升級時程由各自決定，這才是微服務精神真正該有的樣子。

### 4.3 `api-client`：改為獨立 npm 套件，不是 monorepo 內的 `packages/` 資料夾

因為 `ecommerce-storefront` 與 `ecommerce-admin` 現在是兩個獨立 repo，不可能再共用同一個檔案系統路徑下的 `packages/api-client`。改為**獨立發布到私有 npm registry 的套件**（如 `@suxoshop/api-client`），內容由 [09-api-specification.md](09-api-specification.md) 各服務 OpenAPI 規格產生型別，兩個前端各自在 `package.json` 宣告版本並安裝，跟 4.2 的 NuGet 套件是同一個道理。

## 5. 「15 個服務放一個 `.sln` 會不會影響維運」——重新檢視後的結論

**會**，原本 v0.2 的判斷不完整，理由如下：

| 風險 | 說明 |
|---|---|
| **隱性耦合** | 一個 `.sln` 把 15 個服務攤開在同一個建置圖裡，任何人都可以隨手加一個 `ProjectReference` 直接參照另一個服務的類別（如 Order 直接 `using SuxoShop.Catalog.Domain`），繞過「服務間只能透過 API 溝通」的原則——這在多個獨立 repo 下**結構上不可能發生**（你的 repo 裡根本沒有別的服務的原始碼可以參照），但在單一 `.sln` 下只是「大家自律不要這樣做」，約束力差很多 |
| **升級節奏被綁在一起** | 共用函式庫若用 `ProjectReference`，等於每次共用邏輯變更，理論上都跟著同一次建置流動到所有服務，跟 [09-api-specification.md](09-api-specification.md) 已經確立的「各服務可獨立升級」原則互相矛盾 |
| **建置圖越滾越大** | 60+ 個專案的單一 `.sln`，本機 IDE 開啟/還原/建置的時間會隨專案數量持續變差，且這個成本會被所有開發者重複付出，不是一次性成本 |

**修正後的決策**：**不設置橫跨全部服務的單一 `.sln`**。改為：
- 每個服務自己的資料夾內可以有自己的小型 `.sln`（如 `services/catalog/Catalog.sln`，只含該服務自己的 4 個專案 + 測試），方便該服務的開發者在 IDE 內工作。
- 需要同時對照多個服務程式碼時（除錯 Saga 等跨服務情境），開發者可以在自己機器上**臨時**建立一個不進版控的本機 `.sln`，或直接用支援多資料夾工作區的編輯器（如 VS Code Workspace），不需要 repo 裡存在一個「官方」的全服務 `.sln`。
- CI/CD 對每個服務各自執行 `dotnet build`/`dotnet test`（在各自的資料夾內，不透過中央 `.sln`），這樣也天然對應「只建置有變更的服務」的效率需求（[10-gap-analysis.md](10-gap-analysis.md) §3 CI/CD 缺口的具體實作方向之一）。

## 6. 專案數量總計（因 repo 拆分更新）

| Repo | 部署單位數 |
|---|---|
| `shyecms-api` | 1 |
| `shyecms-admin` | 1 |
| `ecommerce-services` | 15 |
| `ecommerce-storefront` | 1 |
| `ecommerce-admin` | 1 |
| **總計相異部署單位** | **19**（與 v0.2 相同，只是現在分散在 6 個 repo 而非 2 個） |

`ecommerce-launch` 不含任何應用程式部署單位，只是組裝設定，不計入上表。`ecommerce-services` 內的 4 個 `SuxoShop.Shared.*` 與 `api-client` 套件同理不計入（見 §4.2、§4.3，兩者都是版本化套件，不是獨立部署的容器）。

## 7. 待決議事項
- [x] ~~ShyeCMS 的前端需求規格尚未撰寫~~——**已解決**：新增 [31-shyecms-frontend-requirements.md](31-shyecms-frontend-requirements.md)，比照 [07](07-storefront-requirements.md)/[08](08-vendor-admin-requirements.md) 的規格深度，§2.2 的 `features/*` 樹狀圖已同步更新
- [ ] 私有 NuGet feed 與私有 npm registry 的實際服務選型（如 Azure Artifacts、GitHub Packages、自架 Verdaccio/BaGet）尚未決定
- [ ] 6 個 repo 各自的 CI/CD 都尚未定案，且現在比 v0.2 的「2 個 repo 各自 CI/CD」更分散，需要一份跨 repo 的版本發布 SOP（哪個 repo 發新版後，`ecommerce-launch` 何時、由誰更新映像檔標籤）
- [ ] `ecommerce-launch` 的版本標籤更新是人工修改 YAML 後 commit，還是要做成自動化（如各 repo CI 發版後自動開 PR 更新 `ecommerce-launch`）
- [ ] `services/*/Dockerfile` 的實際內容（multi-stage build、基礎映像檔版本）尚未撰寫
