# ADR 0009 — Technology Stack (Proposed, Pending Discovery)

| Status     | **Proposed — Pending Discovery + Team-Capability validation** |
|------------|---------------------------------------|
| Date       | 2026-05-11                            |
| Decided by | Human (System Architect) — pending AOT-IT + Team confirmation |
| Authors    | Human + Claude (chat) + Claude Code   |

**Important:** This ADR is **NOT Accepted**. It captures a recommended default stack to enable Phase-0 planning, but the decision is gated on the Discovery and Team-Capability outcomes listed in §5. The stack remains reversible until Phase-0 spike completion.

## 1. Context

Sprint-2 Foundation produced 4 Accepted ADRs (0005 Strangler-Fig, 0006 Modular Monolith, 0007 Content Strategy, 0008 Payment Deferred) and the constitutional digital-revenue-platform.md strategy. None of these committed to a specific technology stack — by design, per strategic-foundation §13.3 "Tech-stack lock-in: validate with team skills" and OQ-062 (hosting strategy, AOT-IT-owned, Open — see oq-master-sprint-2-extension).

Three forces now require a stack proposal:

1. **Phase-0 implementation can't start without a stack.** ADR-0005 Strangler-Fig Phase-0 (foundation infrastructure) needs concrete adapter-runtime, event-bus tooling, persistence layer.

2. **An external implementation-prompt** (AOT.de Full-Stack B2B Commerce + SEO + ERP, dated 2026-05-11) was surfaced that proposed a specific stack: **Vercel + Railway + Next.js**. This proposal is captured here as a candidate default, NOT adopted as-is per Strategy-Drift-Loop discipline.

3. **Discovery questions remain open** that must shape the final decision:
   - OQ-062 (hosting strategy — cloud provider / on-premise / hybrid): AOT-IT-owned, Open. (Migrated from strategic-foundation §12 OQ-59 in 2026-05-11 housekeeping.)
   - Team-capability: which languages/frameworks does the V1 team have experience in?
   - AOT-IT preferences (existing infrastructure, security posture, vendor relationships)

This ADR records a **proposed default** so Phase-0 planning can proceed in parallel with Discovery, with explicit gates before commitment.

## 2. Proposed Decision

### 2.1 Frontend — Next.js on Vercel

**Choice:** Next.js (App Router, TypeScript) deployed on Vercel.

**Rationale:**
- Strong SSR/SSG/ISR support, directly serves ADR-0005 Phase 1 (public catalog SEO) requirements
- Industry-standard for B2B SEO-driven sites; large talent pool
- Vercel managed edge + CDN minimizes operational burden
- App Router supports both server-rendered public-tier and SPA-style authenticated-tier in one codebase, supporting digital-revenue-platform.md Principle 1 (unified UX)

**Trade-off accepted:** Vercel lock-in for hosting + edge functions. Mitigation: Next.js itself is portable to other deployment targets (self-host, AWS Amplify, Netlify) with effort.

### 2.2 Backend Runtime — Node.js / TypeScript on Railway

**Choice:** Node.js (Long-Term Support release, TypeScript strict) deployed on Railway.

**Rationale:**
- Same language as frontend = code-sharing opportunity (domain types, validation logic)
- Modular Monolith (ADR-0006) ships as single deployment unit on Railway
- Railway provides managed Postgres, Redis, deployment pipelines with minimal ops overhead
- Talent pool overlaps with frontend (single skillset)

**Trade-off accepted:** Node.js has weaker CPU-bound performance than Go/Rust. Mitigation: pricing computation and search are I/O-bound, not CPU-bound; performance budget acceptable for V1 load.

### 2.3 Database — Postgres on Railway

**Choice:** Postgres (managed by Railway) for read-models and platform-owned state.

**Rationale:**
- Strong relational + JSON hybrid suitable for Catalog read-models and event-projection state
- Sprint-1 ADR-0006 Modular Monolith §4.3 specified one DB with logical per-module schemas — Postgres handles this natively
- Mature, well-understood, broad tooling ecosystem
- Free upgrade path to dedicated/multi-region in V2

**Trade-off accepted:** Postgres is general-purpose, not search-optimized. Mitigation: Search index lives in a separate system (see §2.6).

### 2.4 Hosting Architecture

| Component | Hosted On | Why |
|-----------|-----------|-----|
| Public frontend (SSR/SSG/ISR) | Vercel | Edge-CDN, automatic ISR, optimized for Next.js |
| Authenticated B2B frontend | Vercel (same app, different routes) | Per ADR-0006 unified deployment |
| Backend Modular Monolith | Railway | Managed Node.js + Postgres |
| Event bus (V1) | In-process within monolith | Per ADR-0006 §2.4 |
| CDN for static assets + caching | Vercel built-in | Adequate for V1 |
| NET7 ERP | AOT existing infrastructure | Unchanged per ADR-0005 Principle 2 (NET7 SoR-only) |

