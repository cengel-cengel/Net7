# Open Questions Master Triage — AOT Procurement Platform

## 1. Purpose

This document is the single decision-cockpit for all 46 open questions surfaced across the Sprint-1 *Domain Modeling* deliverables — five aggregate sketches, the capability map, and the bounded-contexts overview. Without consolidation, the same questions sit in five separate documents and lose architectural traceability.

The cockpit feeds the Sprint-2 *ERP Boundary Design* triage and provides a stable input register for TopM-discovery, customer-discovery, and Carlos-side architecture decisions. Per `ROADMAP.md`, Sprint 2's exit criterion (*"ERP boundaries isolated and reviewed"*) depends on resolving the Critical-Path subset of these OQs before adapter contracts are signed off.

## 2. How to Use

- **OQ Lifecycle**: every OQ moves through *Open* → *In Discovery* → *Decided* → *ADR*. Status starts at *Open*; an ADR closes the question and removes it from the active triage. Governance for this lifecycle is the subject of a future `docs/adr/0004-open-question-governance.md` (Sprint-1-Closeout P5c).
- **Coupled Groups**: §4 below identifies five OQ groups whose members must be triaged together. Resolving one in isolation typically forces the others; coupled discovery interviews are more efficient and avoid divergent decisions.
- **Critical-Path**: §5 below identifies the OQs that block Sprint-2 start or substantially gate Sprint-2 adapter-contract design. These are the priority items for the first round of TopM-discovery and customer-discovery.
- **Source traceability**: every row in the §3 triage table carries a `Source` reference (`<aggregate> OQ-N` or `cap-map §5` or `bounded-ctx §6`) for full-text lookup. The triage table holds an abridged question; the source document holds the full discussion and validation hooks.
- **Risk levels**: *Critical* = source-document-flagged or major architectural pivot; *High* = Sprint-2 adapter-contract blocker; *Medium* = architecture-relevant but not blocking adapter sign-off; *Low* = post-V1 / Future scope or already-decided-by-default.
- **Dependency tags**: *TopM* = TopM-side ERP discovery; *Customer* = customer-side discovery interviews; *AOT-Sales* / *AOT-QM* = AOT-internal stakeholder; *Cross-agg* = resolves with other OQs in this register; *Arch (Carlos)* = decision belongs to architect, no external discovery needed.

## 3. Triage Table

46 OQs across 5 aggregates and 2 supplementary documents. Sorted by `Source` (alphabetical by aggregate, then per-aggregate-OQ-number), then capability-map §5 supplementary, then bounded-contexts §6 supplementary.

