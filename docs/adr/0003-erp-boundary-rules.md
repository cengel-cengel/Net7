# ADR 0003 — ERP Boundary Rules

| Status | **Accepted** |
|---|---|
| Date | 2026-05-09 |
| Decided by | Human (System Architect) |
| Authors | Human + Claude (chat) + Claude Code |

---

## 1. Context

- `AGENTS.md` ERP Ownership Rules and ERP Integration Rules mandate that **NET7 is the single source of truth** for products, prices, contracts, stock, batch traceability, documents, and order history; the web platform owns UX, sessions, dashboards, and workflows. The constitutional framework is in `AGENTS.md`; the *adapter-level operational rules* are not.
- Sprint 1 produced five `Net7XAdapter` profiles inside the §5 sections of the five P1 aggregate sketches (`Net7ProductAdapter`, `Net7DocumentAdapter`, `Net7PricingAdapter`, `Net7SampleAdapter`, `Net7CustomerAdapter`). The adapter behaviour is specified per aggregate; without an explicit ADR there is no single source of truth for the rules that govern Web↔ERP interaction across all adapters.
- **Distinction vs ADR-0002**: ADR-0002 codifies *how we structure aggregate-modelling sketches* (the structural patterns — 7-H2 template, invariant taxonomy, state-machine notation variants, adapter profile set, self-report discipline). ADR-0003 codifies *what rules govern the Web↔ERP interaction at adapter level* (the operational rules — single source of truth, read-only by default, idempotency mandate, identity bridging boundary, failure graceful degradation, status sync mechanism, eventual consistency). Both ADRs are needed: ADR-0002 is the **HOW**, ADR-0003 is the **WHAT**.
- ADR-0003 builds on the ADR-0002 pattern foundation. Each rule below traces back to one or more ADR-0002 patterns where applicable; the explicit Pattern→Rule mapping is given in §2 sub-sections.
- Sprint 2 *ERP Boundary Design* needs explicit rule-foundation as input. Without ADR-0003 the boundary design re-derives rules ad-hoc from the five §5 Adapter sections — slow, inconsistent, and error-prone.
- This ADR is the structural anchoring of the `AGENTS.md` ERP Hard Rules as actionable adapter-level rules.

---

## 2. Decision

Adopt the seven boundary rules below as the binding contract for all `Net7XAdapter` implementations and all Web↔ERP interaction in this codebase. Rules are numbered by logical hierarchy (single-source-of-truth → read-only-default → idempotency → identity → failure → sync → consistency), not by importance.

### 2.1 Rule 1 — ERP as Single Source of Truth