### 2.5 Authentication & Identity Provider

**Status:** **Deferred** to separate ADR (likely ADR-0010). Proposed default: **managed IdP** (Auth0 or AWS Cognito or Keycloak-SaaS). Build-own IdP rejected at proposal-level per strategic-foundation §6.2 Layer 5.

Final vendor selection requires:
- Cost projection at V1 user-volume estimate
- Multi-tenancy capabilities (for V2 account-hierarchies)
- SAML/OIDC support (for V2 SSO)

### 2.6 Search

**Status:** **Deferred** to separate ADR. Strategic-foundation §6.2 Layer 3 proposed Algolia as managed default; not adopted. Re-evaluation required against V1 catalog scope (Hero portfolio per digital-revenue-platform.md §5 is far smaller than full ~800 product catalog — may not require dedicated search engine for V1).

// ASSUMPTION: V1 Hero-portfolio of 30-80 products may be served by Postgres full-text search adequately. Re-evaluate when content scope expands toward V2 long-tail.

### 2.7 Status & Gates Before "Accepted"

This proposal becomes **Accepted** only after ALL of the following gates pass:

- **Gate G1 — OQ-062 (Hosting Strategy) closed:** AOT-IT confirms cloud-acceptable + Vercel/Railway specifically OR mandates alternative
- **Gate G2 — Team-Capability assessment:** confirmation that V1 team (internal + any contractors) has Next.js + Node.js + Postgres experience OR commitment to ramp-up
- **Gate G3 — Cost projection accepted:** Vercel + Railway pricing at V1 + estimated V2 scale within AOT budget envelope (per OQ-060 strategic-foundation §12, "Budget V1")
- **Gate G4 — Phase-0 spike confirms feasibility:** NET7 adapter spike (per strategic-foundation §13.3) succeeds on this stack

Until ALL gates pass, this remains **Proposed** and reversible.

## 3. Alternatives Considered

### Alt 1 — Cloud-Native (AWS / GCP / Azure)

**Approach:** Build on a single hyperscaler. Frontend on CloudFront/Amplify, backend on Fargate/CloudRun/AppService, database on RDS/CloudSQL.

**Trade-offs:**
- Pro: maximum flexibility, future-proof for scale, single-vendor billing
- Con: significantly higher operational burden for small team; Phase-0 setup time substantial; less developer-experience polish than Vercel for Next.js

**Status:** Not rejected — viable alternative if AOT-IT mandates a specific cloud provider via OQ-062.

### Alt 2 — Self-Hosted / On-Premise (alongside NET7)

**Approach:** Deploy frontend + backend on AOT-owned hardware or rented colocation, near NET7.

**Trade-offs:**
- Pro: low NET7-adapter latency, single security perimeter, AOT-IT-aligned
- Con: significant ops investment, no edge-CDN for global SEO, scaling requires hardware procurement
- Conflicts with strategic-foundation §10 capacity targets (regional distribution, edge-cache)

**Status:** Not rejected for backend services if AOT-IT mandates. Frontend SSR-tier still benefits from a CDN — would require Cloudflare or similar in front of self-hosted origin.

### Alt 3 — Mixed (Frontend on Vercel, Backend on AOT-IT-Managed)

**Approach:** Public frontend on Vercel (SEO + edge), backend deployed to AOT-IT-managed infrastructure (Railway, AWS, or on-prem).

**Trade-offs:**
- Pro: best of both — edge-optimized public-tier with AOT-controlled backend
- Con: split-ownership operationally; two distinct deployment pipelines
- This is in fact the most likely outcome if OQ-062 mandates AOT-IT-control of backend

**Status:** Recommended fallback if Alt 0 (full Vercel + Railway) is rejected.

### Alt 4 — Different Frontend Framework (SvelteKit / Astro / Remix)

**Approach:** SvelteKit (smaller bundle, fast), Astro (content-focused, partial-hydration), or Remix (Next.js-similar).

**Trade-offs:**
- Pro: technical merit, sometimes better perf per-feature
- Con: smaller talent pool, less battle-tested for B2B-SEO-commerce specifically, less ecosystem mass
- Astro could be a strong V1 fit for Hero-portfolio-static-pages, but transitions to authenticated B2B-tier weaker

**Status:** Rejected for V1 default. Re-evaluate per team-skill input in Gate G2.

