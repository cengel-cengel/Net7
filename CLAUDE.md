# CLAUDE.md
# Claude Code Operating Rules — AOT Procurement Platform

This file is read by Claude Code (CLI) at the start of every session.
For the full engineering charter see `AGENTS.md`.
For the delivery plan see `ROADMAP.md`.

---

## 1. Project context

- Company: **AOT** — B2B supplier for organic raw materials (cosmetic + food ingredients).
- Platform: **Digital procurement portal**. NOT a classic webshop. No payment. No consumer checkout.
- Core flows: sample requests, RFQs, procurement workflows, account-based customer dashboards.
- ERP: **TopM NET7** = single source of truth for products, prices, contracts, stock, documents, orders, batches.
- Primary user: strategic procurement buyer.

---

## 2. Hard rules (never violate)

1. **NEVER** duplicate ERP business logic in this codebase. ERP is authoritative.
2. **NEVER** call ERP directly from UI, API routes, or domain code. Go through `infrastructure/erp/adapters/*`.
3. **NEVER** put pricing, stock, or contract logic in the frontend.
4. **NEVER** bypass typed contracts. TypeScript strict mode is mandatory on every boundary.
5. **NEVER** commit secrets, ERP credentials, or customer data.
6. **NEVER** generate huge monolithic implementations in one step. Work in small isolated modules.
7. **NEVER** invent ERP endpoints, fields, or behaviors. If undocumented → mark `// ASSUMPTION:` and ask.

---

## 3. Workflow per task

### Step 1 — Understand
- Read related domain/adapter files before changing them.
- If requirements are ambiguous, **stop and ask**. Do not guess ERP behavior.
- Identify ERP boundary, ownership, risks before touching code.

### Step 2 — Plan (for non-trivial tasks)
- Non-trivial = >1 file changed, new module, ERP boundary touched, or architecture impact.
- Output a plan first: folder layout, interfaces, dependencies, risks, assumptions.
- **Wait for human approval** before implementing.
- Trivial tasks (single-file refactor, typo, lint fix) may proceed directly.

### Step 3 — Implement
- One module per step. One responsibility per module.
- Strong typing on every public boundary.
- Composition over inheritance. Pure functions where possible. DI over global state.
- Explicit error handling. Never swallow errors.
- No magic strings — use enums or `as const`.

### Step 4 — Verify
- Run lint, typecheck, tests for affected modules.
- For ERP adapters: include retry, idempotency, and graceful-failure tests.
- Validate edge cases (ERP unavailable, partial data, timeout).

### Step 5 — Document
- Every new module: header comment with **Purpose / Ownership / Dependencies / Risks / Extension points**.
- Every new ERP adapter: document **ERP assumptions / Retry behavior / Failure behavior / Sync strategy**.
- Update relevant README or domain doc when public contracts change.

---

## 4. Folder conventions

Repository is a **Turborepo monorepo** (per ADR 0001).

```
/
├─ apps/
│  ├─ web/                          # Next.js — deployed to Vercel
│  │  └─ src/
│  │     ├─ app/                    # App Router routes
│  │     ├─ components/             # UI only — no business logic
│  │     └─ lib/                    # client-side helpers, no ERP I/O
│  └─ api/                          # NestJS — deployed to Railway
│     └─ src/
│        ├─ domain/                 # entities, value objects — no I/O
│        ├─ application/            # use cases, orchestration
│        ├─ infrastructure/
│        │  ├─ erp/
│        │  │  ├─ adapters/         # ALL ERP I/O lives here
│        │  │  └─ contracts/        # ERP DTOs and mapping
│        │  ├─ db/
│        │  └─ auth/
│        └─ http/                   # NestJS controllers — thin
├─ packages/
│  ├─ contracts/                    # shared API contracts (Zod schemas, DTOs)
│  ├─ domain/                       # shared domain types (only if used by both apps)
│  └─ config/                       # shared tsconfig, eslint, prettier
├─ docs/
│  └─ adr/                          # architecture decision records
├─ AGENTS.md
├─ CLAUDE.md
├─ ROADMAP.md
├─ turbo.json
└─ package.json
```

Strict separation (boundary rules — non-negotiable):

- `apps/api/src/domain/` knows nothing about ERP, DB, or HTTP.
- `apps/api/src/application/` orchestrates domain + infrastructure via interfaces.
- `apps/api/src/infrastructure/` is the **only** place where ERP, DB, or external I/O lives.
- `apps/api/src/http/` (controllers) never imports from `infrastructure/` directly — only via application use cases.
- `apps/web/` **never** imports from `apps/api/src/**`. The frontend is a consumer of the public API contract only.
- The contract between `apps/web` and `apps/api` lives **exclusively** in `packages/contracts`.
- ERP code (TopM NET7) lives **only** in `apps/api/src/infrastructure/erp/`. Nowhere else. Ever.

---

## 5. Naming

- Entities: `PascalCase` — `Product`, `RFQ`, `SampleRequest`, `Customer`.
- Use cases: `VerbNoun` — `SubmitSampleRequest`, `FetchProductDocuments`, `CreateRFQ`.
- Adapters: `Net7<Domain>Adapter` — `Net7ProductAdapter`, `Net7CustomerAdapter`.
- Interfaces (ports): `<Domain>Port` or `<Domain>Repository` — `ProductRepository`.
- No `utils.ts` dump files. Group by feature.

---

## 6. Commit style

```
<type>(<scope>): <short description>
```

Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`.

Examples:
- `feat(product-search): add CAS number search adapter`
- `fix(erp-sync): handle Net7 retry timeout`
- `refactor(rfq): extract approval state machine`
- `docs(adapter): document Net7 customer mapping assumptions`

Before every commit, self-review against `AGENTS.md` checklist:
**Architecture / Security / Maintainability / Scalability / ERP Safety / Handover**.

---

## 7. When uncertain

- Stop and ask. Do not guess.
- Mark assumptions explicitly: `// ASSUMPTION: Net7 returns ISO date strings`.
- In responses, list assumptions and risks separately from the answer.
- Never silently introduce new dependencies without surfacing them.

---

## 8. Communication style for human replies

- The human user writes in German. Reply in German.
- Code, identifiers, and inline code comments stay in English.
- Use structured markdown: short sections, code blocks, tables when useful.
- Always include: **reasoning, tradeoffs, assumptions, risks**.
- Be concise. No filler.

---

## 9. Forbidden shortcuts

- No `any` types to silence the compiler.
- No `// @ts-ignore` without an inline justification.
- No business logic copied from ERP into the codebase to avoid an adapter call.
- No console.log in committed code — use the logger.
- No `TODO` without an owner and a ticket reference.
- No new dependencies without justification in the plan step.