| OQ ID  | Source              | Aggregate         | Question (abridged)                                                                  | Dependency                  | Risk     | Owner          | Status |
|--------|---------------------|-------------------|--------------------------------------------------------------------------------------|-----------------------------|----------|----------------|--------|
| OQ-001 | catalog OQ-1        | Catalog           | `ProductVariant` inside Product or own aggregate?                                    | TopM §3.c, Cross-agg        | High     | Carlos         | Open   |
| OQ-002 | catalog OQ-2        | Catalog           | `Specification` inside Product or own aggregate?                                     | TopM §3.c                   | High     | Carlos         | Open   |
| OQ-003 | catalog OQ-3        | Catalog           | `Certification` Value Object or Entity (versioning, expiry)?                         | Cross-agg (Documents)       | Medium   | Carlos         | Open   |
| OQ-004 | catalog OQ-4        | Catalog           | `AvailabilityIndicator` placement — Catalog vs Pricing vs new Inventory aggregate?   | TopM §3.d, Cross-agg        | Medium   | Carlos         | Open   |
| OQ-005 | catalog OQ-5        | Catalog           | Search-index synchronisation — push from NET7 vs scheduled pull?                     | TopM §3.f                   | High     | TopM-Eng       | Open   |
| OQ-006 | catalog OQ-6        | Catalog           | `ProductVariant` lifecycle independent of Product (deactivate variant only)?         | TopM §3.c                   | Medium   | TopM-Eng       | Open   |
| OQ-007 | catalog OQ-7        | Catalog           | Multilingual product names — web-side translation layer vs ERP-sourced?              | TopM §3.c, AOT-Sales        | Medium   | Carlos         | Open   |
| OQ-008 | catalog OQ-8        | Catalog           | `Category` taxonomy — web-defined facets vs ERP-sourced taxonomy?                    | TopM §3.c, Customer §3.b    | Medium   | Carlos         | Open   |
| OQ-009 | sample OQ-1         | Sample            | `DecisionFeedback` inside Sample or own aggregate?                                   | TopM §3.e, Customer §3.d    | Medium   | AOT-Sales      | Open   |
| OQ-010 | sample OQ-2         | Sample            | Multi-product per `SampleRequest` vs one-sample-per-product?                         | TopM §3.c, §3.e             | Medium   | TopM-Eng       | Open   |
| OQ-011 | sample OQ-3         | Sample            | `ApplicationProject` VO inside Sample or cross-ref to Workspaces?                    | Customer §3.b, §3.h         | Medium   | Customer-PM    | Open   |
| OQ-012 | sample OQ-4         | Sample            | `SampleStatus` state machine ERP-defined or web-modelled with mapping?               | TopM §3.c, §3.f             | High     | TopM-Eng       | Open   |
| OQ-013 | sample OQ-5         | Sample            | Status synchronisation — push from NET7 vs scheduled pull?                           | TopM §3.f                   | High     | TopM-Eng       | Open   |
| OQ-014 | sample OQ-6         | Sample            | `ConversionOutcome` (CC-1) — web orchestrates two ERP writes or NET7-native?         | TopM §3.e                   | Critical | TopM-Eng       | Open   |
| OQ-015 | sample OQ-7         | Sample            | `DeliveryAddress` override — per-request optional or always default?                 | Customer §3.d, TopM §3.e    | Medium   | AOT-Sales      | Open   |
| OQ-016 | sample OQ-8         | Sample            | `SampleRequest` visibility — user-bound or shared within customer-org?               | Customer §3.b               | Low      | Customer-PM    | Open   |
| OQ-017 | cust-acc OQ-1       | Customer Account  | `User` as child entity inside CustomerAccount or own aggregate?                      | Customer §3.b, Cross-agg    | Critical | Carlos         | Open   |
| OQ-018 | cust-acc OQ-2       | Customer Account  | `AuthenticationCredential` inside Customer Account or own Security aggregate?        | Arch (Carlos), Cross-agg    | High     | Carlos         | Open   |
| OQ-019 | cust-acc OQ-3       | Customer Account  | Multi-tenant scope — V1 single-tenant or multi-tenant from V1?                       | Customer §3.b               | Low      | Carlos         | Open   |
| OQ-020 | cust-acc OQ-4       | Customer Account  | `UserId` source — NET7-issued or web-issued?                                         | TopM §3.b, §3.c             | High     | TopM-Eng       | Open   |
| OQ-021 | cust-acc OQ-5       | Customer Account  | User-profile updates — read-only ERP-mirror or write-back via adapter?               | TopM §3.e                   | High     | TopM-Eng       | Open   |
| OQ-022 | cust-acc OQ-6       | Customer Account  | `ContractStatus` + `UserStatus` synchronisation — push or pull?                      | TopM §3.f                   | High     | TopM-Eng       | Open   |
| OQ-023 | cust-acc OQ-7       | Customer Account  | SSO / SAML / OIDC integration — V1 native or post-V1?                                | Customer §3.i               | High     | Customer-PM    | Open   |
| OQ-024 | cust-acc OQ-8       | Customer Account  | `UserRole` mapping — per-User intrinsic or per-CustomerAccount-User-mapping?         | Customer §3.b               | Medium   | Customer-PM    | Open   |
| OQ-025 | docs OQ-1           | Documents         | `DocumentVersion` Entity (versioning) or Value Object?                               | TopM §3.c                   | Medium   | TopM-Eng       | Open   |
| OQ-026 | docs OQ-2           | Documents         | `RequiredDocumentSet` placement — Documents aggregate or Catalog (cross-agg impact)? | Customer §3.f, TopM §3.d    | Critical | Carlos         | Open   |
| OQ-027 | docs OQ-3           | Documents         | `BatchReference` placement — Catalog `ProductVariant` or own Inventory aggregate?    | TopM §3.c, Cross-agg        | High     | Carlos         | Open   |
| OQ-028 | docs OQ-4           | Documents         | `DocumentLanguage` fallback — EN if DE absent or strict per-locale?                  | AOT-QM §5.a                 | Medium   | AOT-QM         | Open   |
| OQ-029 | docs OQ-5           | Documents         | `DocumentFileReference` semantics — stable URI vs signed/short-lived link?           | TopM §3.b, §3.h             | High     | TopM-Eng       | Open   |
| OQ-030 | docs OQ-6           | Documents         | Caching strategy — short-TTL vs hard-TTL with expiry-aware invalidation?             | Arch (Carlos)               | High     | Carlos         | Open   |
| OQ-031 | docs OQ-7           | Documents         | Bulk-download (CC-3) — streaming-ZIP vs pre-bundled vs scheduled async?              | TopM §3.h, Customer §3.f    | Low      | Carlos         | Open   |
| OQ-032 | docs OQ-8           | Documents         | Document search index — per-Document or per-Product (with doc-set as facet)?         | Arch (Carlos)               | Medium   | Carlos         | Open   |
| OQ-033 | pricing OQ-1        | Pricing           | `ProductPrice` Entity (versioning + scale-tiers) or Value Object?                    | TopM §3.c                   | High     | TopM-Eng       | Open   |
| OQ-034 | pricing OQ-2        | Pricing           | `PricingScale` VO collection or Sub-Entity for tier-based pricing?                   | TopM §3.c                   | Medium   | TopM-Eng       | Open   |
| OQ-035 | pricing OQ-3        | Pricing           | Single aggregate vs two-aggregate split (Contract + ProductPrice)?                   | TopM §3.c, §3.d             | Critical | Carlos         | Open   |
| OQ-036 | pricing OQ-4        | Pricing           | Multi-currency support — per-Contract or per-Price granularity?                      | TopM §3.c                   | High     | TopM-Eng       | Open   |
| OQ-037 | pricing OQ-5        | Pricing           | `ContractStatus` + `PriceValidity` synchronisation — push or pull?                   | TopM §3.f                   | High     | TopM-Eng       | Open   |
| OQ-038 | pricing OQ-6        | Pricing           | Future-dated prices (scheduled changes) — V1 or post-V1?                             | Customer §3.g, TopM §3.c    | Low      | Customer-PM    | Open   |
| OQ-039 | pricing OQ-7        | Pricing           | Price-history audit (expired prices retained) — V1 or post-V1?                       | AOT-QM §5.b                 | Low      | AOT-QM         | Open   |
| OQ-040 | pricing OQ-8        | Pricing           | `PricingTier` ownership — AOT-defined or NET7-defined enum?                          | TopM §3.c, §3.e             | High     | TopM-Eng       | Open   |
| OQ-041 | cap-map §5          | Catalog (CAP-004) | Compare priority — P1 vs P2 (procurement-hypotheses §6 vs §9 conflict)?              | Customer §3.b               | Medium   | Customer-PM    | Open   |
| OQ-042 | cap-map §5          | Pricing (CAP-006) | Pricing read latency — synchronous per-request vs cached short-TTL snapshot?         | TopM §3.h                   | High     | TopM-Eng       | Open   |
| OQ-043 | cap-map §5          | RFQ (CAP-015)     | RFQ submission — does NET7 expose a quote-creation API today?                        | TopM §3.e                   | High     | TopM-Eng       | Open   |
| OQ-044 | cap-map §5          | Order (CAP-018)   | Order placement — quote-to-order is ERP-native or web-orchestrated?                  | TopM §3.e                   | High     | TopM-Eng       | Open   |
| OQ-045 | bounded-ctx §6      | Workspaces        | Workspaces really web-only — could TopM later introduce a "project" entity?          | TopM §3.d                   | Medium   | TopM-Eng       | Open   |
| OQ-046 | bounded-ctx §6      | Documents         | Documents context — regulatory-statement generation as own sub-context?              | Arch (Carlos)               | Medium   | Carlos         | Open   |