- **Rule**: NET7 owns master data for: products, prices, customer contracts, stock, logistics, batch traceability, documents, order history. Web owns: UX, SEO, content, sessions, dashboards, workflows.
- **NEVER** duplicate ERP business logic in this codebase. **NEVER** move ERP pricing, stock, or contract logic into frontend or web-side services.
- **Pattern foundation**: ADR-0002 implicit (every aggregate sketch's §5 ERP-defined component classification rests on this rule).
- **Evidence**: every aggregate sketch's §5 Web/ERP Ownership section. The strictest expression is `pricing-contracts.md` (1.4.5) with a 13/0/0 ownership distribution (13 ERP-defined, 0 Shared, 0 Web-only).
- **Constitutional anchor**: `AGENTS.md` ERP Ownership Rules (Hard Rules §3 in `CLAUDE.md`).

### 2.2 Rule 2 — Read-Only by Default

- **Rule**: the default mode for every ERP-defined component on the web side is **read-only mirror**. Write-back to NET7 requires explicit justification, an explicit `Net7XAdapter` write surface, and a documented adapter-contract section.
- Read-only is the strictest structural expression of Rule 1: zero write surface means idempotency by construction (see Rule 3).
- **Pattern foundation**: ADR-0002 Pattern 4 — Adapter Profile Set, *Read-only Adapter* profile.
- **Evidence (read-only adapters, 3 of 5)**: `Net7ProductAdapter` (`catalog.md` §5), `Net7DocumentAdapter` (`documents-compliance.md` §5), `Net7PricingAdapter` (`pricing-contracts.md` §5). All three carry `INV-N (behavioural)` invariants forbidding local mutation (e.g., `catalog.md` `INV-6`, `documents-compliance.md` `INV-5`, `pricing-contracts.md` `INV-5`).
- **Write-back exceptions**: only `Net7SampleAdapter` (Write-path) and `Net7CustomerAdapter` (Hybrid) carry write surfaces. Both are explicitly justified in their §5 sections.

### 2.3 Rule 3 — Idempotency Mandate

- **Rule**: every write-back operation against NET7 **must** be idempotent. The implementation strategy depends on the adapter profile:
  - **Read-only adapters**: idempotency by construction (no write surface to begin with).
  - **Write-path adapters**: idempotency via client-generated **correlation tokens** submitted with each write. Retry against the same token is a no-op or returns the same result.
  - **Hybrid adapters**: per-component idempotency strategy. Some components (e.g., user-profile writes) follow the correlation-token pattern; others are excluded from write-back entirely (e.g., `AuthenticationCredential` per Rule 4).
- **Pattern foundation**: ADR-0002 Pattern 4 — Adapter Profile Set (Write-path + Hybrid profiles).
- **Evidence**: `sample.md` §5 `Net7SampleAdapter` write-back with correlation-token; `customer-account.md` §5 `Net7CustomerAdapter` mixed strategy (idempotent profile writes; preferences and credentials excluded by structure).
- **Implication**: retry-with-same-token is a safe Sprint-2 implementation pattern across the entire adapter family.

### 2.4 Rule 4 — Identity Bridging Boundary

- **Rule**: web-side authenticated identity bridges to NET7 `CustomerId` via `Net7CustomerAdapter` at session start. The bridge is the only place where web identity meets ERP customer master.
- **`AuthenticationCredential` NEVER propagates to NET7** under any code path: not via writes, not via logs, not via analytics, not via debug serialisation. The adapter contract excludes credential fields at compile time; logs reject credential serialisation.
- **`UserPreferences` are Web-only** — never stored in or propagated to NET7. Web-only persistence in the platform database.
- **Pattern foundation**: ADR-0002 Pattern 2 — Invariant Sub-Class Taxonomy, *security-boundary* sub-class.
- **Evidence**: `customer-account.md` `INV-5` (security-boundary). The `Net7CustomerAdapter` §5 section explicitly enumerates write-back exclusions.
- **Open**: `OQ-017` + `OQ-018` + `OQ-023` (User-Auth-SSO Placement Trio per `open-questions-master.md` §4 Group 1) collectively determine bridging granularity. SSO-V1 adoption pulls `AuthenticationCredential` toward its own Security aggregate and re-frames bridging at user level rather than customer level.

### 2.5 Rule 5 — Failure Graceful Degradation

- **Rule**: ERP availability is **not assumed**. Every adapter must specify its failure-mode tier in §5 of the corresponding aggregate sketch. Per `AGENTS.md`: *"All ERP failures must degrade gracefully."*
- **Pattern foundation**: ADR-0002 Pattern 4 — Adapter Profile Set with explicit failure-mode-tier progression per profile.
- **Failure-mode-tier progression** (captured across all 5 adapters):
  1. **1-tier** — `Net7ProductAdapter`: stale-cache + `last-synced-at` banner.
  2. **3-tier** — `Net7DocumentAdapter`: cache → fresh fetch → direct NET7-DMS link.
  3. **3-tier with business-process-blocking** — `Net7PricingAdapter`: stale-cache banner → "pricing temporarily unavailable" → Order/RFQ flow blocked when stale-tolerance exceeded.
  4. **Retry + escalation** — `Net7SampleAdapter`: write-side retry with correlation token + escalation if persistent.
  5. **Mixed** — `Net7CustomerAdapter`: stale-banner for ERP-sourced; auth-resilience for web-only credential paths.
- **Pricing's business-process-blocking is the strictest tier**: stale pricing does not degrade silently; it blocks downstream Order and RFQ submission. Justification: stale pricing causes financial / contractual inconsistency that is downstream of AOT but originates in adapter behaviour.

### 2.6 Rule 6 — Status Sync Mechanism

- **Rule**: ERP-driven status changes mirror to the web via the **state-machine read-only-mirror** notation variant. The web **never** initiates state transitions for ERP-driven entities; ingest-time validation against the state-machine invariant catches malformed transitions and escalates rather than silently accepting.
- **Push-vs-pull architecture should be UNIFORM** across all five adapters. Per-adapter ad-hoc decisions fragment monitoring, backpressure, and replay semantics.
- **Pattern foundation**: ADR-0002 Pattern 3 — State-Machine Notation Variants, *read-only-mirror* variant.
- **Evidence**: `documents-compliance.md` `INV-3` (`DocumentStatus` read-only-mirror); `pricing-contracts.md` `INV-2` (`ContractStatus`) and `INV-7` (`PriceValidity`) — same pattern reused twice within one aggregate; `sample.md` `INV-3` (`SampleStatus` lifecycle, web models same vocabulary but does not originate transitions, per `OQ-012`).
- **Open**: `OQ-005` (catalog), `OQ-013` (sample), `OQ-022` (customer-account), `OQ-030` (documents), `OQ-037` (pricing) — collective resolution required as **CP-5** in `open-questions-master.md` §5 (Group 3 *Status-Sync Mechanism Family*). One TopM-engineer interview on `erp-discovery.md` §3.f resolves all five.

### 2.7 Rule 7 — Eventual Consistency Acknowledgment

- **Rule**: cross-aggregate invariants are **eventually consistent**, not transactional. Acceptable lag windows are specified per adapter, typically `< 5 minutes` for V1 (subject to TopM-discovery validation).
- **Pattern foundation**: ADR-0002 Pattern 2 — Invariant Sub-Class Taxonomy, *cross-aggregate* sub-class.
- **Evidence (concrete couplings)**: `documents-compliance.md` `INV-6` (`RequiredDocumentSet` ↔ Catalog Product master); `pricing-contracts.md` `INV-4` (`ProductPrice` ↔ Catalog Product master). Both mark eventual consistency explicitly with a hypothesis on the lag window pending `erp-discovery.md` §3.f validation.
- **Direct manifestation**: `master-data-risk-register.md` `MDR-018` (NET7 sync-lag exceeds business threshold) — the operational realisation of the eventual-consistency window when sync mechanisms underperform. Couples to Rule 6 collective resolution.
- **Implication**: Sprint-2 adapter implementations must surface lag explicitly (`last-synced-at` timestamps in UI per Rule 5) rather than pretend cross-aggregate consistency is transactional.

---

## 3. Alternatives considered

### Alt 1 — Implicit Boundary-Rules (per-aggregate §5 only)

Leave the operational rules distributed across the five aggregate sketches' §5 Adapter sections. Sprint-2 ERP Boundary Design re-derives rules from those sections.

**Rejected**: §5 Adapter sections are scattered across five documents; there is no single source of truth; Sprint-2 implementations would re-derive boundary rules ad-hoc and risk per-implementation drift. The same critique that produced ADR-0002 (tacit patterns are not handover-ready) applies here.

### Alt 2 — Boundary Rules in `AGENTS.md` only

Extend `AGENTS.md` with a new "ERP Boundary Rules" section instead of an ADR.

**Rejected**: `AGENTS.md` is the high-level constitutional framework — Engineering Principles, Hard Rules, ERP Ownership posture, persona definitions. It is intentionally tool-agnostic and does not specify adapter-level operational rules. Concrete rules with adapter-level evidence and Sprint-2 implementation hooks belong in an ADR. `AGENTS.md` remains the *motivational source*; ADR-0003 is the *actionable rules*.

### Alt 3 — Boundary Rules as 7 individual ADRs (one per rule)

ADR-0003 through ADR-0009, each codifying one rule.

**Rejected**: 7 small ADRs fragments the decision scope. The rules are interrelated — Rule 3 (Idempotency) directly supports Rule 2 (Read-Only-by-Default) which supports Rule 1 (ERP-as-SoT). Splitting them across seven ADRs hides the coupling. A single ADR with 7 sub-decisions in §2 matches the ADR-0001 + ADR-0002 style (both used `### 2.N` sub-decisions) and keeps the rule-set as a coherent contract.

---

## 4. Consequences

### 4.1 Positive

- **Sprint-2 ERP Boundary Design** has explicit rule-foundation as input rather than re-deriving from five scattered §5 sections.
- **Adapter-implementation contracts** can verify against the 7 Rules at design time and at PR review.
- **`AGENTS.md` Hard Rules** are structurally instantiated at adapter level — tractable and verifiable.
- **Cross-adapter consistency** is enforceable, especially Rule 6 (Status-Sync uniformity across the family).
- **SLO design** is grounded in the failure-mode-tier vocabulary (Rule 5) — "this adapter is 1-tier / 3-tier / 3-tier-blocking" is a shared term.
- **Pattern→Rule mapping** in §2 makes ADR-0002's structural patterns operational. ADR-0002 says *how to model*; ADR-0003 says *what the model must obey*.

### 4.2 Negative / Risks

- **Five-sketch evidence base** — the 7 rules are validated against the five P1 aggregate sketches only. Sprint-2 real-data pulls and the first NET7-discovery interviews may surface rule tensions (e.g., Rule 3 idempotency depends on TopM correlation-token support; if NET7 does not expose this, Rule 3 needs an alternative mechanism).
- **Rule 4 Identity-Bridging granularity** depends on the SSO-V1 decision (Group 1 trio resolution). If SSO is in scope for V1, Rule 4 may need to relocate the bridge from session-start to a dedicated Security aggregate.
- **Rule 6 Push-vs-Pull collective decision** is gated by TopM capabilities (`erp-discovery.md` §3.f). Per-adapter ad-hoc decisions may be forced if NET7 only supports polling, in which case Rule 6's *uniform* clause is partially satisfied but the architecture differs from a pure push model.
- **Rule 7 lag window of < 5 minutes** is a hypothesis; real NET7 sync behaviour may force a wider window, with downstream implications for Rule 5 failure-mode tiers (e.g., shifting Pricing's blocking threshold).

### 4.3 Open questions / Revisit triggers

This ADR is revisited if any of the following occurs:

- **Group 3 *Status-Sync Mechanism Family* resolution** (`open-questions-master.md` §4 Group 3 / §5 CP-5) → Rule 6 push-vs-pull architecture finalised. Until then, Rule 6 carries an explicit "uniform" clause with an open implementation choice.
- **Group 1 *User-Auth-SSO Placement Trio* resolution** (Group 1 / `OQ-017` + `OQ-018` + `OQ-023`) → Rule 4 Identity-Bridging granularity finalised. SSO-V1 adoption forces re-framing.
- **`OQ-035` Pricing Aggregate-Boundary** (CP-4, single-vs-two-aggregate split) → Rule 5 failure-mode-tier may need adjustment for the Pricing adapter (single tier or split tier per sub-aggregate).
- **Sprint-3 first code scaffold** validates or breaks the Idempotency Mandate (Rule 3). If correlation-token support is not natively available in NET7, the rule may need an adapter-side dedup window with explicit failure-mode escalation as a fallback.
- **`MDR-018` NET7 sync-lag** real-world measurement → Rule 7 lag-window threshold tightened or loosened.
- **ADR-0004 (Sprint-1-Closeout P5c) Open-Question Governance** defines the OQ → ADR promotion path. As Group 1 / Group 3 / CP-4 resolutions accumulate, ADR-0003 may receive Rule-level amendments via that path.

---

## 5. Approval

- **Decided by**: Human (System Architect, Carlos).
- **Date**: 2026-05-09.
- **Recorded by**: Claude (chat) and Claude Code, as part of Sprint-1-Closeout-Rework P5b.
- **Status transitions**: `Accepted`. Rule-level amendments may be added in successor ADRs that name the affected rule(s); the ADR as a whole moves to `Superseded` only via a successor ADR that names this one.

---

## 6. Related documents

- [`AGENTS.md`](../../AGENTS.md) — constitutional source for the ERP Ownership posture; ERP Hard Rules; ERP Integration Rules. Rule 1 is the actionable instantiation of `AGENTS.md` ERP Ownership Rules.
- [`adr/0001-stack-and-deployment.md`](./0001-stack-and-deployment.md) — predecessor ADR; format and section-order template.
- [`adr/0002-domain-patterns.md`](./0002-domain-patterns.md) — pattern foundation for all rules. Pattern→Rule mapping in §2: Pattern 4 (Adapter Profile Set) → Rules 2, 3, 5; Pattern 2 (Invariant Sub-Class Taxonomy, security-boundary) → Rule 4; Pattern 2 (cross-aggregate) → Rule 7; Pattern 3 (State-Machine read-only-mirror) → Rule 6.
- [`domain/aggregates/catalog.md`](../domain/aggregates/catalog.md) — Rule 2 evidence (read-only, `INV-6`); Rule 5 evidence (1-tier failure-mode).
- [`domain/aggregates/sample.md`](../domain/aggregates/sample.md) — Rule 3 evidence (correlation-token write-path); Rule 5 evidence (retry+escalation); Rule 6 evidence (`SampleStatus` bidirectional with NET7-driven advances).
- [`domain/aggregates/customer-account.md`](../domain/aggregates/customer-account.md) — Rule 4 evidence (`INV-5` security-boundary); Rule 3 evidence (mixed idempotency strategy); Rule 5 evidence (mixed failure-mode).
- [`domain/aggregates/documents-compliance.md`](../domain/aggregates/documents-compliance.md) — Rule 2 evidence (read-only-mirror); Rule 5 evidence (3-tier); Rule 6 evidence (`INV-3` read-only-mirror); Rule 7 evidence (`INV-6` cross-aggregate, first INV-tag use).
- [`domain/aggregates/pricing-contracts.md`](../domain/aggregates/pricing-contracts.md) — Rule 1 evidence (13/0/0 strictest); Rule 2 evidence; Rule 5 evidence (3-tier with business-process-blocking); Rule 6 evidence (`INV-2`, `INV-7` read-only-mirror); Rule 7 evidence (`INV-4` cross-aggregate).
- [`domain/adapter-topology.md`](../domain/adapter-topology.md) — visual reference for the rule application across the five adapters; failure-mode-tier comparison in §3.
- [`domain/open-questions-master.md`](../domain/open-questions-master.md) — Critical-Path OQs gating rule-level amendments (CP-1 OQ-017 Identity-Bridging, CP-4 OQ-035 Pricing-boundary, CP-5 Group 3 Status-Sync).
- [`risks/master-data-risk-register.md`](../risks/master-data-risk-register.md) — `MDR-018` (NET7 sync-lag) is the direct manifestation of Rule 7; Rule 5 failure-mode tiers correspond to the adapter-side mitigations for the master-data risks.
- *Forward*: `docs/adr/0004-open-question-governance.md` (Sprint-1-Closeout P5c) — governs the OQ → ADR promotion path that produces rule-level amendments to this ADR.
