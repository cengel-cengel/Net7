# Catalog Aggregate Sketch — AOT Procurement Platform

## 1. Purpose

This document is the DDD aggregate sketch for the **Catalog** bounded context (`bounded-contexts.md` §4.a). It fixes the aggregate root, the entities and value objects inside the consistency boundary, the invariants that must hold transactionally, and the Web/ERP ownership at component level. Direct input for Sprint 2 ERP Boundary Design (per-component contract definition) and Sprint 3 code scaffold (entity / VO classes).

All structural choices in this document are **working hypotheses** until validated by the Sprint-0 ERP-discovery stream (`erp-discovery.md`) and the customer-discovery stream (`business-discovery.md`).

This is the first of five aggregate sketches for the P1 contexts; the structure below is the pattern template for Sample, Customer Account, Documents & Compliance, and Pricing & Contracts (in subsequent Task 1.4.x deliveries).

## 2. Methodology

- **Aggregate** = a cluster of related domain objects treated as a single unit for data changes. One Aggregate Root acts as the only entry point; external code never reaches inside the aggregate by reference.
- **Entity vs Value Object**: an Entity has identity that persists across state changes (e.g., a `Product` is the same Product even after its description is edited). A Value Object is defined entirely by its attributes (e.g., `INCI` is the value `"Lavandula Angustifolia Oil"`; two equal INCI values are interchangeable).
- **Web / ERP / Shared ownership**: per `AGENTS.md` ERP Ownership Rules, NET7 owns master product data. *ERP-defined* means the source of truth is NET7; *Shared* means web augments or holds local state alongside the ERP record; *web-stored* means the value is replicated locally for search/index purposes but never authored on the web side.
- **Invariant typology**: *structural* (cardinality, presence) — must hold at every read; *behavioural* (rules on transitions) — enforced at write; *cross-aggregate* (depends on another aggregate) — eventually consistent only.
- **Hypothesis marking**: where ERP capability or domain semantics are unverified, the line carries `Hypothesis:` or `Open question:`. The §6 OQ-N register collects these for follow-up.
- **Cross-aggregate references**: aggregates reference each other **by ID only**. No object navigation crosses an aggregate boundary; reads of related data go through repositories or query services on the consumer side.

## 3. Aggregate Boundary

### Aggregate Root

**Product** is the Aggregate Root. Justification: every interesting Catalog operation — search hits, comparison, document discovery, sample request, RFQ submission — starts from a Product reference. Variants, specifications, INCI, CAS, and certifications are meaningful only relative to a Product, never standalone.

### Inside the Aggregate

Components inside the Catalog aggregate (transactional consistency boundary):

- `Product` — the root entity; identity stable across attribute changes.
- `ProductIdentifier` — value object wrapping the SKU / product code from NET7.
- `ProductVariant` — entity, parent-child to Product. *Hypothesis: variants live inside Product (lifecycle bound to parent). See OQ-1.*
- `Specification` — entity, versioned per Product. *Hypothesis: specifications live inside Product. See OQ-2.*
- `INCI`, `CASNumber`, `BotanicalName`, `Origin`, `Application`, `Certification`, `AllergenStatement`, `Category` — value objects describing the product.

The transactional invariants in §4 (INV-1 … INV-7) all hold inside this boundary.

### Outside the Aggregate

Explicitly **not** inside the Catalog aggregate:

- `Document` (SDS, COA, TDS, certificate files) — owned by the **Documents & Compliance** aggregate; referenced by ID. Document inventory and version history have their own consistency rules (per-batch COAs, expiry, download tracking) that do not belong to Product write paths.
- `Price`, `Contract`, `MOQ`, `PaymentTerms` — owned by the **Pricing & Contracts** aggregate; referenced by ID. Pricing logic is ERP-only per `AGENTS.md` hard rules; web never participates in price authoring.
- `AvailabilitySignal` ("Available" / "Low" / "Made-to-order") — *Open question: where does this live?* Catalog (alongside the product), Pricing (alongside contract terms), or a new Inventory aggregate. See OQ-4.
- `Workspace`, `SampleRequest`, `RFQRequest`, `Order` — owned by their respective aggregates; reference Product by `ProductIdentifier`.

### Discussion

**ProductVariant — inside Product or own aggregate?**
- *Pro inside (default)*: variants are meaningless without a parent product; cardinality invariant INV-1 is enforceable transactionally; lifecycle naturally bound (deactivate Product → deactivate Variants).
- *Contra inside*: variants may have independent lifecycles in NET7 (e.g., a packaging variant retired while the base product remains); variant-level pricing or stock could drive contention if variants live inside Product transactions.
- **Provisional decision**: inside Product. Revisit if `erp-discovery.md` §3.c shows variant-level lifecycle independence (OQ-1, OQ-6).

**Specification — inside Product or own aggregate?**
- *Pro inside*: specifications are tied to a single Product; versioning is bounded to that Product.
- *Contra inside*: specifications may be issued and accepted independently from Product master-data updates; multi-batch specifications could fan out.
- **Provisional decision**: inside Product. Versioning lives on the Specification entity. Revisit per OQ-2.