**Risk distribution**: 4 Critical / 20 High / 17 Medium / 5 Low = 46.
**Owner distribution**: 18 Carlos / 19 TopM-Eng / 5 Customer-PM / 2 AOT-Sales / 2 AOT-QM = 46.

## 4. Coupled OQ Groups

Five groups whose members should be triaged together. Resolving any one in isolation typically forces a re-decision on the others.

### Group 1 — User / Auth / SSO Placement Trio

- **Members**: OQ-017 (cust-acc OQ-1, User as child entity), OQ-018 (cust-acc OQ-2, AuthenticationCredential placement), OQ-023 (cust-acc OQ-7, SSO / SAML / OIDC scope).
- **Coupling rationale**: an SSO-V1 decision pulls `AuthenticationCredential` toward its own Security aggregate (`OQ-018` flips), which in turn re-frames `User` as the natural identity-platform entity (`OQ-017` likely flips to *own aggregate*). Conversely, deferring SSO to post-V1 keeps the current Customer-Account-Root + Auth-inside defaults stable. The three questions cannot be resolved independently without producing an inconsistent identity architecture.
- **Resolution path**: combined customer-PM interview on `business-discovery.md` §3.i (Tech-Readiness) plus internal Carlos decision on V1 SSO scope. One decision-event resolves all three.

