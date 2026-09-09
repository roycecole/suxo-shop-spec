# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`suxo-shop-spec` is a **specification-only repository** — it contains nothing but the `docs/` folder (31 Markdown files, written in Traditional Chinese). There is no source code, no build tooling, no package manifests. **There are no build/lint/test commands to run here.** Do not invent them.

The docs describe **two independent systems** designed by 拾夜科技有限公司 (ShyeTech):

1. **ShyeCMS** (`docs/00`–`05`) — ShyeTech's own internal system for managing its clients: client records, subscription plans, contract/feature-entitlement records, deployment inventory. Clients have no login to it.
2. **The e-commerce platform** (`docs/06`–`30`), specced using "爸芭樂" (a guava shop) as the running example — the actual white-label product sold to and run independently by each client. Backend is C# / ASP.NET Core (.NET 10) microservices, storefront is Next.js (SSG/ISR), vendor admin is a Vite SPA, database is PostgreSQL.

**These two systems have zero technical connection to each other** — no API calls, no shared credentials, no runtime dependency in either direction. This is a deliberate, explicitly-locked decision (see `01-architecture.md`), not an oversight: ShyeCMS never queries a client's platform, and a client's platform never calls ShyeCMS. Feature entitlements recorded in ShyeCMS are a *contractual/business record only* — actually enabling a feature on a client's deployment is a manual config-file edit done by ops staff at deploy time, never an automated or runtime-verified sync. Don't propose adding any such connection without flagging that it contradicts a locked decision.

Docs reference repos that **do not exist in this checkout** (e.g. `shyecms-api`, `ecommerce-services`, `ecommerce-storefront`, `ecommerce-admin`, `ecommerce-deploy` — see `26-project-structure.md`). Treat any file path or namespace mentioned in the docs (e.g. `SuxoShop.Catalog.Domain`, `ShyeCMS.Application`) as describing a sibling/downstream implementation repo, not something to locate or edit here. If a task asks you to "implement" something from these docs, check with the user whether they mean editing the spec itself or whether they expect a separate codebase to exist.

Work in this repo is almost always: reading, writing, or restructuring the Markdown specs themselves.

Note: an earlier, larger spec set (originally numbered `00`–`16`, describing a single modular-monolith platform) was deleted by the user and is **not recoverable** — `00-overview.md` §8 lists what it used to cover (full field-level data models, admin requirements, design tokens, NFRs, SLA/rollout process, dev-setup commands, payment/storage implementation detail, data-retention policy, a `13-requirements-changelog.md` R-001–R-013 history). Don't assume any of that content still exists elsewhere in this repo; it would need to be rewritten from scratch if needed.

## Document set and how it fits together

Start from `docs/00-overview.md` §6 for the authoritative index. Roughly:

**ShyeCMS (internal, non-technical-connection to the platform below):**

| File | Content |
|---|---|
| `00-overview.md` | Origin, feasibility analysis, locked architecture decisions (A–H), terminology, full doc index |
| `01-architecture.md` | Why ShyeCMS has zero connection to client platforms; how feature entitlement is manually operationalized instead |
| `02-data-model.md` | ShyeCMS entities: `Client`, `StaffUser`, `SubscriptionPlan`, `ClientSubscription`, `FeatureFlag`, `ClientFeatureEntitlement`, `ClientDeployment` (inventory only, not a live endpoint), `AuditLog` |
| `03-client-lifecycle.md` | Client onboarding → operation → termination lifecycle (manual/business process) |
| `04-feature-entitlement-and-metering.md` | **Deprecated design** — the v0.1 live-query/usage-pull mechanism, superseded by the zero-connection decision; kept for history |
| `05-scope-and-open-items.md` | Explicit exclusions and open items — **scoped to ShyeCMS only** (`00`–`05`); platform-side items are in `30` |

**E-commerce platform ("爸芭樂" case), microservices architecture:**

| File | Content |
|---|---|
| `06-ecommerce-platform-architecture.md` | Tech stack, overall architecture diagram, deployment topology (single VPS + Docker Compose), 3 DB connection modes, checkout Saga |
| `07-storefront-requirements.md` | Buyer-facing storefront requirements (guest checkout, LINE/Google login) |
| `08-vendor-admin-requirements.md` | Seller back-office requirements, incl. ShyeTech's `PlatformSupportStaff` support role, WooCommerce CSV export |
| `09-api-specification.md` | Cross-service API versioning strategy (`/api/v{n}`, per-service version numbers) and doc format (OpenAPI 3.0, Problem Details) |
| `10-gap-analysis.md` | **Living** gap analysis across the whole platform spec — edited in place, not append-only |
| `11-service-identity.md` | Identity Service: accounts, auth, JWT, address book |
| `12-service-catalog.md` | Catalog Service: product/category/variant master data, pricing, WooCommerce export |
| `13-service-wms.md` | WMS Service: inventory, batches/expiry, atomic stock deduction |
| `14-service-vendor.md` | Vendor Service: store profile, sub-accounts, commission settings |
| `15-service-cart.md` | Cart Service |
| `16-service-promotions.md` | Promotions Service: coupons |
| `17-service-order.md` | Order Service: order/sub-order state machine, checkout Saga orchestrator |
| `18-service-payment.md` | Payment Service: gateway integration, callback verification |
| `19-service-media.md` | Media Service: upload, storage backend, thumbnails, quota |
| `20-service-cms.md` | CMS Service: homepage/landing page templates |
| `21-service-shipping.md` | Shipping Service: shipping methods, rate calculation |
| `22-service-analytics.md` | Analytics Service: read-only reporting (TradingView Lightweight Charts for time series) |
| `23-service-notification.md` | Notification Service: LINE Official Account integration |
| `24-service-reviews.md` | Reviews Service |
| `25-service-gateway.md` | Open API Gateway: sole public entry point, API-key auth, routing/aggregation |
| `26-project-structure.md` | Full repo layout: 6 repos, folder trees, shared-library packaging strategy |
| `27-pwa-and-accessibility.md` | RWD/PWA (storefront + admin) and WCAG 2.1 AA (storefront only) |
| `28-i18n.md` | zh-TW (default) / EN / JA — URL routing, translation data model |
| `29-shared-service-conventions.md` | Rules **all 15 services must follow**: correlation ID, health-check endpoints, structured logging, shared Markdown-sanitization pipeline, inter-service auth, security baseline |
| `30-open-decisions-register.md` | Index of all 77 open items across every doc, top-10 prioritized — index only, edit the source doc, not this file |

