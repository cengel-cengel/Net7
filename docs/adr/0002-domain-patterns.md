# ADR 0002 — Domain Modelling Patterns

| Status | **Accepted** |
|---|---|
| Date | 2026-05-09 |
| Decided by | Human (System Architect) |
| Authors | Human + Claude (chat) + Claude Code |

---

## 1. Context

- Sprint 1 produced five aggregate sketches as the Web/ERP boundary foundation: [`catalog.md`](../domain/aggregates/catalog.md) (Task 1.4.1), [`sample.md`](../domain/aggregates/sample.md) (1.4.2), [`customer-account.md`](../domain/aggregates/customer-account.md) (1.4.3), [`documents-compliance.md`](../domain/aggregates/documents-compliance.md) (1.4.4), [`pricing-contracts.md`](../domain/aggregates/pricing-contracts.md) (1.4.5).
- A domain-modelling pattern set emerged organically across these sketches — not from a top-down specification but as Pattern reuse and refinement at each new sketch. By Task 1.4.5 the pattern was stable enough to declare *"no new sub-classes — pure pattern application"* in the closing aggregate.
- Without an explicit ADR the pattern remains **tacit** in five separate documents. `AGENTS.md` mandates *"explicit architecture, modular services, documentation"* and a *"handover-ready"* posture. Tacit patterns fail both criteria.
- Future aggregate sketches outside the five P1 contexts (RFQ Lifecycle, Order Management, Workspaces, and possibly Inventory if `OQ-027` resolves toward a sixth aggregate per [`open-questions-master.md`](../domain/open-questions-master.md) §5 CP-6) need the same pattern discipline. Without an ADR they would re-discover the patterns from the five existing sketches by archaeology, with associated drift risk.
- This ADR codifies the pattern set as the canonical structure for Domain-Modelling deliverables. It does **not** itself create new patterns; it consolidates what Sprint 1 already produced.

---

## 2. Decision

Adopt the five-pattern set documented below as canonical for all current and future Domain-Modelling sketches in this repository. Patterns are numbered in the order they emerged across Sprint 1 (catalog → pricing), not by importance.

### 2.1 Pattern 1 — 7-H2 Aggregate-Sketch Template

Every aggregate sketch uses the same seven-section H2 structure:

1. **Purpose** — DDD framing, Sprint-N context, working-hypothesis disclaimer.
2. **Methodology** — DDD vocabulary; Web/ERP/Shared/Web-only ownership; invariant typology; hypothesis marking; cross-aggregate references by ID only.
3. **Aggregate Boundary** — Aggregate Root + Inside + Outside + Discussion (Pros/Contras + provisional decisions).
4. **Internal Structure** — Schema (markdown table) + Invariants (`INV-N` register) + Consistency Rules (transactional + eventual).
5. **Web/ERP Ownership and Adapter Boundary** — per-component statement + named adapter (`Net7XAdapter`) + Sprint-2 Hook (ERP capabilities to validate).
6. **Cross-Aggregate Relationships and Open Architecture Questions** — ID-references + `OQ-N` register.
7. **References** — predecessor aggregate sketches + supporting docs.

H3 sub-section count is uniformly **11** across all five sketches: 4 in §3 (Root / Inside / Outside / Discussion), 3 in §4 (Schema / Invariants / Consistency), 2 in §5 (Adapter / Sprint-2 Hook), 2 in §6 (ID-References / Open Questions). Established in [`catalog.md`](../domain/aggregates/catalog.md) (1.4.1); applied identically in [`sample.md`](../domain/aggregates/sample.md), [`customer-account.md`](../domain/aggregates/customer-account.md), [`documents-compliance.md`](../domain/aggregates/documents-compliance.md), [`pricing-contracts.md`](../domain/aggregates/pricing-contracts.md).

### 2.2 Pattern 2 — Invariant Sub-Class Taxonomy (5)

Every invariant in an aggregate sketch carries one of five sub-class tags:

- **`structural`** — cardinality, presence, reference rules. *Evidence*: `catalog.md` `INV-1` (Product has at least one ProductVariant); `sample.md` `INV-1` (SampleRequest references at least one Product).
- **`behavioural`** — read-only enforcement, transition rules, all-fields rules. *Evidence*: `catalog.md` `INV-6` (ERP-sourced fields read-only on web); `customer-account.md` `INV-6` (same pattern); `pricing-contracts.md` `INV-5` (pricing logic never on web — `AGENTS.md` Hard Rule).
- **`state-machine`** — legal status transitions, status-history immutable. *Evidence*: `sample.md` `INV-3` (`SampleStatus` lifecycle); `customer-account.md` `INV-3` + `INV-4` (`ContractStatus` + `UserStatus`); `documents-compliance.md` `INV-3` (`DocumentStatus` read-only-mirror); `pricing-contracts.md` `INV-2` + `INV-7` (`ContractStatus` + `PriceValidity` read-only-mirror).
- **`cross-aggregate`** — eventual-consistency boundaries. *Evidence*: first applied as INV-tag in `documents-compliance.md` `INV-6` (`RequiredDocumentSet` ↔ Catalog Product master); reused in `pricing-contracts.md` `INV-4` (`ProductPrice` ↔ Catalog).
- **`security-boundary`** — Web-only data with hard never-propagate-to-ERP rules. *Evidence*: introduced in `customer-account.md` `INV-5` (`AuthenticationCredential` content never reaches NET7; logs reject credential serialization; adapter contract excludes credential fields at compile time).

The taxonomy is closed for the five P1 aggregates. Future aggregates may introduce additional sub-classes, treated as additive extensions rather than redefinitions.

### 2.3 Pattern 3 — State-Machine Notation Variants (2)

The `state-machine` invariant sub-class supports two notation variants:

- **`bidirectional`** — the web side can transition state locally, optionally synced back to NET7. *Evidence*: `sample.md` `SampleStatus` (web orchestrates request creation, status advances driven by NET7 fulfilment events but web models the same vocabulary); `customer-account.md` `UserStatus` (web can lock a user on failed-login; ERP can suspend on contract event; both writes converge on the same VO).
- **`read-only-mirror`** — the web mirrors a NET7-driven state machine without local transitions; web-side mutation is a violation. *Evidence*: `documents-compliance.md` `INV-3` (`DocumentStatus`: *Active → Expired / Superseded*, NET7-driven); `pricing-contracts.md` `INV-2` (`ContractStatus`) and `INV-7` (`PriceValidity`) — same pattern reused twice within one aggregate.

Notation tag in invariant text reads `(state-machine, read-only-mirror)` to mark the variant; bidirectional variant uses the unqualified `(state-machine)` tag.

### 2.4 Pattern 4 — Adapter Profile Set (3)

The `Net7XAdapter` mediation layer is one of three profiles. Failure-Mode-Tier escalates from simple to business-process-blocking across the read-only set; the write-path and hybrid profiles add their own failure shapes.

- **Read-only Adapter** — no write surface; idempotency by construction. *Evidence + Failure-Mode-Tier progression*:
  - `Net7ProductAdapter` (`catalog.md` §5) — 1-tier: stale-cache + `last-synced-at` banner.
  - `Net7DocumentAdapter` (`documents-compliance.md` §5) — 3-tier: cache → fresh fetch → direct NET7-DMS link.
  - `Net7PricingAdapter` (`pricing-contracts.md` §5) — 3-tier with **business-process-blocking** at threshold: stale-cache banner → "pricing temporarily unavailable" → Order/RFQ flow blocked.
- **Write-path Adapter** — explicit write-back methods; idempotency via client-generated correlation tokens; retry + escalation failure mode. *Evidence*: `Net7SampleAdapter` (`sample.md` §5) — first write-path adapter in the platform; pattern source for Customer Account write-back.
- **Hybrid Adapter** — ERP ground-truth + Web-only writes excluded by structure + Identity-Bridging. *Evidence*: `Net7CustomerAdapter` (`customer-account.md` §5) — read of customer master, write-back of user-profile (gated by `OQ-021`), `UserPreferences` and `AuthenticationCredential` excluded from write surface at compile time per `INV-5`. Identity-Bridging maps web-side authenticated identity to NET7 `CustomerId` at session start.

Per-profile cross-references and the failure-mode tier comparison live in [`adapter-topology.md`](../domain/adapter-topology.md) §3.

### 2.5 Pattern 5 — Pattern-Application Self-Report Discipline

Every aggregate sketch from `pricing-contracts.md` (1.4.5) onward must explicitly self-report which patterns it reuses, which extensions are *not relevant*, and whether any new patterns are introduced. The discipline lives at four anchor points inside the sketch document itself:

1. **§1 Purpose final paragraph** — explicit "no new sub-classes" or "introduces new sub-class X" statement.
2. **§2 Methodology bullets** — name each carried-over Sub-Class and Notation-Variant.
3. **§2 Methodology** — explicit "not relevant for this aggregate" enumeration of patterns introduced in earlier sketches but not used here.
4. **§7 References** — each predecessor sketch annotated with the patterns it contributed.

*Evidence*: established in `pricing-contracts.md` (1.4.5) — *"no new methodology sub-classes are introduced here"* in §1; explicit list of "not relevant" extensions in §2 (Shared, Web-only, security-boundary, write-path adapter); §7 references annotate each predecessor with its pattern contribution. Reusable for future aggregate sketches (RFQ, Order, Workspaces, possibly Inventory).

The discipline is the structural defence against pattern drift across the growing sketch corpus.

---

## 3. Alternatives considered

### Alt 1 — No explicit Pattern-ADR (tacit pattern)

Leave the pattern set distributed across the five aggregate sketches. Future engineers extract patterns by reading all five.

**Rejected**: violates `AGENTS.md` handover-readiness goals; pattern drift risk on every new sketch is high; impossible to verify pattern consistency at PR-review time without a single source of truth.

### Alt 2 — Pattern Library as a separate `/docs/domain/patterns/` folder

Three files: `aggregate-template.md`, `invariant-subclasses.md`, `adapter-profiles.md`. Patterns documented as templates rather than as a single ADR.

**Rejected**: ADRs are the canonical form for architectural decisions in this repository (see ADR-0001). A folder-split fragments the decision scope, makes status (`Accepted` / `Proposed`) ambiguous, and weakens handover traceability. The pattern set *is* an architectural decision; treating it as one is the right form. (A future Pattern Library, derived from this ADR, remains an option as a how-to document — but the *decision* belongs in an ADR.)

### Alt 3 — ADR Status `Proposed` rather than `Accepted`

Open the ADR with `Proposed` status to signal the patterns are still subject to revision pending Sprint 2/3 implementation.

**Rejected**: the patterns are validated across five aggregate sketches and have already proved transferable across read-only, write-path, and hybrid contexts. Sprint-1-Closeout discipline requires *closure* of decisions made during Sprint 1 — `Proposed` is closure-deferral. `Accepted` with explicit `§4.2 Risks` and `§4.3 Open questions` provides the revisit path without leaving the decision indefinite.

---

## 4. Consequences

### 4.1 Positive

- **Pattern consistency enforceable** — future aggregate sketches verify against this ADR; drift detectable at PR-review.
- **Onboarding friction reduced** — new engineers read one ADR rather than archaeology across five aggregate sketches.
- **Sprint-2+ aggregate sketches** (RFQ, Order, Workspaces, possibly Inventory per `OQ-027`) follow the template without re-discovery.
- **Cross-Aggregate INV-tags** + Coupled-OQ-Groups (per [`open-questions-master.md`](../domain/open-questions-master.md) §4) build on the pattern foundation rather than competing with it.
- **Adapter implementation in Sprint 2** has a fixed profile set (Read-only / Write-path / Hybrid) to map onto, plus a failure-mode tier vocabulary for SLO design.
- **Pattern-Application Self-Report Discipline** (Pattern 5) is now an enforceable expectation rather than a one-off practice in `pricing-contracts.md`.

### 4.2 Negative / Risks

- **Five-sketch evidence base** — the patterns are validated across the five P1 aggregates only. RFQ Lifecycle, Order Management, and Workspaces have different lifecycle and ownership shapes (especially RFQ multi-step transitions); the pattern set may need additive extensions when those aggregates are sketched.
- **Notation Variant differentiation** — Pattern 3's two variants (bidirectional / read-only-mirror) cover the five P1 aggregates; new domain shapes (e.g., bidirectional with web-side authority for some transitions) might require a third variant. Treated as additive when it occurs.
- **Pattern reification was emergent** — none of the patterns were specified up-front in Sprint 1; they were retroactively named. Future patterns may also emerge organically and require their own ADR amendments. Acceptable cost for an emergent-design phase.
- **`Accepted` status based on single-session convergence** — the five sketches were produced in one continuous session. Cross-session validation (different contributors, different time horizons) is pending and may surface convention drift.

