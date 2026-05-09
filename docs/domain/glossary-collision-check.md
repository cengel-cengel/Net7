# Glossary Collision Check — AOT Procurement Platform

## 1. Purpose

This document is the verification artefact for the Sprint-1 Exit Criterion *"ubiquitous language is stable and documented"* per `ROADMAP.md`. It audits 12 high-frequency terms across all Sprint-1 corpus documents and reports, per term, whether the same word carries the same meaning everywhere. P6 of the Sprint-1-Closeout-Rework series — the eighth and last of the eight mandatory closeout artefacts.

The audit is a **verification artefact**, not a remediation PR. Drift findings (if any) and gap findings produce concrete recommendations for follow-up PRs (see §5); the recommendations are **not** applied inline here, so that the Audit-Report retains its evidential character.

**Sprint-1 Closeout context** — by P6 the closeout series has produced: P1 [`open-questions-master.md`](./open-questions-master.md), P2 [`../risks/master-data-risk-register.md`](../risks/master-data-risk-register.md), P3 `workflow.md` v2 (PR #17), P4 [`adapter-topology.md`](./adapter-topology.md), P5a [`../adr/0002-domain-patterns.md`](../adr/0002-domain-patterns.md), P5b [`../adr/0003-erp-boundary-rules.md`](../adr/0003-erp-boundary-rules.md), P5c [`../adr/0004-open-question-governance.md`](../adr/0004-open-question-governance.md). P6 closes the series.

## 2. Methodology

- **Carlos-mandatory term set (6)**: *Customer*, *Account*, *Contract*, *Sample*, *Document*, *Order*. Audited regardless of glossary status.
- **Sprint-1-Emergent term set (6)**: *Product*, *Batch*, *User*, *Status*, *Reference*, *Workspace*. Selected for high frequency across the corpus or known-polysemy potential.
- **Source coverage (14 documents)** — all Sprint-1 corpus material:
  - Domain (10): [`glossary.md`](./glossary.md) (canonical baseline), [`bounded-contexts.md`](./bounded-contexts.md), [`capability-map.md`](./capability-map.md), [`aggregates/catalog.md`](./aggregates/catalog.md), [`aggregates/sample.md`](./aggregates/sample.md), [`aggregates/customer-account.md`](./aggregates/customer-account.md), [`aggregates/documents-compliance.md`](./aggregates/documents-compliance.md), [`aggregates/pricing-contracts.md`](./aggregates/pricing-contracts.md), [`open-questions-master.md`](./open-questions-master.md), [`adapter-topology.md`](./adapter-topology.md).
  - Risk (1): [`../risks/master-data-risk-register.md`](../risks/master-data-risk-register.md).
  - ADR (3): [`../adr/0002-domain-patterns.md`](../adr/0002-domain-patterns.md), [`../adr/0003-erp-boundary-rules.md`](../adr/0003-erp-boundary-rules.md), [`../adr/0004-open-question-governance.md`](../adr/0004-open-question-governance.md).
- **Per-term schema**:
  - **Glossary canonical** — definition from `glossary.md`, or `no standalone entry` when missing.
  - **Uses inventory** — where the term appears, with brief context per occurrence.
  - **Verdict** — one of: `Consistent` / `Drift detected` / `Polysemy intentional` / `Action needed`.
  - **Recommendation** — concrete glossary update, sketch correction, or disambiguation guidance, where applicable.
- **Drift vs Gap distinction (methodological)**:
  - **Drift** = the same term carries conflicting meanings across documents. Severity HIGH; structurally damages the ubiquitous-language goal.
  - **Gap** = the term is used pervasively but has no canonical definition in `glossary.md`. Severity LOW; the meaning is stable (implicit via aggregate sketches), but glossary completeness is incomplete.
  - This audit reports both; the §4 Summary distinguishes them.

## 3. Findings

### 3.1 Customer

- **Glossary canonical**: no standalone entry. `Customer-ID` and `Customer/Supplier` (DDD relationship type) exist.
- **Uses inventory**: pervasive across the corpus. `bounded-contexts.md` §4.g; `capability-map.md` §3.g (CAP-022 … CAP-024); `aggregates/customer-account.md` (entire aggregate — `CustomerAccount` Root, `CustomerId` VO, `CompanyName`, `TaxStatus`, `ContractStatus`, default addresses); `aggregates/pricing-contracts.md` (`CustomerContract` Root, `CustomerReference` to Customer Account); `aggregates/sample.md` `CustomerReference`; `aggregates/documents-compliance.md` (entitlement scope notes); `adr/0002` Pattern 2 / Pattern 4 evidence; `adr/0003` Rules 1, 4; `adr/0004` Rule 5 Group 1; `open-questions-master.md` OQ-014, OQ-017, OQ-020, OQ-022, OQ-036; `master-data-risk-register.md` MDR-013, MDR-014; `adapter-topology.md` §2 + §4.
- **Verdict**: **Action needed** — pervasive use (15+ surfaces) with no standalone glossary entry. Implicit meaning is stable (the B2B-customer organisation purchasing from AOT, single source of truth in NET7), but the canonical definition is missing.
- **Recommendation**: add a `Customer` entry to `glossary.md` §3 Domain group: *"B2B customer organisation purchasing from AOT. Source of truth in NET7. Web-side Aggregate-Root is `CustomerAccount` (see `customer-account.md`)."*

### 3.2 Account

- **Glossary canonical**: no entry.
- **Uses inventory**: `Account` appears almost exclusively as the second token in the compound term *"Customer Account"* — bounded-context name (`bounded-contexts.md` §4.g), aggregate name (`aggregates/customer-account.md`), Aggregate-Root entity (`CustomerAccount`), and Net7-adapter name (`Net7CustomerAdapter` is named after the customer-account boundary). No standalone `Account` usage that would require an independent definition.
- **Verdict**: **Polysemy intentional** — `Account` is a compound-term modifier, not an independent noun in this corpus. The compound term `Customer Account` is well-defined via `customer-account.md`.
- **Recommendation**: optional cleanup — if a `Customer` entry is added (per §3.1), include a parenthetical *"(also: Customer Account, the bounded-context and aggregate)"* hint to disambiguate. Not blocking.

### 3.3 Contract

- **Glossary canonical**: no standalone entry. The term appears inside the definitions of `Pricing Tier` (the bounded-context name `Pricing & Contracts`) and `MOQ` (which is set by *contract terms*).
- **Uses inventory**: `aggregates/pricing-contracts.md` (`CustomerContract` Aggregate-Root, `ContractId`, `ContractStatus`, `ContractValidity`, `PaymentTerms`, "contract terms"); `aggregates/customer-account.md` (`ContractStatus` mention in §4.g cross-references and §6 ID-references); `adr/0003` Rule 1 (single source of truth); `adr/0003` Rule 6 (`ContractStatus` read-only-mirror); `open-questions-master.md` OQ-035 (CP-4 Pricing Aggregate-Boundary, single-vs-two-aggregate split); `master-data-risk-register.md` (Pricing-related risks reference contract context); `adapter-topology.md` §2 `Net7PricingAdapter` + §3 cache strategy.
- **Verdict**: **Action needed** — pervasive use without a standalone glossary entry. Risk: latent polysemy between `CustomerContract` (the entity) and *"contract terms"* (the general legal-agreement concept) is not disambiguated.
- **Recommendation**: add two entries to `glossary.md`:
  - **`Contract`** — *"Legal agreement between AOT and a Customer granting access to a product set with negotiated pricing tier, payment terms, and lead time. Source of truth in NET7."*
  - **`CustomerContract`** — *"Aggregate-Root entity in `pricing-contracts.md` modelling the Contract on the web side; carries `ContractStatus` and binds `ProductPrice` set to a single Customer."*

### 3.4 Sample

- **Glossary canonical**: explicit polysemy with three distinct entries — `Sample` (the physical material), `Sample Request` (the digital ask), `Sample Order` (the fulfilment record).
- **Uses inventory**: `aggregates/sample.md` (entire aggregate built around `SampleRequest` Root, `SampleStatus`, `SampleOrder` hypothesis component, `DecisionFeedback`, `ConversionOutcome`); `master-data-risk-register.md` MDR-015 (Missing batch refs on shipped samples), MDR-016 (Sample-COA missing), MDR-017 (Sample-to-product cross-reference broken); `capability-map.md` §4 CC-1 Sample-to-RFQ Conversion; `bounded-contexts.md` §4.d; `capability-map.md` §3.d (CAP-011 … CAP-014); `adr/0002` Pattern 4 (sample as Write-path Adapter source); `adr/0003` Rules 3, 6; `open-questions-master.md` Group 5 Cross-Aggregate Conversion-Path; `adapter-topology.md` Sample node + Net7SampleAdapter.
- **Verdict**: **Consistent** — explicit polysemy is intentional and properly disambiguated in glossary. The 3-term triplet (Sample / Sample Request / Sample Order) is exemplary for stable ubiquitous-language design across this corpus.
- **Recommendation**: none. `Sample` is the gold-standard pattern other terms should follow (see §3.5 Document, where the same triplet treatment is incomplete).

### 3.5 Document

- **Glossary canonical**: partial. The `DocumentType` instances are well-defined as standalone entries — `SDS`, `COA`, `TDS`, `Allergen Statement`, `Bio-Zertifikat`, `Kosher / Halal`, `Certificate`. But `Document` (the abstract Aggregate-Root) and `DocumentVersion` (versioning Entity) have no standalone entries.
- **Uses inventory**: `aggregates/documents-compliance.md` (Aggregate-Root `Document`, `DocumentVersion`, `DocumentStatus`, `DocumentScope`, `DocumentLanguage`, `DocumentFileReference`, `RequiredDocumentSet`); `master-data-risk-register.md` MDR-005 (Broken document links), MDR-006 (Language gaps), MDR-007 (Expired certificates), MDR-008 (Allergen statement missing), MDR-009 (Versioning conflicts); `capability-map.md` §3.c (CAP-008 … CAP-010); `capability-map.md` §4 CC-3 Document Bulk Download per Workspace; `adr/0002` Pattern 3 (read-only-mirror notation source); `adr/0003` Rules 2, 7; `open-questions-master.md` OQ-025 … OQ-032 (8 OQs in Documents context); `adapter-topology.md` Documents node + Net7DocumentAdapter.
- **Verdict**: **Action needed** — `Document` and `DocumentVersion` are pervasive entity-terms without canonical glossary entries. The Type-instances are well-defined; the abstract container is not. This is the *Sample* triplet treatment incompletely applied to Documents.
- **Recommendation**: add two entries:
  - **`Document`** — *"Abstract regulatory or technical artifact (SDS, COA, TDS, Bio-Zertifikat, Allergen Statement, Kosher / Halal, etc.) attached to a Product, Batch, Order, or Sample. Source of truth in NET7. Aggregate-Root in `documents-compliance.md`."*
  - **`DocumentVersion`** — *"Identity-bearing version of a Document. Versions are append-only; supersession is a state-machine transition (per `documents-compliance.md` `INV-3`). Hypothesis: Entity vs Value Object pending OQ-025."*

### 3.6 Order

- **Glossary canonical**: `Order` and `Order Line` exist as standalone entries; `Sample Order` is a separate entry under the Sample triplet (§3.4).
- **Uses inventory**: `bounded-contexts.md` §4.f; `capability-map.md` §3.f (CAP-018 … CAP-021); `aggregates/customer-account.md` `OrderReference` (out-of-scope ghost); `aggregates/pricing-contracts.md` `OrderReference`; `aggregates/documents-compliance.md` `OrderReference`; `adr/0003` Rules 5, 7; `open-questions-master.md` CC-2 *Reorder via Order History*, OQ-044 (CAP-018 quote-to-order); `master-data-risk-register.md` MDR-018 (sync-lag direct manifestation); `adapter-topology.md` Order ghost node.
- **Verdict**: **Consistent** — `Order` is well-defined; the cross-aggregate `OrderReference` pattern is consistent across the four aggregates that reference it; `Sample Order` is explicitly distinguished by its own glossary entry. No drift, no gap.
- **Recommendation**: none.

### 3.7 Product (Sprint-1-Emergent)

- **Glossary canonical**: no standalone entry. Many Product-related attributes have entries — `Variant`, `Specification`, `Origin`, `Application`, `Botanical Name`, `INCI`, `CAS`, `Allergen Statement`, `Certificate` — but the `Product` term itself does not.
- **Uses inventory**: most-referenced entity in the corpus (50+ surfaces). `aggregates/catalog.md` (Aggregate-Root `Product`, `ProductIdentifier`, `ProductVariant`); `aggregates/pricing-contracts.md` (`ProductPrice` entity, `ProductReference` to Catalog); `aggregates/sample.md` `SampleProductReference`; `aggregates/documents-compliance.md` `ProductReference`; all three ADRs cite Product evidence; all 18 MDR risks at least implicitly involve Product; `adapter-topology.md` Catalog node centred on Product; `bounded-contexts.md` §4.a; `capability-map.md` §3.a (CAP-001 … CAP-005).
- **Verdict**: **Action needed** — most-referenced entity in the entire corpus, no standalone glossary entry. Implicit definition lives in `catalog.md`.
- **Recommendation**: add a `Product` entry: *"AOT raw-material item identified by `ProductIdentifier` in NET7. Web-side Aggregate-Root in `catalog.md`. Cross-references: `Variant` (different configurations of the same Product), `Specification` (technical attributes), `INCI` / `CAS` / `Botanical Name` (identification facets)."*

### 3.8 Batch (Sprint-1-Emergent)

- **Glossary canonical**: `Batch` entry exists, with `Lot` as a synonym (`Lot` has its own entry pointing to `Batch`).
- **Uses inventory**: `aggregates/documents-compliance.md` `BatchReference` (per-batch COA placement, gated by OQ-027); `aggregates/pricing-contracts.md` (no BatchReference, pricing is per-product not per-batch); `aggregates/sample.md` (sample-COA references Batch implicitly); `master-data-risk-register.md` MDR-015 (Missing batch references on shipped samples — Carlos-mandatory risk); `open-questions-master.md` CP-6 OQ-027 BatchReference Placement, Group 2 BatchReference Coupling; `adr/0002` Pattern 2 cross-aggregate INV-tag evidence; `adapter-topology.md` Inventory ghost node carrying the OQ-027 trigger.
- **Verdict**: **Polysemy intentional + disambiguation needed** — `Batch` as the domain-concept entry is solid. The `BatchReference` cross-aggregate-ID pattern is heavily used but not separately defined; the placement of Batches (Catalog `ProductVariant` vs an own Inventory aggregate per OQ-027) is not yet decided, which colours the term with provisional polysemy.
- **Recommendation**: add a `BatchReference` entry to `glossary.md`: *"Cross-aggregate ID-reference to a Batch. Used by Documents (per-batch COA), Sample (shipped-batch traceability), and Order (batch assignment per line). Target placement of Batch is open per OQ-027 (Catalog `ProductVariant` vs new Inventory aggregate)."* Keep `Batch` as the domain term.

### 3.9 User (Sprint-1-Emergent)

- **Glossary canonical**: no entry.
- **Uses inventory**: `aggregates/customer-account.md` (`User` entity, child of `CustomerAccount`; `UserId`, `UserRole`, `UserStatus`, `UserPreferences`, `AuthenticationCredential`); `adr/0002` Pattern 2 security-boundary evidence (`AuthenticationCredential`); `adr/0003` Rule 4 Identity-Bridging Boundary; `adr/0004` Rule 3 Owner-Authority distinguishes Carlos / TopM-Eng / Customer-PM / AOT-Sales-QM; `open-questions-master.md` Group 1 SSO-Trio (OQ-017 User-as-Root question), Group 4 Multi-Tenant Cluster; `capability-map.md` §3.g (CAP-022 … CAP-024).
- **Verdict**: **Action needed** — `User` is a first-class entity in the customer-account aggregate without a glossary entry. The User vs Customer distinction (User = individual person; Customer = organisation) is critical for the V1 single-tenant decision and is not glossary-stable.
- **Recommendation**: add a `User` entry: *"Individual human authenticated against the AOT platform. Child entity of `CustomerAccount` per V1 single-tenant decision (`customer-account.md` `INV-2`). Carries a `UserRole` from the `AGENTS.md` persona vocabulary (Strategic / Tactical / Mixed Buyer / Product Developer / Quality Manager). User-vs-Customer distinction: User = person; Customer = organisation."*

### 3.10 Status (Sprint-1-Emergent)

- **Glossary canonical**: no standalone entry. Specific Status types are mentioned inside other entries (e.g., `Sample` references `SampleStatus`).
- **Uses inventory**: pervasive — `SampleStatus` (`aggregates/sample.md`), `DocumentStatus` (`documents-compliance.md`), `ContractStatus` and `PriceValidity` (`pricing-contracts.md`), `UserStatus` and `ContractStatus` (`customer-account.md`); `adr/0002` Pattern 2 *state-machine* sub-class + Pattern 3 *bidirectional vs read-only-mirror* notation variants; `adr/0003` Rule 6 Status Sync Mechanism; `adr/0004` Rule 1 (OQ-Lifecycle Status `Open` → `Closed`); `open-questions-master.md` Group 3 *Status-Sync Mechanism Family* (CP-5); all five `Net7*Adapter` profiles in `adapter-topology.md` carry status-sync responsibility.
- **Verdict**: **Polysemy intentional** — `Status` is a meta-term; each per-aggregate `XStatus` carries its own state machine. Polysemy is structurally enforced by `adr/0002` Pattern 3 notation variants (bidirectional vs read-only-mirror); no actual drift detected.
- **Recommendation**: add a `Status` meta-term entry: *"Per-aggregate state-machine value. Notation variants: bidirectional (web can transition locally — e.g., `SampleStatus`, `UserStatus`) vs read-only-mirror (web mirrors NET7-driven transitions — e.g., `DocumentStatus`, `ContractStatus`, `PriceValidity`) per `adr/0002` Pattern 3."*

### 3.11 Reference (Sprint-1-Emergent)

- **Glossary canonical**: no entry.
- **Uses inventory**: pervasive as the `XReference` cross-aggregate-ID pattern. `ProductReference`, `CustomerReference`, `DocumentReference`, `OrderReference`, `RFQReference`, `SampleReference`, `BatchReference`, `WorkspaceReference`, `DeliveryAddressReference`, `PriceReference` — distributed across all five aggregate sketches' §6 sections; consolidated in `adapter-topology.md` Diagram 2 (23 cross-references, 13 in-scope solid + 10 out-of-scope dashed); cited in `adr/0002` Pattern 1 §6 Cross-Aggregate-ID-References convention.
- **Verdict**: **Polysemy intentional** — `Reference` is a structural pattern (cross-aggregate-ID-pattern per `adr/0002` Pattern 1 §6). The naming convention `<Source>Reference` is consistent across all 23 cross-references; no drift.
- **Recommendation**: add a `Reference` cross-cutting pattern entry: *"Cross-aggregate ID-reference per `adr/0002` Pattern 1 (§6 `Cross-Aggregate-ID-References` convention). Naming: `<Source>Reference` (e.g., `ProductReference`). Aggregates reference each other by ID only; no object navigation crosses aggregate boundaries."*

### 3.12 Workspace (Sprint-1-Emergent)

- **Glossary canonical**: `Workspace` entry exists.
- **Uses inventory**: `bounded-contexts.md` §4.h; `capability-map.md` §3.h (CAP-025, CAP-026); `aggregates/customer-account.md` `WorkspaceReference`; `aggregates/documents-compliance.md` `WorkspaceReference` (CC-3 Bulk Download per Workspace, P3 / Future); `open-questions-master.md` CC-3, Group 2 (BatchReference impacts); `adapter-topology.md` Workspaces ghost node (P2 out-of-scope); `adr/0002` Pattern 4 framing (Workspaces is a Web-only future aggregate).
- **Verdict**: **Consistent** — Workspace is well-defined as a Web-only artifact; consistently treated as out-of-scope ghost in `adapter-topology.md` Diagram 2; CC-3 P3/Future framing is uniform across docs.
- **Recommendation**: none.

## 4. Summary

**Total terms checked**: 12 (6 Carlos-mandatory + 6 Sprint-1-Emergent).

**Verdict distribution**:

- **Consistent** (3): Sample (§3.4), Order (§3.6), Workspace (§3.12).
- **Action needed** (5): Customer (§3.1), Contract (§3.3), Document (§3.5), Product (§3.7), User (§3.9).
- **Polysemy intentional** (4): Account (§3.2 — compound-term), Batch (§3.8 — `BatchReference` pattern needs entry), Status (§3.10 — meta-term), Reference (§3.11 — cross-cutting pattern).
- **Drift detected**: 0.

**Drift vs Gap distinction (methodological)**: this audit reports zero **drift** findings (no conflicting definitions across documents) and five **gap** findings (terms used pervasively without a canonical `glossary.md` entry). Gap severity is LOW because the implicit meaning is stable in every case — each gap-term has a clear definition via its Aggregate-Root in the relevant aggregate sketch. The issue is glossary completeness, not semantic drift.

**Sprint-1 Exit Criterion *"ubiquitous language is stable"***: **Achieved with caveats**.

- *Stable*: yes. No drift; no conflicting definitions; the 12 audited terms carry the same meaning everywhere they are used.
- *Caveats*: 5 of the 6 Carlos-mandatory terms (Customer, Contract, Document) plus 2 Sprint-1-Emergent terms (Product, User) lack standalone `glossary.md` entries. Their meaning is stable, but the glossary surface area is incomplete. 4 polysemy-intentional terms (Account, Batch, Status, Reference) would benefit from disambiguation guidance entries.

## 5. Recommendations

The recommendations below are **not applied in this PR** — P6 is a verification artefact, not remediation. The proposed follow-up PR is `sprint-1-closeout/glossary-completeness-additions`, scoped to a single glossary update.

**Glossary additions — standalone entries (5)**:

- `Customer` (per §3.1)
- `Contract` and `CustomerContract` (per §3.3)
- `Document` and `DocumentVersion` (per §3.5)
- `Product` (per §3.7)
- `User` (per §3.9)

**Glossary additions — meta / pattern entries (3)**:

- `BatchReference` (per §3.8)
- `Status` meta-term (per §3.10)
- `Reference` cross-cutting pattern (per §3.11)

**Glossary disambiguations (optional)**:

- `Customer` entry includes a parenthetical *"(also: `Customer Account`)"* hint (per §3.2).

**Total estimated**: 8 standalone/meta entries + 1 disambiguation hint, in one follow-up PR after the Sprint-1-Closeout-ADR-Triplet (PR #19, #20, #21) lands on `main`.

**Triage strategy** — single follow-up PR (not inline) preserves the verification character of P6 and avoids interleaving audit + remediation in the same artefact. The follow-up PR can be raised in Sprint-1-Closeout-Postscript or as the first task of Sprint 2 *ERP Boundary Design* preparation.

## 6. References

- [`glossary.md`](./glossary.md) — canonical definition baseline (55 entries).
- [`bounded-contexts.md`](./bounded-contexts.md) — eight bounded contexts; Customer / Sample / Document / Order context-name uses.
- [`capability-map.md`](./capability-map.md) — 26 capabilities + 3 cross-context; per-capability term usage.
- [`aggregates/catalog.md`](./aggregates/catalog.md) — Product Aggregate-Root.
- [`aggregates/sample.md`](./aggregates/sample.md) — Sample Lifecycle, Sample triplet exemplar.
- [`aggregates/customer-account.md`](./aggregates/customer-account.md) — Customer / Account / User / AuthenticationCredential.
- [`aggregates/documents-compliance.md`](./aggregates/documents-compliance.md) — Document Aggregate-Root, DocumentVersion gap.
- [`aggregates/pricing-contracts.md`](./aggregates/pricing-contracts.md) — Contract / CustomerContract gap.
- [`open-questions-master.md`](./open-questions-master.md) — Sprint-1-Closeout P1; OQ-IDs cited per term.
- [`adapter-topology.md`](./adapter-topology.md) — Sprint-1-Closeout P4; Reference pattern visualisation (23 cross-references).
- [`../risks/master-data-risk-register.md`](../risks/master-data-risk-register.md) — Sprint-1-Closeout P2; per-term risk mentions (e.g., MDR-015 Sample/Batch).
- [`../adr/0002-domain-patterns.md`](../adr/0002-domain-patterns.md) — Sprint-1-Closeout P5a; Pattern 1, 2, 3 referenced for term-classification.
- [`../adr/0003-erp-boundary-rules.md`](../adr/0003-erp-boundary-rules.md) — Sprint-1-Closeout P5b; Rule 4 Identity-Bridging cites `User`, Rule 6 cites `Status`.
- [`../adr/0004-open-question-governance.md`](../adr/0004-open-question-governance.md) — Sprint-1-Closeout P5c; Rule 5 Coupled-Group references cite `User` (Group 1, 4) and `Status` (Group 3).
- [`../workflow.md`](../workflow.md) — operational protocol; relevant only for the workflow-vs-architectural-decision distinction (P6 is a domain-content audit, not an operational protocol).
- [`../../AGENTS.md`](../../AGENTS.md) — engineering charter; Sprint-1 Exit Criterion lives in `ROADMAP.md` but is motivated by the `AGENTS.md` handover-readiness goal.
- *Future*: Sprint 2 introduces new terms (RFQ Lifecycle, Order Management aggregate-internal terms, possibly Inventory if `OQ-027` resolves). Re-audit recommended after RFQ / Order / Workspaces sketches land — tentatively *Sprint-2-Closeout P-X glossary-collision-check-v2*.
