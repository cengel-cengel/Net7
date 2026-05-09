# ADR 0004 — Open-Question Governance

| Status | **Accepted** |
|---|---|
| Date | 2026-05-09 |
| Decided by | Human (System Architect) |
| Authors | Human + Claude (chat) + Claude Code |

---

## 1. Context

- Sprint-1-Closeout P1 produced [`open-questions-master.md`](../domain/open-questions-master.md) with 46 globally numbered OQs (`OQ-001 … OQ-046`), 5 Coupled-OQ Groups, and 6 Critical-Path OQs gating Sprint-2 *ERP Boundary Design*.
- ADR-0002 ([`adr/0002-domain-patterns.md`](./0002-domain-patterns.md), Sprint-1-Closeout P5a) and ADR-0003 ([`adr/0003-erp-boundary-rules.md`](./0003-erp-boundary-rules.md), P5b) both forward-reference ADR-0004 as the OQ → ADR promotion path that produces rule-level amendments.
- Without governance: OQ resolution is ad-hoc, promotion criteria implicit, coupled-group resolution risks happening uncoordinated (e.g., Group 1 SSO-Trio resolved one-question-at-a-time would leave the trio in an inconsistent state).
- This ADR is **meta-architectural** — it governs the lifecycle of architectural decisions themselves, not domain content.

**Triple-distinction across the Sprint-1-Closeout ADR triplet**:

- **ADR-0002 — HOW** we structure aggregate-modelling sketches (5 structural patterns: 7-H2 template, invariant taxonomy, state-machine notation variants, adapter profile set, self-report discipline).
- **ADR-0003 — WHAT** rules govern Web↔ERP interaction at adapter level (7 boundary rules: SoT, read-only-default, idempotency, identity-bridging, failure-graceful-degradation, status-sync, eventual-consistency).
- **ADR-0004 — HOW DECISIONS EVOLVE** (this ADR): governance for unresolved questions transitioning to resolved decisions.

