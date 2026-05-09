# Pricing & Contracts Aggregate Sketch — AOT Procurement Platform

## 1. Purpose

This document is the DDD aggregate sketch for the **Pricing & Contracts** bounded context (`bounded-contexts.md` §4.b). It fixes the aggregate root, the entities and value objects inside the consistency boundary, two state machines (`ContractStatus` and `PriceValidity` in *read-only-mirror* form — NET7 owns the transitions), and the Web/ERP ownership at component level. Direct input for Sprint 2 ERP Boundary Design (pricing read API, multi-currency support, future-dated price handling) and Sprint 3 code scaffold (entities, value objects, contract / price aggregation).

This is the **fifth and final aggregate sketch** in the P1 series, completing the Sprint-1 *Domain Modeling* phase per `ROADMAP.md`. Together with [`catalog.md`](./catalog.md) (1.4.1), [`sample.md`](./sample.md) (1.4.2), [`customer-account.md`](./customer-account.md) (1.4.3), and [`documents-compliance.md`](./documents-compliance.md) (1.4.4), it covers the five P1 contexts identified in `bounded-contexts.md` §4 (Catalog, Sample Lifecycle, Customer Account, Documents & Compliance, Pricing & Contracts).

The Pricing & Contracts aggregate is the strictest expression of the platform's ERP ownership posture. Per `AGENTS.md` (ERP Ownership Rules, §3 Hard Rules):

> **NEVER** put pricing, stock, or contract logic in the frontend.
> **NEVER** duplicate ERP business logic in this codebase. ERP is authoritative.

Web is a pure read-only display surface for prices and contracts; no calculation, no derived business logic, no "minor adjustments" happen on the web side. `INV-5` below makes this hard rule structurally enforceable inside the aggregate.

All structural choices in this document are **working hypotheses** until validated by the Sprint-0 ERP-discovery stream (`erp-discovery.md`) and the customer-discovery stream (`business-discovery.md`). No new methodology sub-classes are introduced here — Pricing & Contracts is a pure pattern application that closes Sprint 1.

## 2. Methodology

- **Aggregate** = a cluster of related domain objects treated as a single unit for data changes. One Aggregate Root acts as the only entry point; external code never reaches inside the aggregate by reference.
- **Entity vs Value Object**: an Entity has identity that persists across state changes (e.g., a `CustomerContract` is the same contract even after a status transition from *Active* to *Suspended*; a `ProductPrice` has identity across versions and scale tiers per OQ-1). A Value Object is defined entirely by its attributes (e.g., `UnitPrice` is a `(Currency, amount)` tuple; two equal UnitPrices are interchangeable).
- **Web / ERP / Shared ownership**: per `AGENTS.md` ERP Ownership Rules, NET7 owns master contract and pricing data. *ERP-defined* means the source of truth is NET7. *Shared* and *Web-only* sub-classes (introduced in earlier aggregates) are **not** relevant here — Pricing & Contracts has no Shared and no Web-only components; the aggregate is fully ERP-driven and read-only on the web side. *Security-boundary* and *write-path adapter* shapes from earlier aggregates are also **not** relevant — the AGENTS.md hard rule prohibits any web-side write of pricing data.
- **Invariant typology** — five sub-classes carried over from earlier aggregates; **no new sub-classes are introduced here**: *structural*, *behavioural*, *state-machine* (with two notation variants — bidirectional from `sample.md` and `customer-account.md`, read-only-mirror from `documents-compliance.md`), *cross-aggregate* (first applied as INV-tag in `documents-compliance.md`), *security-boundary* (introduced in `customer-account.md`, not used here). The state-machine *read-only-mirror* notation is applied twice in this aggregate (`INV-2` `ContractStatus`, `INV-7` `PriceValidity`); the *cross-aggregate* INV-tag is applied once (`INV-4`, `ProductPrice` ↔ Catalog Product master).
- **Hypothesis marking**: where ERP capability or domain semantics are unverified, the line carries `Hypothesis:` or `Open question:`. The §6 OQ-N register collects these for follow-up.
- **Cross-aggregate references**: aggregates reference each other **by ID only**. No object navigation crosses an aggregate boundary; reads of related data go through repositories or query services on the consumer side.

## 3. Aggregate Boundary

### Aggregate Root