### Group 2 — BatchReference Placement Coupling

- **Members**: OQ-027 (docs OQ-3, BatchReference placement), OQ-001 (catalog OQ-1, Variant lifecycle independence), OQ-004 (catalog OQ-4, AvailabilityIndicator placement).
- **Coupling rationale**: batches, variants, and availability are domain-tightly-coupled. Promoting `Batch` to its own *Inventory* aggregate forces variant-lifecycle independence (OQ-001 flips toward *independent*) and shifts `AvailabilityIndicator` into the Inventory aggregate (OQ-004 flips). Keeping Batch inside Catalog `ProductVariant` keeps the current defaults but cramps an Inventory concern into a Catalog aggregate.
- **Resolution path**: TopM-engineer interview on `erp-discovery.md` §3.c (Datenmodell) and §3.d (Lese-Operationen) — how does NET7 model batches relative to variants and stock? May trigger a Sprint-1.5 *Inventory aggregate* sketch before Sprint 2.

### Group 3 — Status-Sync Mechanism Family

- **Members**: OQ-005 (catalog OQ-5, search-index sync), OQ-013 (sample OQ-5, status sync), OQ-022 (cust-acc OQ-6, contract + user status sync), OQ-030 (docs OQ-6, caching strategy with sync), OQ-037 (pricing OQ-5, contract + price status sync).
- **Coupling rationale**: a push-vs-pull architecture decision should be uniform across all five `Net7*Adapter` implementations. A hybrid (push for state-change events, pull for bulk data) is plausible, but per-adapter ad-hoc choices would fragment operational concerns (monitoring, backpressure, replay) and inflate implementation cost.
- **Resolution path**: single TopM-engineer interview on `erp-discovery.md` §3.f (Sync-Strategie). One architectural decision for the whole adapter family.

### Group 4 — Identity & Multi-Tenant Cluster

- **Members**: OQ-019 (cust-acc OQ-3, multi-tenant scope), OQ-020 (cust-acc OQ-4, UserId source), OQ-024 (cust-acc OQ-8, UserRole mapping), OQ-016 (sample OQ-8, SampleRequest visibility).
- **Coupling rationale**: V1 single-tenant simplifies UserId-bridging (OQ-020), per-User intrinsic role (OQ-024), and user-bound sample visibility (OQ-016). V1 multi-tenant inverts all three. The cluster lives or dies together.
- **Resolution path**: customer-discovery interview on `business-discovery.md` §3.b (multi-contact handling) — does real B2B reality demand multi-tenant from V1? Combined with Carlos-side scope decision.

### Group 5 — Cross-Aggregate Conversion-Path Family

- **Members**: OQ-014 (sample OQ-6, Sample-to-RFQ conversion), OQ-043 (cap-map §5 / CAP-015, RFQ submission API), OQ-044 (cap-map §5 / CAP-018, Order placement).
- **Coupling rationale**: all three concern multi-step web-orchestration vs ERP-native transitions across aggregate boundaries. If NET7 exposes native sample-to-RFQ, RFQ-creation, and quote-to-order transitions, the web becomes a thin orchestrator. If not, the web carries multi-step state and idempotency burden across all three flows. The architectural answer is consistent.
- **Resolution path**: single TopM-engineer interview on `erp-discovery.md` §3.e (Schreib-Operationen) covering all three transitions.

## 5. Critical-Path OQs for Sprint-2

Six items that block Sprint-2 start or substantially gate Sprint-2 adapter-contract design. These are the priority subset for the first round of TopM-discovery and customer-discovery.

### CP-1 — OQ-017: Customer Account Aggregate-Root

- **Question**: `User` as child entity inside `CustomerAccount` (current default) or own aggregate?
- **Why blocking**: every authenticated capability resolves through `CustomerAccount` for identity-bridging and customer-scope. If the Root flips to `User`, the identity-bridging pattern in `Net7CustomerAdapter` changes shape across the whole adapter family.
- **Gates**: `Net7CustomerAdapter` Sprint-2 implementation; downstream ripples into `Net7PricingAdapter`, `Net7SampleAdapter`, `Net7OrderAdapter`, `Net7DocumentAdapter` identity scoping.

### CP-2 — OQ-014: Sample-to-RFQ Conversion (CC-1)