Cross-references between docs use relative Markdown links — keep these valid when renaming or moving files.

## Claude Skills for this repo

`.claude/skills/` has one skill per doc (`shyecms-*`, `platform-*`, `service-*`, plus `project-structure`/`pwa-and-accessibility`/`i18n`/`shared-service-conventions`/`open-decisions-register`) for quickly loading a doc's role/sections/gotchas, and six `spec-*` workflow skills that encode the maintenance conventions below procedurally (`spec-add-changelog-entry`, `spec-resolve-open-item`, `spec-update-gap-analysis`, `spec-sync-open-decisions-register`, `spec-new-service-doc`, `spec-check-cross-references`). Prefer invoking the relevant one over re-deriving these steps from scratch.

## Conventions to follow when editing these docs

- **Every doc opens with an 異動紀錄 (changelog) table** (`版本` / `日期` / `作者` / `說明`). Any substantive edit to a doc must bump its version and add a row — don't silently edit content without logging it.
- **`docs/10-gap-analysis.md` is a living document, not a changelog** — edited/re-organized in place as gaps get resolved (struck through with `~~...~~` and a note on what resolved it, not deleted) or new ones surface.
- **`docs/30-open-decisions-register.md` is an index only** — when resolving or adding an open item, edit the source doc first; this register needs a re-scan/sync afterward. Don't treat it as the source of truth.
- **Open items are tracked as `- [ ] ...` checklists** at the end of most docs (待決議事項); resolved ones get struck through with `~~...~~` plus a note on what was actually decided, not deleted.
- Docs are written in **Traditional Chinese (zh-TW)**; match that unless the user asks otherwise.
- Prefer tables over prose for structured facts (entities/fields, service lists, tech stack); use Mermaid diagrams for architecture/ERD/sequence flows (already the dominant style in `01`, `02`, `06`, `17`).

## Key decisions already locked in (don't relitigate without cause)

- **ShyeCMS and the client e-commerce platform have zero technical connection** — no API calls, no shared credentials, no polling. Feature entitlement is a contract record in ShyeCMS; enabling it on a client's actual environment is a manual config edit by ops staff. This reversed an earlier (v0.1) design that had them talking to each other (`00-overview.md` decision C/D, `01-architecture.md`).
- **ShyeTech never ingests client business data** — no product, pricing, member, or even aggregated usage data flows from a client's platform back to ShyeTech. GMV-overage billing therefore has no data source and is an explicitly accepted open gap, not something to silently design around.
- **The e-commerce platform is greenfield microservices, not a monolith-first evolution** — 14 domain services + Open API Gateway, each with its own PostgreSQL schema, no direct cross-service table access, sync REST only (no message queue — deliberate, given expected single-client traffic scale).
- **Single VPS + Docker Compose per client deployment**, not multi-tenant SaaS — each client gets one virtual host running everything via `docker compose up -d`. DB connection is one of three interchangeable modes (Docker-internal Postgres / external managed, Supabase preferred / internal client-hosted Postgres), selected by env var, never hardcoded.
- **6 separate repos, not a monorepo**: `shyecms-api`, `shyecms-admin`, `ecommerce-services` (15 services + shared libs), `ecommerce-storefront`, `ecommerce-admin`, `ecommerce-deploy` (per-client deploy config, the only repo actually cloned onto a client VPS). Shared logic crosses repo boundaries only as versioned packages (NuGet / npm), never `ProjectReference`/monorepo path imports — this was a deliberate reversal of an earlier "shared `.sln`" design, kept precisely so services can upgrade independently.
- **Order Service is the checkout Saga orchestrator** across Cart → WMS → Promotions → Order → Payment, with synchronous REST calls and compensating actions on failure; Notification is fired asynchronously and never blocks the order.
- **Inventory ownership lives in WMS Service, not Catalog** — Catalog is product/price display only. This correction from an earlier draft is called out explicitly in `06-ecommerce-platform-architecture.md` §7 step 2 — don't reintroduce stock fields into Catalog.
- **API versioning is per-service, not platform-wide** (`/api/v{n}/...`, each service accrues its own version independently) — one service being on `v2` while another is still `v1` is expected, not a bug (`09-api-specification.md`).
- **`29-shared-service-conventions.md` binds all 15 services** — correlation ID propagation, `/health/live` + `/health/ready`, structured JSON logs, the shared Markdown→sanitized-HTML pipeline (Catalog + CMS must not each roll their own), and the two-layer internal-API auth (Docker network isolation + short-lived service-identity JWT, no mTLS). Don't let a per-service doc quietly redefine any of these.
- System vendor's formal name is **拾夜科技有限公司 (ShyeTech)** — use this, not a generic "system vendor."