### Alt 5 — Different Backend Language (Go / Python / Rust)

**Approach:** Go for performance + concurrency, Python for ecosystem (esp. data tooling), Rust for performance + safety.

**Trade-offs:**
- Pro: language strengths per domain
- Con: language-fragmentation across frontend/backend (different talent pool); less code-sharing
- Rust learning-curve cost not justified for I/O-bound workloads

**Status:** Rejected for V1 default unless team-skill input indicates strong preference.

## 4. Consequences

### 4.1 If Adopted (post-gates)

- Time-to-value: fastest path to V1 launch with managed services + framework defaults
- Operational burden: minimized — Vercel + Railway absorb deploy/scale/monitoring concerns
- Talent pool: largest possible for Next.js + Node.js
- Code-sharing: single TypeScript codebase across frontend + backend
- Migration cost from this stack to alternatives (e.g., self-hosted in V2): moderate — Next.js portable, Node.js portable, Postgres trivially portable
- Lock-in risk: present but bounded; data exit strategies exist for both Vercel and Railway

### 4.2 Risks

- **Vendor risk:** Vercel and Railway are growth-stage companies. Pricing model changes, acquisitions, or service degradations could force migration. Mitigation: keep code framework-portable, avoid platform-specific APIs where alternatives exist.
- **Cost-at-scale:** Vercel + Railway predictable at small scale, can grow non-linearly at large scale. Mitigation: monitor monthly cost trajectory; V2 evaluation includes self-host break-even analysis.
- **Edge-function limitations:** Vercel edge functions have memory + timeout constraints. Mitigation: heavy work in Railway backend, not edge.
- **Railway maturity:** Less mature than AWS — outage history exists. Mitigation: maintain operational runbook; Phase-0 evaluates uptime SLA.
- **AOT-IT misalignment:** if AOT-IT prefers internal infrastructure, this stack may be rejected outright at Gate G1. Fallback: Alt 3 (mixed) or Alt 1 (single cloud).

### 4.3 Reversibility

This stack is **reversible** until Phase-0 spike completion. Concrete reversibility points:
- Before any production deployment: trivially reversible (no infrastructure dependency yet)
- After Phase-0 spike: moderate cost (adapter code, Postgres schema portable; deployment configs vendor-specific)
- After V1 launch: significant cost (operational tooling, monitoring, runbooks tied to stack)

The principle: this ADR creates a default for planning purposes. Real lock-in begins at production deployment, not before.

## 5. Approval

**Status: Proposed.**

Decided by: Human (System Architect, Carlos) — proposal only, not Accepted.
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Phase-2 deliverable.

**Status transitions:**
- Currently: Proposed
- Upgrade to Accepted: requires ALL 4 gates (G1–G4 per §2.7) to pass
- Downgrade to Superseded: requires successor ADR if Discovery mandates different stack
- This ADR does NOT block Phase-0 implementation planning; it enables it.

## 6. Related Documents

- /AGENTS.md (engineering charter — modular architecture, ERP isolation mandate)
- /docs/strategy/digital-revenue-platform.md (Principle 1 unified UX served by Next.js single-codebase; Principle 2 NET7 SoR-only served by Railway-hosted adapters)
- /docs/adr/0005-strangler-fig.md (Phase-0 infrastructure decisions live in this ADR)
- /docs/adr/0006-modular-monolith.md (single deployment unit served by Railway; 8 modules in one Node.js codebase)
- /docs/adr/0007-content-strategy.md (sanitization adapter runs in `catalog` module on Node.js backend)
- /docs/adr/0008-payment-strategy.md (no PSP integration in V1 means no PCI-DSS infrastructure constraints to satisfy now)
- /docs/architecture/strategic-foundation.md §13.3 (Tech-stack deferral that this ADR partially closes)
- /docs/architecture/strategic-foundation.md §12 (Hosting OQ originated here as deliberation-OQ-59; migrated to OQ-062 in extension on 2026-05-11; §12 now marked as historical deliberation namespace per housekeeping)
- /docs/domain/open-questions-master.md (Sprint-1 OQs — none directly blocking)
- /docs/domain/open-questions-master-sprint-2-extension.md (Sprint-2 OQs — see OQ-062 for hosting strategy. OQ-numbering namespace clarified in 2026-05-11 housekeeping.)
- /docs/discovery/aot-discovery-phase-0-brief.md (Discovery format that Gate G1 conversation should follow)
- External input: AOT.de Full-Stack B2B Commerce + SEO + ERP Integration system-prompt (2026-05-11) — surfaced Vercel/Railway/Next.js candidate; not adopted as-is, captured here with explicit gating