## 4. Internal Structure

### Schema

| Name                | Type          | Owner                       | Notes                                                                             |
|---------------------|---------------|-----------------------------|-----------------------------------------------------------------------------------|
| `Product`           | Entity (Root) | Shared                      | Identity is `ProductIdentifier`; root of the aggregate.                           |
| `ProductIdentifier` | Value Object  | ERP-defined                 | Stable SKU / product code from NET7. *Hypothesis: stable across renames; OQ.*    |
| `ProductVariant`    | Entity        | Shared                      | *Hypothesis: inside Product (OQ-1).* Identified within parent.                    |
| `Specification`     | Entity        | Shared                      | Versioned per Product. *Hypothesis: inside Product (OQ-2).*                       |
| `INCI`              | Value Object  | ERP-defined, web-stored     | International nomenclature for cosmetic ingredients; primary search facet.        |
| `CASNumber`         | Value Object  | ERP-defined, web-stored     | Chemical Abstracts Service identifier; primary search facet.                      |
| `BotanicalName`     | Value Object  | ERP-defined                 | Latin binomial for plant-derived materials.                                       |
| `Origin`            | Value Object  | ERP-defined                 | Country / region (sometimes cooperative) of origin.                               |
| `Application`       | Value Object  | Shared                      | Use case (emulsifier, anti-aging active, preservative, …); web-mappable to facets.|
| `Certification`     | Value Object  | ERP-defined                 | Bio, organic, kosher, halal. *Hypothesis: VO; revisit if versioning needed (OQ-3).* |
| `AllergenStatement` | Value Object  | ERP-defined                 | EU-14 allergen declaration alongside other regulatory statements.                 |
| `Category`          | Value Object  | Shared                      | Product-taxonomic classification. *Hypothesis: web-defined facet (OQ-8).*         |

### Invariants

Transactional invariants — must hold at every commit of the aggregate:

- **INV-1** (structural). `Product` has at least one `ProductVariant`. Active products without variants are not permitted.
- **INV-2** (structural). `ProductIdentifier` is unique across all active `Product` instances.
- **INV-3** (structural). A `ProductVariant` cannot exist without a parent `Product` (cardinality 1:1 child-to-parent).
- **INV-4** (behavioural). Each `Specification` version is uniquely owned by exactly one `Product`. Versions are immutable once published; new versions are new entity instances.
- **INV-5** (structural). Active `Product` has at least one `Application` assigned. A product without a use case cannot be surfaced in catalog search.
- **INV-6** (behavioural). ERP-sourced fields (`INCI`, `CASNumber`, `BotanicalName`, `Origin`, `Certification`, `AllergenStatement`, `ProductIdentifier`) are read-only on the web side. Local mutation is a violation; the only legal path is re-ingest from NET7 via the adapter.
- **INV-7** (structural). `Category` assignment exists for any active `Product`. Uncategorised products are draft-only.

### Consistency Rules

- **Transactional (inside aggregate)**: changes to Product attributes, variant cardinality, specification versions, and Application list must commit atomically. INV-1 through INV-7 hold at commit time.
- **Eventual (cross-aggregate)**: search-index updates downstream of a Product change are eventually consistent — the web search index may lag a Product write by seconds. *Hypothesis: lag of < 60 s acceptable for V1; validate in CAP-002 discovery.* Document availability (Documents aggregate), price (Pricing aggregate), and availability signal (placement TBD per OQ-4) all converge eventually.

## 5. Web/ERP Ownership and Adapter Boundary

Per-component ownership statement and adapter mediation.

- `Product` — **Shared**. Master attributes (name, description, identifier, INCI/CAS, etc.) sourced from NET7. Web augments with SEO metadata, search-index entries, and presentation slots not represented in NET7. Mediated by `Net7ProductAdapter`.
- `ProductIdentifier` — **ERP-defined**. NET7 issues and owns; web treats as opaque stable handle. Adapter never mints new IDs.
- `ProductVariant` — **Shared**. Variant master comes from NET7; web may attach presentation order or display flags. `Net7ProductAdapter`.
- `Specification` — **Shared**. Versioned spec content from NET7; web stores acceptance state per customer if needed (likely Customer Account aggregate, not here). `Net7ProductAdapter`.
- `INCI`, `CASNumber` — **ERP-defined, web-stored**. Replicated into the web search index for fast facet lookup; never authored on the web. `Net7ProductAdapter` ingest path.
- `BotanicalName`, `Origin`, `Certification`, `AllergenStatement` — **ERP-defined**. Read-through with short-TTL cache. `Net7ProductAdapter`.
- `Application` — **Shared**. ERP may emit raw application strings; web maps onto a controlled vocabulary for faceting. *Hypothesis: web-side mapping table is acceptable; revisit if NET7 already publishes a controlled list.*
- `Category` — **Shared**. *Hypothesis: web defines the customer-facing taxonomy independently of NET7's internal categorisation (OQ-8).*

### Net7ProductAdapter