**CustomerContract** is the Aggregate Root (provisional). Justification: in a B2B procurement context, customer-specific pricing exists only inside a contract. Prices, MOQs, payment terms, lead times, and price validity windows are all properties of a specific contract between AOT and a customer; there is no meaningful "standalone product price" without the customer-contract context. Treating the contract as the umbrella entity matches the way pricing decisions are made on the AOT side and read on the web side (one customer dashboard renders one customer's contracts and the prices they unlock).

This is a non-trivial decision; the alternative (`Contract` and `ProductPrice` as two separate aggregates) is discussed below and tracked as `OQ-3` *(critical, Aggregate-Boundary)*.

### Inside the Aggregate

Components inside the Pricing & Contracts aggregate (transactional consistency boundary). All components are ERP-defined; the aggregate is fully ERP-driven and read-only on the web side. There are no Shared or Web-only components.

- `CustomerContract` — the root entity; identity stable across status transitions.
- `ContractId` — value object wrapping the stable identifier from NET7.
- `ContractStatus` — value object stamped onto the contract (state-machine, read-only-mirror: *Pending → Active → Suspended / Expired / Closed*).
- `ContractValidity` — value object: from-to dates of contract validity.
- `PaymentTerms` — value object: NET 30 / NET 60 / prepayment / etc.
- `LeadTime` — value object: typical delivery time (days or weeks) under this contract.
- `PricingTier` — value object: customer-category that influences pricing logic on the ERP side. *Hypothesis: ownership of the tier vocabulary is open (OQ-8).*
- `ProductPrice` — entity, child of CustomerContract. *Hypothesis: entity for versioning and scale-tier semantics (OQ-1).*
- `UnitPrice` — value object: `(Currency, amount)` tuple.
- `Currency` — value object: ISO currency code.
- `MOQ` — value object: minimum order quantity per product under this contract.
- `PriceValidity` — value object: from-to dates of a specific `ProductPrice`. *Hypothesis: state-machine read-only-mirror in INV-7 (Active → Expired by ValidityEnd date).*
- `PricingScale` — value object: volume-based pricing scale. *Hypothesis: VO collection of `(quantityThreshold, UnitPrice)` tuples (OQ-2).*

The transactional invariants in §4 (INV-1 … INV-7) all hold inside this boundary, including the two read-only-mirror state-machine invariants and the cross-aggregate invariant.

### Outside the Aggregate

Explicitly **not** inside the Pricing & Contracts aggregate:

- `Customer`, `User`, `CustomerId` — owned by the **Customer Account** aggregate; referenced by `CustomerReference`. The contract belongs to a customer (`INV-1`); the customer master data lives elsewhere.
- `Product`, `ProductVariant`, `Specification` — owned by the **Catalog** aggregate; referenced by `ProductReference` from each `ProductPrice`. Catalog is the source of truth for what a product *is*; Pricing & Contracts is the source of truth for what it *costs* to a specific customer.
- `Document` (signed contract documents, contractual amendments, KYC docs) — owned by the **Documents & Compliance** aggregate; referenced by `DocumentReference`. Aligns with `customer-account.md` `DocumentReference` for contract-related documents.
- `Order`, `OrderLine` — owned by the **Order Management** aggregate; referenced by `OrderReference`. Orders consume `ProductPrice`, `MOQ`, `LeadTime`, and `PaymentTerms` at order-placement time, but the order itself is a separate consistency boundary.
- `RFQRequest`, `Quote`, `QuoteValidity` — owned by the **RFQ Lifecycle** aggregate; referenced by `RFQReference`. RFQs consume contract context for pricing; quote validity is a separate VO from `PriceValidity`.
- `Workspace`, `ProductList` — owned by **Workspaces / Saved Lists**; not directly referenced from Pricing in V1.

### Discussion

**`CustomerContract` as Single-Aggregate-Root, or two-aggregate split (Contract + ProductPrice separate)?**
- *Pro single aggregate (default)*: customer-specific pricing only exists inside a contract — there is no meaningful standalone `ProductPrice` outside contractual context; transactional consistency between contract status, price validity, and the price set is naturally enforced by `INV-3`, `INV-6`, and `INV-7`; one customer dashboard render needs one aggregate fetch (one round trip to the adapter).
- *Pro two-aggregate split (alternative)*: contracts and prices have somewhat independent change cadences (contract status changes rarely, prices may change frequently with seasonality); a large customer with thousands of `ProductPrice` rows produces a heavy aggregate when treated as single-root; two aggregates allow independent caching strategies (long-TTL contract metadata, short-TTL prices); separate adapter contracts for `CustomerContract.read` and `ProductPrice.read` may map more naturally onto NET7 endpoints.
- *Contra two-aggregate split*: the cross-aggregate seam between Contract and ProductPrice would require its own consistency reasoning (e.g., expiry of a contract must invalidate all its prices); two aggregates means two repositories and two query patterns inside Pricing alone.
- **Provisional decision**: single aggregate (`CustomerContract` as Root) for V1. The aggregate-as-fetch-unit advantage matches V1 dashboard read patterns. Revisit per `OQ-3` *(critical, Aggregate-Boundary)* after `erp-discovery.md` §3.c, §3.d clarifies how NET7 exposes contracts and prices and after Sprint-2 reveals the actual round-trip and payload-size profile. Decision substantially affects Sprint 2 query patterns and adapter contract.

**`PricingScale` (volume-based pricing) — VO collection or Sub-Entity?**
- *Pro VO collection (default)*: scale tiers are inert lookup tables `(quantityThreshold, UnitPrice)` with no identity of their own; replacing the scale on a `ProductPrice` is a wholesale replacement (simpler than tracking identity per tier); fewer schema rows, simpler adapter mapping.
- *Contra VO collection*: if AOT later wants to track tier-level expiry, tier-level audit ("price changed from X to Y at tier 2 on date Z"), or tier-level access controls (e.g., visible to certain user roles), tier identity becomes necessary — Sub-Entity is the right model.
- **Provisional decision**: VO collection. The value-collection model is simpler and sufficient for V1 *display*; the distinction matters only when tier-level semantics need identity. Revisit per `OQ-2` if NET7 exposes tier-level metadata that the web should preserve.

**Multi-currency support — per-Contract or per-Price granularity?**
- *Pro per-Contract (default)*: a customer organisation typically transacts in one currency (their corporate currency); one `Currency` on the contract simplifies cache strategy, simplifies dashboard rendering, simplifies any future order calculations on the ERP side.
- *Contra per-Contract*: international customers with multiple legal entities in different countries may need per-product currency variation; AOT may negotiate USD pricing for some products and EUR for others within the same contract.
- *Pro per-Price*: maximum flexibility; `UnitPrice` already carries `(Currency, amount)`, so the schema supports per-Price granularity natively without contract-level promotion.
- **Provisional decision**: per-Price granularity inside `UnitPrice`, with the typical case being all prices in a single currency. The schema does not enforce contract-level currency uniformity — that constraint, if AOT requires it, lives at the ERP business-rule layer, not in the aggregate. Revisit per `OQ-4` after `erp-discovery.md` §3.c clarifies multi-currency support in NET7.

## 4. Internal Structure

### Schema

| Name                | Type          | Owner       | Notes                                                                                    |
|---------------------|---------------|-------------|------------------------------------------------------------------------------------------|
| `CustomerContract`  | Entity (Root) | ERP-defined | Identity is `ContractId`; root of the aggregate; carries `ContractStatus`.               |
| `ContractId`        | Value Object  | ERP-defined | NET7-issued.                                                                             |
| `ContractStatus`    | Value Object  | ERP-defined | State machine (read-only-mirror): *Pending → Active → Suspended / Expired / Closed*.     |
| `ContractValidity`  | Value Object  | ERP-defined | From-to dates of contract validity.                                                      |
| `PaymentTerms`      | Value Object  | ERP-defined | NET 30 / NET 60 / prepayment / etc.                                                      |
| `LeadTime`          | Value Object  | ERP-defined | Typical delivery time under this contract (days or weeks).                               |
| `PricingTier`       | Value Object  | ERP-defined | Customer-category influencing pricing logic. *Hypothesis: ownership open (OQ-8).*        |
| `ProductPrice`      | Entity        | ERP-defined | Per-product pricing. *Hypothesis: Entity for versioning + scale-tiers (OQ-1).*           |
| `UnitPrice`         | Value Object  | ERP-defined | `(Currency, amount)` tuple.                                                              |
| `Currency`          | Value Object  | ERP-defined | ISO currency code.                                                                       |
| `MOQ`               | Value Object  | ERP-defined | Minimum order quantity per product under this contract.                                  |
| `PriceValidity`     | Value Object  | ERP-defined | From-to dates of a specific `ProductPrice`. *Hypothesis: state-machine read-only-mirror.*|
| `PricingScale`      | Value Object  | ERP-defined | Volume-based pricing. *Hypothesis: VO collection of (threshold, UnitPrice) tuples (OQ-2).*|

### Invariants

Transactional invariants — must hold at every commit of the aggregate (where "commit" means the read-side update path: ingest from NET7 into the web mirror):

- **INV-1** (structural). `CustomerContract` belongs to exactly one `Customer` via `CustomerReference`. A contract without a customer is malformed; cross-customer contract sharing is not permitted.
- **INV-2** (state-machine, read-only-mirror). `ContractStatus` follows the lifecycle *Pending → Active → Suspended / Expired / Closed*. Closed and Expired are terminal; Suspended is reversible (back to Active). The web mirror reflects NET7-driven transitions but never originates them; web-side mutation of `ContractStatus` is a violation. Status history is append-only.
- **INV-3** (structural). `ProductPrice` references both its parent `CustomerContract` and a `Product` via `ProductReference`. A price without one or the other is malformed.
- **INV-4** (cross-aggregate). `ProductPrice` is eventually consistent with the Product master in the **Catalog** aggregate. A change to a Product (retirement, identifier merge, certification change that triggers a contract-side price adjustment) propagates from Catalog to Pricing within an acceptable lag. *Hypothesis: lag of < 5 minutes acceptable for V1; validate via `erp-discovery.md` §3.f.* Aligns with `documents-compliance.md` `INV-6` cross-aggregate pattern.
- **INV-5** (behavioural). All fields are read-only on the web side per the `AGENTS.md` hard rule: *"NEVER put pricing, stock, or contract logic in the frontend"*. Local mutation is a violation; the only legal write path is *re-ingest* from NET7 via `Net7PricingAdapter`. The adapter contract has no write surface (see §5).
- **INV-6** (structural). An `Active` `CustomerContract` has at least one `ProductPrice`. An active contract without prices is malformed and indicates an ingest error or an upstream NET7 inconsistency (escalate, do not silently surface).
- **INV-7** (state-machine, read-only-mirror). `PriceValidity` follows the lifecycle *Active → Expired* (automatic transition driven by the `ValidityEnd` date) when modelled with lifecycle. The web mirror reflects expiry events from NET7; web never transitions `PriceValidity` locally. Expired prices remain accessible for audit (analog to `documents-compliance.md` `INV-7` — display vs audit-history distinction).

### Consistency Rules

- **Transactional (inside aggregate)**: ingest of a `CustomerContract` plus its `ProductPrice` set, contract status, contract validity, payment terms, lead time, pricing tier, and per-price validity commits atomically on the web side. INV-1, INV-2, INV-3, INV-5, INV-6, and INV-7 hold at commit time; status transitions in the read-only mirror (both `ContractStatus` and `PriceValidity`) are stamped from NET7 events and validated against INV-2 / INV-7 before being persisted to the local view.
- **Eventual (cross-aggregate)**: `INV-4` is explicitly eventual — `ProductPrice` lag relative to Catalog `Product` changes is bounded but non-zero. The `CustomerReference → Customer Account`, `DocumentReference → Documents`, `OrderReference → Order Management`, and `RFQReference → RFQ Lifecycle` linkages are eventually consistent — a contract can be marked *Closed* before all in-flight references are reconciled.

## 5. Web/ERP Ownership and Adapter Boundary

Per-component ownership statement and adapter mediation. Pricing & Contracts is the platform's strictest expression of read-only ERP ownership: every component is ERP-defined, the adapter has no write surface, and the `AGENTS.md` hard rule on pricing logic in the frontend is structurally enforced through `INV-5`.

- `CustomerContract`, `ContractId`, `ContractStatus`, `ContractValidity`, `PaymentTerms`, `LeadTime`, `PricingTier`, `ProductPrice`, `UnitPrice`, `Currency`, `MOQ`, `PriceValidity`, `PricingScale` — **ERP-defined** without exception. NET7 is source of truth for every component. Web reads through `Net7PricingAdapter` and caches per-customer with TTL discipline (OQ-5).

There are **no Shared and no Web-only components** in this aggregate. This is the only aggregate in the P1 series with a 13/0/0 ownership distribution; `documents-compliance.md` was the closest precedent at 11/1/0 (one Shared `DocumentScope`).

### Net7PricingAdapter

The single mediation layer for Pricing & Contracts ↔ NET7 traffic. Read-only adapter; idempotent by construction. Builds on the read-only adapter pattern from `catalog.md` (Task 1.4.1) and `documents-compliance.md` (Task 1.4.4).

- **Read**: contracts, `ProductPrice` set per contract, `MOQ`, `LeadTime`, `PaymentTerms`, contract and price validity dates, `PricingTier`. Read scope is per-customer — pricing reads are always authenticated and customer-scoped (no anonymous price discovery).
- **Cache strategy**: per-customer cache (not a global search index, unlike `Net7ProductAdapter` and `Net7DocumentAdapter`) — pricing is customer-specific by construction, so caching is keyed on `(CustomerId, ContractId)`. TTL discipline aligned to OQ-5 sync mechanism. Cache invalidation on a `ContractStatus` or `PriceValidity` event from NET7 (push) or on TTL expiry (pull).
- **Write-back**: **none**. Documents are authored exclusively in NET7; web is a read-only mirror. The adapter contract has no write surface — idempotency is by construction. This enforces the `AGENTS.md` hard rule on pricing logic in the frontend at the contract level: there is no syntactic way to write a price from the web side.
- **Status-sync**: read-only-mirror updates of `ContractStatus` and `PriceValidity` from NET7 (push from event/webhook vs scheduled pull, OQ-5). The web never transitions either status locally; ingest-time validation against INV-2 / INV-7 catches malformed transitions and escalates.
- **Failure mode**: pricing-display freshness is more business-critical than other read-only aggregates — stale pricing is not just a UX problem, it can cause financial / contractual inconsistency. Three-stage fallback chain:
  1. **Stale cache with banner**: serve cached prices with explicit `last-synced-at` timestamp and a UI banner indicating staleness within tolerance.
  2. **"Pricing temporarily unavailable"**: when sync lag exceeds a configured threshold, suppress price display on product pages and dashboards rather than show stale prices that may have changed.
  3. **Order / RFQ flow blocked**: when stale-tolerance is exceeded *and* the user attempts a downstream action (CAP-018 *Place Order*, CAP-015 *Submit RFQ*), the flow is blocked with an explicit message rather than allowed to proceed with stale prices. *Hypothesis: blocking threshold is configurable per-customer per their contractual sensitivity; OQ may emerge from Sprint-2 review.*

### Sprint-2 Hook

Input for the Sprint 2 ERP Boundary Design adapter contract. Open ERP capabilities to validate before contract sign-off:

- **Pricing read API** (CAP-006, CAP-007) — REST? OData? Per-customer scope? Bulk-read of all prices for a customer, or per-product on demand? (`erp-discovery.md` §3.b, §3.d.)
- **`ContractStatus` and `PriceValidity` event stream** — does NET7 push status / expiry transitions, or must web poll? (`erp-discovery.md` §3.f.) Drives OQ-5 and the cache-invalidation strategy.
- **`PricingTier` source** — ERP-enum (NET7 publishes a controlled list and web mirrors) or web-mapped (web maintains its own vocabulary, ERP emits a hint)? (OQ-8.) Affects the role-translation layer in the adapter.
- **Multi-currency support** — does NET7 model currency at contract level or per-price level? (OQ-4.) Affects the `UnitPrice` / `Currency` schema decision.
- **Future-dated prices** — does NET7 surface scheduled price changes (e.g., a new price effective on a future date)? (OQ-6.) Affects whether the aggregate needs to model "active now" vs "active later" prices distinctly.

## 6. Cross-Aggregate Relationships and Open Architecture Questions

### Cross-Aggregate ID-References

Pricing & Contracts references other aggregates **by identifier only**. No object navigation crosses these boundaries; consumers fetch the related data via the target aggregate's repository.

- **`CustomerReference`** → Customer Account aggregate. INV-1 enforces that every contract belongs to exactly one customer. Aligns with `customer-account.md` ContractStatus mention — the customer-side `ContractStatus` is the same status reflected here, sourced once from NET7 and surfaced in both aggregates.
- **`ProductReference`** → Catalog aggregate. Each `ProductPrice` carries a `ProductReference`; INV-3 enforces that every price has both a contract parent and a product reference. INV-4 acknowledges the cross-aggregate eventual-consistency relationship.
- **`DocumentReference`** → Documents & Compliance aggregate. Signed contract documents, contractual amendments, and KYC documents are referenced via `DocumentReference`. Aligns with `customer-account.md` `DocumentReference` for contract-related documents.
- **`OrderReference`** → Order Management aggregate. Contract-driven orders consume `ProductPrice`, `MOQ`, `LeadTime`, and `PaymentTerms` at order-placement time; the order itself is a separate consistency boundary that references Pricing back via `ContractId` for the snapshot used at order time.
- **`RFQReference`** → RFQ Lifecycle aggregate. RFQs consume contract context for pricing; quote validity is a separate VO from `PriceValidity` but may be aligned to it (e.g., a quote cannot extend past `PriceValidity`).

### Open Architecture Questions

These questions are the primary input to Sprint 2 ERP Boundary Design and may also be answered by `business-discovery.md` / `erp-discovery.md` interviews.

- **OQ-1**. `ProductPrice` — Entity (current default, versioning + scale-tiers) or Value Object? Triggers for re-evaluation: NET7 models prices as immutable VOs that are wholesale-replaced on any change; no scale-tier semantics.
- **OQ-2**. `PricingScale` — VO collection (current default) or Sub-Entity for tier-based pricing? Trigger: tier-level audit, expiry, or access controls become required.
- **OQ-3** *(critical, Aggregate-Boundary)*. Single aggregate (`CustomerContract` as Root, current default) vs two-aggregate split (Contract and ProductPrice as separate aggregates)? Decision substantially affects Sprint 2 query patterns and adapter contract. Validate via `erp-discovery.md` §3.c, §3.d on contract / price exposure shape and Sprint-2 round-trip / payload profiling.
- **OQ-4**. Multi-currency support — per-Contract granularity (one Currency per contract) or per-Price granularity (each `ProductPrice` may have its own Currency)? Validate via `erp-discovery.md` §3.c.
- **OQ-5**. `ContractStatus` and `PriceValidity` synchronisation — push from NET7 (event / webhook) vs scheduled pull on the web side? Drives the cache-invalidation strategy and the failure-mode threshold. Validate via `erp-discovery.md` §3.f.
- **OQ-6**. Future-dated prices (scheduled price changes) — in scope for V1 or post-V1? If in V1, the aggregate needs to model "active now" vs "active later" prices distinctly; if post-V1, the read mirror only surfaces currently-active prices. Validate via `business-discovery.md` §3.g (Decision Criteria) and `erp-discovery.md` §3.c.
- **OQ-7**. Price-history audit (older versions of expired `ProductPrice` retained for audit) — in scope V1 or post-V1? If V1, INV-7 expiry semantics need to preserve history; if post-V1, expired prices can be discarded from the web mirror. Validate via `business-discovery.md` §5.b (Quality Manager Audit-Trail-Bedürfnisse).
- **OQ-8**. `PricingTier` ownership — AOT-defined business logic (web maintains a controlled vocabulary, ERP emits hints) or NET7-defined enum (web mirrors NET7's internal classification)? Validate via `erp-discovery.md` §3.c, §3.e.

## 7. References

- [`catalog.md`](./catalog.md) — pattern source (read-only adapter, schema-table style, `INV-6` read-only enforcement).
- [`sample.md`](./sample.md) — second aggregate; state-machine bidirectional notation (not used here, only the read-only-mirror variant).
- [`customer-account.md`](./customer-account.md) — third aggregate; `ContractStatus` mention, hybrid-ownership pattern (not used here — Pricing is fully ERP-driven), security-boundary sub-class (not used here).
- [`documents-compliance.md`](./documents-compliance.md) — fourth aggregate; primary template for the read-only-mirror state-machine variant (`INV-2`, `INV-7`) and the cross-aggregate INV-tag (`INV-4`).
- [`bounded-contexts.md` §4.b, §5](../bounded-contexts.md) — Pricing & Contracts bounded context; `Catalog → Pricing` Customer/Supplier with ACL edge (#1); `Customer Account → Pricing` Customer/Supplier with ACL edge (#4).
- [`capability-map.md` §3.b](../capability-map.md) — Pricing capabilities CAP-006 *View customer-specific price for a product*, CAP-007 *View contract terms*; `Net7PricingAdapter` naming.
- [`glossary.md`](../glossary.md) — `MOQ`, `Pricing Tier`, `Lead Time`, `Quote` definitions; `RFQ` and `Order` cross-references.
- [`AGENTS.md`](../../../AGENTS.md) — engineering charter; ERP Ownership Rules; Hard Rules §3 (*"NEVER put pricing, stock, or contract logic in the frontend"* — structurally enforced by `INV-5` and the no-write-surface adapter contract).
- [`procurement-hypotheses.md`](../../procurement-hypotheses.md) — §6 customer-specific pricing display, §7 ERP ownership of pricing.
- [`erp-discovery.md`](../../erp-discovery.md) — §3.b Auth/Authz (per-customer scope), §3.c Datenmodell (contract / price exposure shape), §3.d Lese-Operationen, §3.e Schreib-Operationen (pricing writes — currently only NET7-side), §3.f Sync-Strategie (status events).