### 4.3 Open questions / Revisit triggers

This ADR is revisited if any of the following occurs:

- **Sprint-3 first code scaffold** validates or breaks the adapter profile set in implementation. Code scaffold reveals whether the failure-mode tiers, idempotency mechanisms, and identity-bridging shapes hold under real implementation constraints.
- **RFQ + Order + Workspaces aggregate sketches** in Sprint-2/3 reveal whether the 7-H2 template and invariant taxonomy generalise. RFQ in particular has a multi-step lifecycle that may stress Pattern 3.
- **`OQ-027` BatchReference resolution** flips toward an Inventory aggregate ([`open-questions-master.md`](../domain/open-questions-master.md) §5 CP-6 / §4 Group 2). The sixth aggregate sketch validates the pattern further or requires extension.
- **`OQ-017` Customer Account Aggregate-Root** (CP-1) flips toward User-as-Root. Hybrid adapter pattern may need re-framing — the Identity-Bridging responsibility relocates.
- **ADR-0003 (Sprint-1-Closeout P5b)** ERP-Boundary-Rules formalises the adapter contract specification and may surface pattern-set tensions.
- **ADR-0004 (Sprint-1-Closeout P5c)** Open-Question Governance defines the OQ → ADR promotion path and may amend Pattern 5 (self-report discipline).

---

## 5. Approval

- **Decided by**: Human (System Architect, Carlos).
- **Date**: 2026-05-09.
- **Recorded by**: Claude (chat) and Claude Code, as part of Sprint-1-Closeout-Rework P5a.
- **Status transitions**: `Accepted`. Re-open to `Superseded` only via a successor ADR that names this one.

---

## 6. Related documents

- [`AGENTS.md`](../../AGENTS.md) — engineering charter; Engineering Principles (*"explicit architecture, modular services, documentation"*); handover-readiness goals.
- [`adr/0001-stack-and-deployment.md`](./0001-stack-and-deployment.md) — predecessor ADR; format and section-order template for this ADR.
- [`domain/aggregates/catalog.md`](../domain/aggregates/catalog.md) — Pattern 1 source (7-H2 template); evidence for Pattern 2 (`INV-6` behavioural).
- [`domain/aggregates/sample.md`](../domain/aggregates/sample.md) — Pattern 4 source (Write-path Adapter); evidence for Pattern 3 bidirectional variant.
- [`domain/aggregates/customer-account.md`](../domain/aggregates/customer-account.md) — Pattern 2 source (security-boundary sub-class); Pattern 4 Hybrid Adapter source.
- [`domain/aggregates/documents-compliance.md`](../domain/aggregates/documents-compliance.md) — Pattern 3 source (read-only-mirror notation variant); Pattern 2 first INV-tag use of cross-aggregate.
- [`domain/aggregates/pricing-contracts.md`](../domain/aggregates/pricing-contracts.md) — Pattern 5 source (self-report discipline); Pattern 4 read-only-with-business-process-blocking failure-mode tier.
- [`domain/bounded-contexts.md`](../domain/bounded-contexts.md) — eight bounded contexts and pair-relationships; the structural backdrop the pattern set serves.
- [`domain/capability-map.md`](../domain/capability-map.md) — adapter-to-capability mapping.
- [`domain/glossary.md`](../domain/glossary.md) — DDD terminology baseline used across all five sketches.
- [`domain/open-questions-master.md`](../domain/open-questions-master.md) — Sprint-1-Closeout P1; Critical-Path OQs that may trigger §4.3 revisits.
- [`domain/adapter-topology.md`](../domain/adapter-topology.md) — Sprint-1-Closeout P4; visual complement to Pattern 4 Adapter Profile Set.
- [`workflow.md`](../workflow.md) — collaboration protocol; §15 PR Review Sequence (codified in PR #17 / Sprint-1-Closeout P3) governs the operational application of this ADR.
- *Forward*: `docs/adr/0003-erp-boundary-rules.md` (Sprint-1-Closeout P5b) — ERP boundary contract specification building on Patterns 4 and 2.
- *Forward*: `docs/adr/0004-open-question-governance.md` (Sprint-1-Closeout P5c) — OQ-lifecycle governance and Risk-Register lifecycle; may amend Pattern 5.
