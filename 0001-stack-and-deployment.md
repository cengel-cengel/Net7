# ADR 0001 — Stack and Deployment Topology

| Status | **Accepted** |
|---|---|
| Date | 2026-05-07 |
| Decided by | Human (System Architect) |
| Authors | Human + Claude (chat) |

---

## 1. Context

- Project: AOT digital procurement platform (not a classic webshop — no payment, no consumer checkout).
- ERP **TopM NET7** is the single source of truth (products, prices, contracts, stock, documents, batches, orders).
- `AGENTS.md` mandates: strict ERP isolation, strongly typed contracts, Clean Architecture, DDD, separation of concerns.
- Existing infrastructure already owned by AOT: **GitHub**, **Vercel**, **Railway**.
- Development workflow is currently mobile-first via Claude Code in the Claude mobile app, with PR-based delivery.

This ADR locks in the **stack** and the **deployment topology**. It does not yet decide auth, queues, or cache (deferred to later ADRs).

---

## 2. Decision

### 2.1 Frontend
- **Next.js (App Router)** + **TypeScript strict**.
- Deployed on **Vercel**.
- Preview deployments per PR are mandatory (enables mobile review workflow).

### 2.2 Backend
- **NestJS** + **TypeScript strict**.
- Deployed on **Railway** as a long-running service.
- Hosts: ERP adapters, sync workers, RFQ workflow engine, document services, queue consumers, cron.

### 2.3 Database
- **PostgreSQL on Railway**.
- Owns: sessions, users, RFQ state, sample request state, audit logs, cached projections of ERP data where explicitly justified.
- **Never** owns: products, prices, contracts, stock, batch data — those stay in TopM NET7.

### 2.4 Repository layout — Monorepo with Turborepo

```
/
├─ apps/
│  ├─ web/             # Next.js — deployed to Vercel
│  └─ api/             # NestJS — deployed to Railway
├─ packages/
│  ├─ contracts/       # shared API contracts (Zod schemas, DTOs)
│  ├─ domain/          # pure domain types (if shared between web + api)
│  └─ config/          # shared tsconfig, eslint, prettier
├─ docs/
│  ├─ adr/             # architecture decision records
│  ├─ business-discovery.md
│  └─ erp-discovery.md
├─ AGENTS.md
├─ CLAUDE.md
├─ ROADMAP.md
├─ turbo.json
└─ package.json
```

Reasoning for monorepo:
- `AGENTS.md` requires *strongly typed contracts* between layers. A shared `packages/contracts` package eliminates type drift between Next.js and NestJS.
- Atomic changes: one PR can update API contract + producer (NestJS) + consumer (Next.js). With two repos this becomes coordination overhead and breaks the "small commits" rule.
- Vercel and Turborepo integrate natively. Railway can be pointed at the `apps/api` workspace.
- Single CI pipeline → type errors caught across the entire system in one run.

### 2.5 ERP adapter placement (binding)
- All ERP I/O lives in `apps/api/src/infrastructure/erp/adapters/`.
- `apps/web` never imports from `apps/api/src/infrastructure/**`.
- `apps/web` only consumes the **public API contract** from `packages/contracts`.
- This is enforced both by folder boundaries and by Turborepo workspace dependencies.

---

## 3. Alternatives considered

### Frontend
- **Remix / SvelteKit / SolidStart** — rejected. Vercel is Next.js-optimized; SEO + dashboard profile is squarely the Next.js sweet spot; team velocity advantage is significant.

### Backend
- **Fastify / Hono on Railway** — rejected. Leaner runtime but provides less structural discipline. `AGENTS.md` mandates explicit architecture, DI, modular services. NestJS provides DI, modules, guards, pipes, interceptors out of the box and aligns directly with the engineering principles.
- **Next.js API Routes / Server Actions only (no separate backend)** — rejected. ERP integration requires long-running, retry-able, queueable workers. Vercel Serverless cannot host that reliably (timeouts, statelessness). Forcing ERP logic into Vercel Functions would violate the ERP-isolation principle and create a critical reliability risk.

### Repository layout
- **Two separate repos (web + api)** — rejected. Forces duplicated types or a published shared package. Adds ceremony for a small team. Cross-cutting changes require coordinated PRs across two repos. AGENTS.md's "strongly typed contracts" mandate is harder to enforce.

### Database
- **Vercel Postgres / Neon** — considered. Both are credible. Railway Postgres chosen because the backend already runs on Railway → fewer network hops, simpler credential management, single-vendor for stateful infrastructure. Revisitable if Railway Postgres proves insufficient at scale.

---

## 4. Consequences

### 4.1 Positive
- ERP isolation enforced at **infrastructure level**, not just code-level discipline.
- Shared contracts via Turborepo eliminate type drift between frontend and backend.
- Mobile workflow viable: PR → Vercel preview → mobile review → merge.
- Each layer scales independently. ERP-heavy workload on Railway, edge-cached UI on Vercel.

### 4.2 Negative / Risks
- **Two deployment targets** (Vercel + Railway) = two failure surfaces, two sets of secrets, two dashboards to monitor.
- **Inter-service network latency**: Vercel ↔ Railway calls cross the public internet. Needs co-located regions (e.g., Vercel `fra1` + Railway EU) for latency and GDPR.
- **Turborepo configuration** must be set up correctly (pipeline, caching, workspace deps). Mistakes here cause slow builds or broken type-checking.
- **Service-to-service auth** between Next.js and NestJS is a new attack surface — must be designed in ADR 0002.

### 4.3 Open questions (to be answered before Sprint 3 implementation)
1. **Auth model**: where does the session live? Cookie issued by Next.js, JWT issued by NestJS, or hybrid? → ADR 0002 (Sprint 4).
2. **Service-to-service auth** between Next.js and NestJS (signed requests, internal JWT, or shared secret?). → ADR 0002.
3. **Region selection**: Vercel + Railway must be co-located in EU for latency and data-residency. To confirm.
4. **Queue / worker stack**: BullMQ + Redis on Railway vs Inngest vs Railway native cron. → ADR 0003 (Sprint 2 — ERP boundary design).
5. **GitHub repo connection status**: Are Vercel and Railway already connected to the GitHub repo, or only the accounts exist? Needed before first deployable PR.

---

## 5. Approval

- [x] Approved by: **Human (System Architect)**
- Date: **2026-05-07**

This ADR is now binding. Subsequent ADRs may extend or supersede it but must reference it explicitly.

---

## 6. Related documents

- `AGENTS.md` — engineering charter
- `CLAUDE.md` — Claude Code operating rules (§4 reflects the monorepo layout decided in this ADR)
- `ROADMAP.md` — delivery plan