- **Question**: `ConversionOutcome` — web orchestrates two ERP writes or NET7 exposes a native sample-to-RFQ transition?
- **Why blocking**: determines `Net7SampleAdapter` and the (yet-unsketched) `Net7RfqAdapter` co-design. Affects the *Partnership* classification of `bounded-contexts.md` §5 #5 between Sample and RFQ.
- **Gates**: Sprint-2 `Net7SampleAdapter` adapter contract; affects Sprint-2 RFQ aggregate sketch (deferred from Sprint 1).

### CP-3 — OQ-026: RequiredDocumentSet Placement

- **Question**: `RequiredDocumentSet` placement — Documents aggregate (current default) or Catalog (product metadata)?
- **Why blocking**: a flip to Catalog requires retroactive revision of the Catalog aggregate sketch — `Certification` VO scope and the completeness-logic source-of-truth shift. Catalog vs Documents adapter responsibility split is renegotiated.
- **Gates**: `Net7DocumentAdapter` and `Net7ProductAdapter` adapter-responsibility boundary; possible Sprint-1 `catalog.md` revision.

### CP-4 — OQ-035: Pricing Aggregate-Boundary

- **Question**: Single aggregate (`CustomerContract` as Root) vs two-aggregate split (Contract and ProductPrice as separate aggregates)?
- **Why blocking**: substantially affects `Net7PricingAdapter` query patterns, payload-size profile, and per-customer cache strategy. Two-aggregate split changes the adapter contract from one read endpoint to two.
- **Gates**: `Net7PricingAdapter` Sprint-2 adapter contract.

### CP-5 — Group 3 Status-Sync Mechanism Family (collective)

- **Question**: push-vs-pull architecture for status synchronisation across all five adapters (covers OQ-005, OQ-013, OQ-022, OQ-030, OQ-037).
- **Why blocking**: a per-adapter ad-hoc decision fragments monitoring, backpressure, and replay semantics. The decision is foundational for adapter implementation across the platform.
- **Gates**: every `Net7*Adapter` Sprint-2 implementation. Cannot start adapter coding without this resolved.

### CP-6 — OQ-027: BatchReference Placement (Group 2 lead)

- **Question**: `BatchReference` placement — Catalog `ProductVariant` or own Inventory aggregate?
- **Why blocking**: a flip to Inventory requires a Sprint-1.5 *Inventory aggregate* sketch before Sprint 2 starts (because Inventory becomes a sixth P1 aggregate that needs the same DDD treatment). Coupled to OQ-001 (Variant lifecycle) and OQ-004 (Availability placement) in Group 2.
- **Gates**: possible Sprint-1.5 Inventory aggregate sketch; affects Catalog adapter responsibility split.

## 6. References

- [`aggregates/catalog.md`](./aggregates/catalog.md) — source for OQ-001 … OQ-008.
- [`aggregates/sample.md`](./aggregates/sample.md) — source for OQ-009 … OQ-016.
- [`aggregates/customer-account.md`](./aggregates/customer-account.md) — source for OQ-017 … OQ-024.
- [`aggregates/documents-compliance.md`](./aggregates/documents-compliance.md) — source for OQ-025 … OQ-032.
- [`aggregates/pricing-contracts.md`](./aggregates/pricing-contracts.md) — source for OQ-033 … OQ-040.
- [`bounded-contexts.md`](./bounded-contexts.md) — source for OQ-045, OQ-046; §5 pair-relationships background.
- [`capability-map.md`](./capability-map.md) — source for OQ-041 … OQ-044; §5 capability-level open questions.
- [`glossary.md`](./glossary.md) — terminology baseline used throughout the OQ register.
- [`../erp-discovery.md`](../erp-discovery.md) — TopM-side discovery clusters referenced in the Dependency column.
- [`../business-discovery.md`](../business-discovery.md) — customer-side discovery clusters referenced in the Dependency column.
- [`../../AGENTS.md`](../../AGENTS.md) — engineering charter; ERP Ownership Rules; handover-readiness goals that motivate this triage cockpit.
- [`../../ROADMAP.md`](../../ROADMAP.md) — Sprint 2 *ERP Boundary Design* exit criterion (*"ERP boundaries isolated and reviewed"*) depends on Critical-Path OQ resolution.
- *Future*: `docs/adr/0002` (Sprint-1-Closeout P5a, ADR Index), `docs/adr/0003` (Sprint-1-Closeout P5b, Adapter-Pattern ADR), `docs/adr/0004-open-question-governance.md` (Sprint-1-Closeout P5c) — formalise lifecycle and ADR-promotion path for closed OQs.
