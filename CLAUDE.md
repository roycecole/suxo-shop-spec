# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

`suxo-shop-spec` is a **specification-only repository** — it contains nothing but the `docs/` folder (17 Markdown files, written in Traditional Chinese). There is no source code, no build tooling, no package manifests, and no `.git` history in this checkout. **There are no build/lint/test commands to run here.** Do not invent them.

The docs describe (and, per their own narrative, track the implementation status of) "Suxo Shop": a white-label, multi-vendor e-commerce platform built by a system vendor (拾夜科技有限公司 / ShyeTech) and deployed independently per client. Several docs (especially `08-roadmap-and-gaps.md` and `16-microservices-split.md`) reference an actual codebase — file paths like `src/SuxoShop.Api`, `src/SuxoShop.Application/Modules/Orders/CheckoutService.cs`, `apps/storefront`, `apps/admin`, `src/SuxoShop.OpenApiGateway` — and even commit hashes and test counts. **That codebase does not exist in this repository.** Treat any such reference as describing a sibling/downstream implementation repo, not something to locate or edit here. If a task asks you to "implement" something from these docs, check with the user whether they mean editing the spec itself or whether they expect a separate codebase to exist.

Work in this repo is almost always: reading, writing, or restructuring the Markdown specs themselves.

## Document set and how it fits together

All docs live in `docs/` and are numbered `00`–`16`. Start from the index in `docs/00-project-overview.md` §5 for the authoritative list, but roughly:

| File | Content |
|---|---|
| `00-project-overview.md` | Vision, roles, tech stack, WooCommerce concept mapping, doc index |
| `01-architecture.md` | Logical architecture, modular monolith design, auth, caching, background jobs, LINE OA integration |
| `02-data-model.md` | Core entities/fields (reference for EF Core modeling, not final DDL) |
| `03-storefront-requirements.md` | Buyer-facing storefront requirements |
| `04-seller-requirements.md` | Seller/vendor back-office requirements |
| `05-admin-requirements.md` | Platform admin back-office requirements |
| `06-design-spec.md` | Design/UX spec |
| `07-non-functional-requirements.md` | Performance/security/compliance requirements |
| `08-roadmap-and-gaps.md` | **Living** gap analysis — implementation progress + known gaps, updated in place |
| `09-maintenance-and-support.md` | System-vendor maintenance model, multi-client version rollout, SLA, subscription pricing framework |
| `10-development-setup.md` | Dev environment setup for the (separate) implementation repo |
| `11-payments-and-open-api.md` | Payment gateway integration + public Open API |
| `12-frontend-and-storage.md` | Frontend architecture, SEO, file storage |
| `13-requirements-changelog.md` | **Append-only** log of requirement changes (R-001, R-002, ...) |
| `14-design-system.md` | Design tokens: colors, type scale, spacing |
| `15-data-retention.md` | Data retention/deletion policy |
| `16-microservices-split.md` | Microservices split specification (specced, partially piloted) |

Cross-references between docs use relative Markdown links (`[09-maintenance-and-support.md](09-maintenance-and-support.md)`) — keep these valid when renaming or moving files.

## Conventions to follow when editing these docs

- **Every doc opens with an 異動紀錄 (changelog) table** (`版本` / `日期` / `作者` / `說明`). Any substantive edit to a doc must bump its version and add a row — don't silently edit content without logging it.
- **`docs/13-requirements-changelog.md` is append-only.** Each new requirement/decision gets a new `R-xxx` entry (sequential); existing entries are never rewritten, only referenced. Each entry records the *original ask* (not just the conclusion), the decision, what shipped, and status (`已完成` / `部分完成` / `保留` / `已取消`). If a "保留" (deferred) item exists, state precisely what was deferred (data model only? endpoint only?).
- **`docs/08-roadmap-and-gaps.md` is a living document, not a changelog** — it gets edited/re-organized in place as things move from "gap" to "done" or as facts turn out to be stale (see its §0 "已完成/尚未實作" split). If you learn something in these docs is out of date relative to a referenced implementation, correct it here rather than leaving stale status.
- **Open items are tracked as `- [ ] ...` checklists** at the end of most docs (待決議事項); resolved ones get struck through with `~~...~~` and a note on what was actually decided, not deleted.
- Docs are written in **Traditional Chinese (zh-TW)**; match that unless the user asks otherwise.
- Prefer tables over prose for structured facts (entities/fields, roles, tech stack, SLAs) — this is the dominant style throughout.

## Key decisions already locked in (don't relitigate without cause)

These are settled in the docs and downstream sections depend on them — flag it if a new request seems to contradict one, rather than silently overriding it:

- **White-label, per-client deployment — not multi-tenant SaaS.** Each client gets its own environment + database. This is the premise behind `09-maintenance-and-support.md` and explicitly carried forward into the microservices split (`16-microservices-split.md` §0): splitting into services does **not** mean introducing cross-tenant data isolation.
- **Single monorepo for the core**, customization via config/feature flags/plugin points — never per-client source forks (`09-maintenance-and-support.md` §2, reaffirmed in `13-requirements-changelog.md` R-006).
- **Backend is pure API**, shared by three frontends (storefront/vendor/admin) differentiated only by JWT role — no frontend-specific coupling in the backend (`01-architecture.md` §10).
- **Frontend split**: `apps/storefront` (Next.js, SSR/SSG for SEO) + `apps/admin` (Vite SPA), one repo, not two (`13-requirements-changelog.md` R-006).
- **Modular monolith first**, organized as Bounded Contexts with no direct cross-module table access; a microservices split was specced and one service (Open API Gateway) has been piloted, but Orders/Payments (the highest-coupling area, requiring a Saga rewrite) has not been touched (`16-microservices-split.md`).
- **Sync REST, no message queue**, for the Saga/cross-service calls — deliberately chosen over event-driven given expected single-client traffic scale (`16-microservices-split.md` §4–5).
- **Subscription pricing**: tiered (入門/成長/企業) + hybrid (fixed monthly + overage on GMV), not a single flat rate or pure GMV cut (`09-maintenance-and-support.md` §9.1, `13-requirements-changelog.md` R-011/R-012). Concrete pricing numbers are explicitly out of scope until finance decides.
- System vendor's formal name is **拾夜科技有限公司 (ShyeTech)** — use this, not a generic "system vendor," when the docs need it (`00-project-overview.md` §1.1).