The single mediation layer for Catalog ↔ NET7 traffic. Responsibilities:

- **Read**: master product data, variants, specifications, INCI/CAS, origin, certifications, allergen statements.
- **Search-index sync**: ingest into web-side search index. *Open question: push from NET7 (webhook / event) vs scheduled pull (cron). See OQ-5.*
- **Write-back**: *currently none expected.* Catalog is read-only on the web side; new products are authored in NET7. *Hypothesis: holds for V1; if AOT later wants web-authored draft products, this section needs revision.*
- **Idempotency**: read-only adapter, idempotent by construction. The search-index sync must be idempotent on re-ingest (same Product version twice → no-op).
- **Failure mode**: stale cache with explicit "last-synced-at" timestamp; degraded UX (banner) when sync fails for > N minutes.

### Sprint-2 Hook

The above is the input for the Sprint 2 ERP Boundary Design adapter contract. Open ERP capabilities to validate before contract sign-off:

- Read-mechanism for the 12 components (REST? OData? File export? — `erp-discovery.md` §3.a).
- Sync mechanism (push vs pull, see OQ-5).
- Field-level read scope (can web read the full product entity, or only entitlement-filtered fields? — `erp-discovery.md` §3.b, §3.d).
- ID stability (does `ProductIdentifier` survive product renames/merges? — `erp-discovery.md` §3.c).

## 6. Cross-Aggregate Relationships and Open Architecture Questions

### Cross-Aggregate ID-References

Catalog references other aggregates **by identifier only**. No object navigation crosses these boundaries; consumers fetch the related data via the target aggregate's repository.

- **`DocumentReference`** → Documents & Compliance aggregate. Catalog stores `(DocumentId, DocumentType)` tuples per Product; the Documents aggregate owns content, version, batch linkage.
- **`PriceReference`** → Pricing & Contracts aggregate. Catalog stores nothing about price; UX renders price by querying Pricing with `(ProductIdentifier, CustomerId)`.
- **`AvailabilityIndicator`** — placement is an **Open question (OQ-4)**. Three candidates:
  - *In Catalog*: per-product banded signal (Available / Low / Made-to-order) cached from NET7.
  - *In Pricing*: alongside contract-driven lead time and MOQ.
  - *In a new Inventory aggregate*: separate consistency boundary for stock-related state.

### Open Architecture Questions

These questions are the primary input to Sprint 2 ERP Boundary Design and may also be answered by `business-discovery.md` / `erp-discovery.md` interviews.

- **OQ-1**. `ProductVariant` — inside Product (current default) or own aggregate? Trigger for re-evaluation: NET7 surfaces independent variant lifecycle (deactivate variant, keep base product). Validate via `erp-discovery.md` §3.c.
- **OQ-2**. `Specification` — inside Product (current default) or own aggregate? Trigger: per-batch specifications or cross-product reuse. Validate via `erp-discovery.md` §3.c.
- **OQ-3**. `Certification` — Value Object (current) or Entity? Becomes Entity if certifications carry independent identity (issuer-issued certificate number, expiry, re-issue events). Likely Entity once Documents & Compliance aggregate is sketched.
- **OQ-4**. `AvailabilityIndicator` placement — Catalog vs Pricing vs new Inventory aggregate? Decision impacts read paths for CAP-005 *Discover Alternatives* and CAP-001 *Browse and Filter*.
- **OQ-5**. Search-index synchronisation — push from NET7 (webhook / event) vs scheduled pull on the web side. Drives the ingest path of `Net7ProductAdapter`. Validate via `erp-discovery.md` §3.f.
- **OQ-6**. `ProductVariant` lifecycle — independent of Product (deactivate variant only) or strictly bound? Tied to OQ-1.
- **OQ-7**. Multilingual product names and descriptions — web-side translation layer (per locale) vs ERP-sourced multilingual fields? Affects `Net7ProductAdapter` schema.
- **OQ-8**. `Category` taxonomy — web-defined customer-facing facets vs ERP-sourced taxonomy? `procurement-hypotheses.md` §6 *Smart Search* assumes web controls faceting; needs explicit confirmation.

## 7. References

- [`bounded-contexts.md` §4.a](../bounded-contexts.md) — Catalog bounded context, ownership, key entities, open questions.
- [`capability-map.md` §3.a](../capability-map.md) — Catalog capabilities CAP-001 … CAP-005, `Net7ProductAdapter` naming.
- [`glossary.md`](../glossary.md) — `INCI`, `CAS`, `Botanical Name`, `Origin`, `Application`, `Variant`, `Specification`, `Certificate`, `Allergen Statement`, `Category` definitions.
- [`AGENTS.md`](../../../AGENTS.md) — engineering charter; ERP Ownership Rules, ERP Integration Rules.
- [`procurement-hypotheses.md`](../../procurement-hypotheses.md) — §6 *Smart Search*, §6 *Availability Confidence*, §5 Stage 1.
- [`erp-discovery.md`](../../erp-discovery.md) — §3.c Datenmodell, §3.d Lese-Operationen, §3.f Sync-Strategie.