ADR-0004 is the terminal ADR of the Sprint-1-Closeout triplet. It **closes the forward-reference loops** opened by ADR-0002 §4.3 and ADR-0003 §4.3 (both expected an ADR-0004 to govern OQ-lifecycle). It also references back to ADR-0002 (Pattern 5 Self-Report Discipline as methodical precursor for the bidirectional-traceability rule below) and ADR-0003 (7-Rule format as structural precursor for this ADR's 6 governance rules).

---

## 2. Decision

Adopt the six governance rules below as the binding lifecycle for every Open Question in this codebase, current and future. Six rules rather than seven: OQ-master maintenance is naturally part of Rule 1 (Lifecycle status updates) and Rule 6 (Backreference bidirectional traceability) — splitting it into a seventh rule would fragment the governance without substantive benefit.

### 2.1 Rule 1 — OQ Lifecycle (5 States)

Every OQ moves through a forward-only lifecycle of five states. Rollback to an earlier state is not permitted; if a decision needs revisiting, a new OQ is raised.

- **`Open`** — the question is recorded in [`open-questions-master.md`](../domain/open-questions-master.md) §3 with no resolution path scheduled. Default state for every newly raised OQ.
- **`In Discovery`** — discovery is scheduled or in progress. Concrete sources: `erp-discovery.md` clusters (typically c Datenmodell / d Lese-Operationen / e Schreib-Operationen / f Sync-Strategie) for TopM-Engineer questions, or `business-discovery.md` clusters (typically b Beschaffungsprozess / d Sample-Workflow / f Dokumentations-Bedürfnisse) for Customer-PM questions.
- **`Decided`** — Carlos has chosen the resolution + the resolution-form (per Rule 2). The decision is recorded in conversation or commit message, but not yet encoded in a canonical artifact.
- **`Encoded`** — the decision is recorded in a canonical artifact (ADR / Aggregate-Sketch / `AGENTS.md` / code-decision-comment per Rule 2). Encoding may span multiple artifacts (e.g., a coupled-group decision producing one ADR plus updates to two aggregate sketches).
- **`Closed`** — `open-questions-master.md` §3 Status column is updated to `Closed (artifact-ID)`. The status update happens in the same PR that performs the encoding (Rule 6 bidirectional traceability).

Rule 1 implication: an OQ is not "Closed" simply because Carlos has decided — it is Closed only when the decision is encoded in a canonical artifact and the OQ-master Status column reflects the location of the encoding.

### 2.2 Rule 2 — Resolution Form Choice (4 Forms)

When an OQ moves from `Decided` to `Encoded`, Carlos chooses one of four resolution forms based on the scope of the decision.

- **ADR** — for cross-cutting architectural decisions. Trigger: ≥2 aggregates affected, Critical-Path status (CP-1 … CP-6), Group-level coupling (Group 1 … Group 5), or foundational changes to the adapter contract or pattern foundation. See Rule 4 for promotion criteria.
- **Aggregate-Sketch update** — for aggregate-internal refinements that do not cross aggregate boundaries. The OQ resolution amends an `INV-N` definition, a §3 Discussion provisional decision, or a §5 adapter detail in a single aggregate sketch.
- **`AGENTS.md` amendment** — for constitutional changes (rare). The OQ resolution alters Engineering Principles, ERP Ownership Rules, ERP Integration Rules, or Persona definitions. Carlos's discretion; reserved for changes that ripple across all aggregates.
- **Code-decision-comment** — for implementation-level edge cases that surface during Sprint-3+ code work. Not architectural; lives in the code with a `// ASSUMPTION:` or `// DECISION:` comment and a commit-message reference to the OQ.

The decision criterion between `ADR` and `Aggregate-Sketch` is the most consequential edge; Rule 4 makes it explicit.

### 2.3 Rule 3 — Owner Authority (4 Owners)

The OQ-master `Owner` column (§3) names the primary input source for an OQ. Decision-authority always rests with Carlos; other owners provide input.

- **`Carlos`** — ultimate decision-authority for all 46 OQs. Single arbiter on borderline classifications (Rule 4) and on conflicts between input sources. The 18 OQs with `Owner = Carlos` in OQ-master §3 are pure architecture decisions where no external discovery is needed.
- **`TopM-Eng`** — technical-input-authority for `erp-discovery.md` clusters c, d, e, f. Carlos remains decision-authority but defers technical reality-check to TopM. 19 OQs in this category.
- **`Customer-PM`** — customer-input-authority for `business-discovery.md` clusters b, d, f, h, i. 5 OQs in this category.
- **`AOT-Sales / QM`** — domain-input-authority for AOT-internal stakeholders. AOT-Sales for §3.d / §3.h sales-process clusters; AOT-QM for §5.a / §5.b regulatory and audit clusters. 4 OQs in this category (2 AOT-Sales + 2 AOT-QM).

Some OQs require input from multiple owners (e.g., Multi-Tenant Cluster OQ-019 needs both Carlos architecture decision and Customer-PM customer-reality input). The Owner column names the primary input-source; Carlos coordinates multi-owner input as needed.

### 2.4 Rule 4 — Promotion Criteria (OQ → ADR)

Promote an OQ to an ADR if **any** of the following hold:

- **Cross-cutting impact**: the OQ resolution affects ≥2 aggregates (e.g., `OQ-026` `RequiredDocumentSet` placement affects both Catalog and Documents).
- **Critical-Path status**: the OQ is one of CP-1 … CP-6 in OQ-master §5 (CP-1 OQ-017 Aggregate-Root, CP-2 OQ-014 Sample-to-RFQ, CP-3 OQ-026 RequiredDocumentSet, CP-4 OQ-035 Pricing-boundary, CP-5 Group 3 Status-Sync, CP-6 OQ-027 BatchReference).
- **Group-level coupling**: the OQ is a member of one of the 5 Coupled-OQ Groups (Group 1 … Group 5 in OQ-master §4). See Rule 5.
- **Foundational decision**: the OQ resolution changes the adapter contract (per ADR-0003) or the pattern foundation (per ADR-0002). For example, an SSO-V1 outcome that splits `AuthenticationCredential` into a Security aggregate is foundational.

Stay in the **Aggregate-Sketch** form if:

- Single-aggregate operational refinement that does not cross boundaries.
- An internal Hypothesis becomes confirmed without cross-aggregate impact.
- The adapter contract is unaffected.

**Borderline OQs**: when the criterion is unclear, default is *"promote when in doubt"*. Handover-readiness (per `AGENTS.md`) prefers an explicit ADR over a buried sketch update. Carlos decides borderlines; consistency over time matters more than per-OQ optimisation.

### 2.5 Rule 5 — Coupled-OQ-Group Discipline

The 5 Coupled-OQ Groups in OQ-master §4 cannot be resolved one-question-at-a-time without producing inconsistent decisions. Each group has a discipline:

- **Group 1 — User / Auth / SSO Placement Trio** (`OQ-017` + `OQ-018` + `OQ-023`). **MUST resolve together** in a single ADR or `AGENTS.md` amendment. SSO-V1 adoption flips all three; deferring SSO keeps all three at current defaults. Resolution path: combined Customer-PM interview on `business-discovery.md` §3.i + Carlos SSO scope decision.
- **Group 2 — BatchReference Placement Coupling** (`OQ-027` + `OQ-001` + `OQ-004`). **Collective resolution preferred**. CP-6 BatchReference is the lead; resolution may trigger an Inventory aggregate sketch (sixth aggregate beyond the P1 set) which encodes all three OQs simultaneously. Resolution path: TopM-Eng interview on `erp-discovery.md` §3.c + §3.d.
- **Group 3 — Status-Sync Mechanism Family** (`OQ-005` + `OQ-013` + `OQ-022` + `OQ-030` + `OQ-037`). **MUST resolve as a single architecture decision** (CP-5 in OQ-master §5). The push-vs-pull architecture must be uniform across all five `Net7XAdapter` implementations per ADR-0003 Rule 6. Resolution path: single TopM-Eng interview on `erp-discovery.md` §3.f produces one ADR that resolves all five.
- **Group 4 — Identity & Multi-Tenant Cluster** (`OQ-019` + `OQ-020` + `OQ-024` + `OQ-016`). **Collective per SSO-V1 trigger** — couples to Group 1. V1 single-tenant simplifies UserId-bridging, per-User intrinsic role, user-bound sample visibility; V1 multi-tenant inverts all four.
- **Group 5 — Cross-Aggregate Conversion-Path Family** (`OQ-014` + `OQ-043` + `OQ-044`). **Collective in Sprint-2 RFQ Aggregate Sketch context**. All three concern multi-step web-orchestration vs ERP-native transitions. Resolution path: single TopM-Eng interview on `erp-discovery.md` §3.e covering all three transitions.

OQs not listed in any group may resolve individually per Rule 1 + Rule 2.

### 2.6 Rule 6 — ADR Backreference Discipline

Bidirectional traceability between OQs and the artifacts that resolve them. The discipline scales with resolution form (Rule 2):

- **ADR resolution path**:
  - The resolving ADR's §6 Related documents **MUST** list the OQ-IDs it resolves explicitly (e.g., *"resolves OQ-017, OQ-018, OQ-023 (Group 1)"*).
  - The OQ-master §3 Status column **MUST** be updated to `Closed (ADR-XXXX)` in the **same PR** that introduces or amends the ADR.
  - Forward-link from each OQ-master row to the resolving ADR (markdown link or ADR-ID reference).
- **Aggregate-Sketch resolution path**:
  - The sketch's §6 References **should** mention the closed OQ.
  - OQ-master §3 Status: `Closed (sketch-name §X.Y)`.
  - Same-PR update of OQ-master.
- **`AGENTS.md` amendment**:
  - Inline reference to the resolved OQ in the amended section.
  - OQ-master §3 Status: `Closed (AGENTS.md §X)`.
- **Code-decision-comment**:
  - `// DECISION: resolves OQ-NNN — see commit / PR / sketch` in the code.
  - OQ-master §3 Status: `Closed (code: <module>)`.

The bidirectional discipline is the structural defence against *ghost OQs* — questions marked Closed but unresolvable on inspection because the decision artifact cannot be located. Pattern foundation: ADR-0002 Pattern 5 Self-Report Discipline (the same anti-drift posture, applied to OQ resolution).

---

## 3. Alternatives considered

### Alt 1 — No formal OQ Governance

Let the 46 OQs resolve ad-hoc per Carlos discretion as discovery sessions surface answers.

**Rejected**: 46 OQs is too many for ad-hoc tracking; coupled-Group resolution risks happening uncoordinated (e.g., resolving OQ-017 without resolving OQ-018 and OQ-023 leaves the User/Auth/SSO architecture in an inconsistent state). Bidirectional traceability is essential for handover-readiness per `AGENTS.md`. The same critique that produced ADR-0002 (tacit patterns are not handover-ready) and ADR-0003 (tacit boundary rules are not actionable) applies here at the meta-level.

### Alt 2 — OQ Governance as `workflow.md` section

Embed OQ-lifecycle and promotion criteria in `workflow.md` as an additional operational section.

**Rejected**: governance over architectural decisions is itself an architectural decision (a meta-decision), not an operational protocol. `workflow.md` codifies *how Carlos and the AI tools collaborate* (Bridge Pattern, GO commands, Recovery DIFF, etc.); it is the *operational* tool. The lifecycle and promotion path of architectural decisions belongs in an ADR — that is what ADRs are for. Putting it in `workflow.md` would conflate two distinct concerns and make the constitutional vs operational layering unclear.

### Alt 3 — One ADR per Coupled-Group instead of a governance-meta-ADR

Three separate ADRs for Group 1 (SSO-V1), Group 3 (Status-Sync), and Group 5 (Conversion-Path), each with its own group-specific governance.

**Rejected**: the governance pattern (Lifecycle, Promotion Criteria, Backreference Discipline) is reusable across all 46 OQs and any future OQ. A meta-ADR captures the pattern once and applies to every group plus every individual OQ. Group-specific resolution ADRs *will* exist (they are the artifacts produced by Rule 5), but they follow this governance rather than each carrying their own redundant lifecycle definition.

---

## 4. Consequences

### 4.1 Positive

- **46 OQs have a clear lifecycle path** — Open → In Discovery → Decided → Encoded → Closed — with status visible at all times in OQ-master §3.
- **Promotion criteria transparent** (Rule 4) — Carlos and Claude (chat) and Claude Code share the same criteria for ADR-vs-Aggregate-Sketch decisions, reducing classification drift.
- **Coupled-Group discipline** (Rule 5) prevents inconsistent partial-group resolution; especially CP-5 Group 3 Status-Sync uniformity per ADR-0003 Rule 6.
- **Bidirectional traceability** (Rule 6) — every OQ can be traced to its resolution artifact, every resolution artifact lists its closed OQs.
- **Sprint-2/3 OQ-resolution** has a known process. New OQs raised during Sprint 2/3 enter the same lifecycle.
- **Future OQs** (added in Sprint 2+ from new aggregate sketches, RFQ / Order / Workspaces / possibly Inventory) follow the same lifecycle without rework.
- **Closes the Sprint-1-Closeout-ADR-Triplet forward-reference loops** — ADR-0002 §4.3 and ADR-0003 §4.3 forward-referenced ADR-0004; this ADR resolves both forward-references. The triplet (ADR-0002 + ADR-0003 + ADR-0004) is now self-consistent.
- **Pattern-set extension** — the bidirectional-traceability discipline (Rule 6) extends ADR-0002 Pattern 5 (Self-Report Discipline) from per-aggregate-sketch scope to per-OQ-resolution scope. Same anti-drift posture, broader application.

### 4.2 Negative / Risks

- **Single-session evidence base** — governance is calibrated against the Sprint-1-Closeout 46-OQ inventory. Future scale (Sprint 2+ adding 20+ OQs from RFQ / Order / Workspaces sketches) may stress the promotion criteria or the Coupled-Group discipline.
- **Coupled-Group discipline assumes coordination capability** — Rule 5 assumes Carlos can schedule combined discovery sessions (Group 1 SSO-Trio + Customer-PM, Group 3 Status-Sync + TopM-Eng). Real-world TopM and Customer scheduling delays may force partial-group resolution; the rule needs an exception clause then.
- **Owner-Authority (4 owners) assumes clear input-source distinction** — some OQs need multi-owner input (e.g., Multi-Tenant Cluster needs Carlos architecture + Customer-PM reality + possibly TopM-Eng technical). The Owner column names the primary input-source but doesn't fully capture multi-input scenarios.
- **Borderline OQs may oscillate** between Aggregate-Sketch and ADR (Rule 4 promotion-criteria edge cases). The "promote when in doubt" default biases toward more ADRs; this may produce ADR proliferation if future OQs hit the borderline frequently.
- **Cross-session validation pending** — like ADR-0002 and ADR-0003, this ADR is single-session emergent. Cross-session contributors (different time horizons, different team members) may surface convention drift in lifecycle interpretation.

### 4.3 Open questions / Revisit triggers

This ADR is revisited if any of the following occurs:

- **First OQ → ADR promotion** (likely Group 1 SSO-V1 in Customer-PM-Discovery, or Group 3 Push-vs-Pull in TopM-Engineer-Discovery) tests the governance end-to-end. Friction discovered at that point may amend Rule 4 (Promotion Criteria) or Rule 6 (Backreference Discipline).
- **Sprint-2 RFQ + Order Aggregate Sketches** add new OQs. The governance scaling test: does Rule 4 still produce reasonable promotion decisions when the OQ inventory is ~70+? Does Rule 5 need new Coupled-Group entries?
- **`OQ-027` BatchReference Resolution** triggers a sixth aggregate sketch (Inventory) per Group 2 collective resolution. The sixth aggregate is a real test of "single-aggregate vs ADR" promotion criterion: introducing a new aggregate is foundational and demands an ADR amendment to the pattern set (or a new ADR-0005).
- **Real-world TopM and Customer-PM Discovery scheduling delays** test the `In Discovery` lifecycle state duration. If discovery sessions stretch over weeks, OQ status may stagnate and need an explicit "stale" sub-state.
- **Future Sprint-2+ Process-Discovery** may introduce new resolution forms beyond the four in Rule 2 (e.g., RFC-style proposals for cross-team review, Discovery Notes for non-architectural findings). Rule 2 may grow.
- **Master-data risk register lifecycle** — `master-data-risk-register.md` (Sprint-1-Closeout P2) tracks 18 MDR risks with their own `Hypothesis → Identified → In-Mitigation → Mitigated → Resolved` lifecycle (5 states, parallel to the 5 OQ states here). Future amendment may unify the two lifecycles into a shared Decision-Lifecycle ADR if the parallels prove too strong to keep separate.

---

## 5. Approval

- **Decided by**: Human (System Architect, Carlos).
- **Date**: 2026-05-09.
- **Recorded by**: Claude (chat) and Claude Code, as part of Sprint-1-Closeout-Rework P5c.
- **Status transitions**: `Accepted`. Rule-level amendments may be added in successor ADRs that name the affected rule(s); the ADR as a whole moves to `Superseded` only via a successor ADR that names this one.
- **Triplet completion**: with this ADR, the Sprint-1-Closeout-ADR-Triplet (ADR-0002 + ADR-0003 + ADR-0004) is complete and self-consistent. Forward-reference loops opened by ADR-0002 §4.3 and ADR-0003 §4.3 are closed by this document.

---

## 6. Related documents

- [`AGENTS.md`](../../AGENTS.md) — engineering charter; handover-readiness goals motivate every governance rule below.
- [`adr/0001-stack-and-deployment.md`](./0001-stack-and-deployment.md) — predecessor ADR; format and section-order template.
- [`adr/0002-domain-patterns.md`](./0002-domain-patterns.md) — Pattern 5 Self-Report Discipline is the methodical precursor for Rule 6 (Backreference Discipline) — both impose bidirectional traceability between a decision and the artifact recording it. **Closes forward-reference loop opened by ADR-0002 §4.3** (which forward-referenced this ADR for OQ-lifecycle governance).
- [`adr/0003-erp-boundary-rules.md`](./0003-erp-boundary-rules.md) — 7-Rule format is the structural precursor for this ADR's 6-rule format. ADR-0003 Rule 6 (Status-Sync uniformity) maps directly onto this ADR's Rule 5 Group 3 collective-resolution discipline. **Closes forward-reference loop opened by ADR-0003 §4.3** (which forward-referenced this ADR for OQ → ADR promotion path).
- [`domain/open-questions-master.md`](../domain/open-questions-master.md) — Sprint-1-Closeout P1; the 46 OQs and the 5 Coupled-OQ Groups governed by this ADR. Primary subject of the governance.
- [`risks/master-data-risk-register.md`](../risks/master-data-risk-register.md) — Sprint-1-Closeout P2; tracks 18 MDR risks with their own 5-state lifecycle (Hypothesis / Identified / In-Mitigation / Mitigated / Resolved). 7 explicit MDR↔OQ couplings already exist; the MDR lifecycle could in future be unified with OQ governance into a shared Decision-Lifecycle ADR (see §4.3 revisit triggers).
- [`domain/aggregates/catalog.md`](../domain/aggregates/catalog.md), [`sample.md`](../domain/aggregates/sample.md), [`customer-account.md`](../domain/aggregates/customer-account.md), [`documents-compliance.md`](../domain/aggregates/documents-compliance.md), [`pricing-contracts.md`](../domain/aggregates/pricing-contracts.md) — OQ source-of-record per aggregate (8 OQs each in §6 Open Architecture Questions).
- [`workflow.md`](../workflow.md) — operational protocol. Explicit distinction: this ADR governs the lifecycle of architectural decisions (meta-architectural); `workflow.md` governs the operational collaboration protocol (meta-operational). The two are complementary, not overlapping.
- *Forward*: Sprint-2 and Sprint-3 OQ resolutions follow this governance. The first OQ → ADR promotion (Group 1 SSO-V1 or Group 3 Push-vs-Pull, whichever discovery completes first) will be the operational test of Rule 4 + Rule 6.
