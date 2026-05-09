# Documents & Compliance Aggregate Sketch — AOT Procurement Platform

## 1. Purpose

This document is the DDD aggregate sketch for the **Documents & Compliance** bounded context (`bounded-contexts.md` §4.c). It fixes the aggregate root, the entities and value objects inside the consistency boundary, the document state machine (in *read-only-mirror* form — NET7 owns the transitions), and the Web/ERP ownership at component level. Direct input for Sprint 2 ERP Boundary Design (document read API, batch-document linkage, file-reference semantics) and Sprint 3 code scaffold (entities, value objects, completeness rules).

All structural choices in this document are **working hypotheses** until validated by the Sprint-0 ERP-discovery stream (`erp-discovery.md`) and the customer-discovery stream (`business-discovery.md`).

This is the fourth of five aggregate sketches for the P1 contexts. The 7-section structure follows the pattern template established in [`catalog.md`](./catalog.md) (Task 1.4.1); the *state-machine* invariant sub-class comes from [`sample.md`](./sample.md) (Task 1.4.2) and is applied here in its **read-only-mirror** variant — the web mirrors NET7-driven state transitions but never transitions state locally. The *security-boundary* sub-class introduced in [`customer-account.md`](./customer-account.md) (Task 1.4.3) is **not** relevant for this aggregate; *hybrid-ownership* and *write-path adapter* shapes are also **not** relevant — Documents & Compliance is fully ERP-driven and the adapter is read-only.

## 2. Methodology

- **Aggregate** = a cluster of related domain objects treated as a single unit for data changes. One Aggregate Root acts as the only entry point; external code never reaches inside the aggregate by reference.
- **Entity vs Value Object**: an Entity has identity that persists across state changes (e.g., a `DocumentVersion` is a specific issuance — version 3 of an SDS is the same artefact regardless of how it is read). A Value Object is defined entirely by its attributes (e.g., `DocumentLanguage` is a string code; two documents with the same language code are interchangeable on that attribute).
- **Web / ERP / Shared ownership**: per `AGENTS.md` ERP Ownership Rules, NET7 owns master document data, version history, batch-document linkage, and file storage. *ERP-defined* means the source of truth is NET7; *Shared* means web augments or holds local state alongside the ERP record. *Web-only* and *Web-only security-sensitive* sub-classes (introduced in earlier aggregates) are not relevant here — Documents & Compliance has no Web-only components.
- **Invariant typology** — five sub-classes (carried over from earlier aggregates), with one notation refinement: *structural* (cardinality, presence) — must hold at every read; *behavioural* (transition rules) — enforced at write; *state-machine* (legal status transitions) — enforced at every status change, status-history immutable; *cross-aggregate* (depends on another aggregate) — eventually consistent only; *security-boundary* — hard rules that some data never crosses a boundary at all (not used here). The *state-machine* sub-class supports two notation variants: **bidirectional** (web can transition state locally, as in `sample.md` `SampleStatus` and `customer-account.md` `UserStatus`) and **read-only-mirror** (NEW — web mirrors a NET7-driven state machine without local transitions, as in `INV-3` below). This aggregate is also the first to apply the **cross-aggregate** sub-class as an invariant tag (`INV-6`).
- **Hypothesis marking**: where ERP capability or domain semantics are unverified, the line carries `Hypothesis:` or `Open question:`. The §6 OQ-N register collects these for follow-up.
- **Cross-aggregate references**: aggregates reference each other **by ID only**. No object navigation crosses an aggregate boundary; reads of related data go through repositories or query services on the consumer side.

## 3. Aggregate Boundary

### Aggregate Root

**Document** is the Aggregate Root (provisional). Justification: every compliance-relevant operation — completeness check on a product, single-doc download, bulk-doc download per workspace, batch-traceability lookup — is anchored to a specific document with a specific version. `DocumentVersion`, `DocumentStatus`, `IssueDate`, `ExpiryDate`, `DocumentFileReference`, language and scope are all properties of one specific Document. There is no compliance operation that does not start from a document reference.

### Inside the Aggregate

Components inside the Documents & Compliance aggregate (transactional consistency boundary). All components except `DocumentScope` are ERP-defined; the aggregate is fully ERP-driven and read-only on the web side.

- `Document` — the root entity; identity stable across version transitions and status changes.
- `DocumentId` — value object wrapping the stable identifier from NET7.
- `DocumentType` — value object: *SDS / COA / TDS / Allergen Statement / Bio-Cert / Kosher / Halal / REACH-Reg / FDA-Reg* (extensible per `DocumentType` master from NET7).
- `DocumentVersion` — entity, child of Document. *Hypothesis: entity for versioning semantics (OQ-1); each version has its own identity, IssueDate, file reference.*
- `DocumentStatus` — value object stamped onto the document (state-machine, read-only-mirror: *Active → Expired* by `ExpiryDate` automation, or *Active → Superseded* when a newer version is published).
- `IssueDate` — value object; the date the version was issued by the issuing authority.
- `ExpiryDate` — value object, optional. SDS does not expire; certifications, bio-certificates, and audited statements do.
- `DocumentLanguage` — value object: *EN / DE* (extensible). *Hypothesis: web fallback to EN if locale-specific document is absent (OQ-4).*
- `DocumentFileReference` — value object; URI / handle to the binary in NET7-DMS. *Hypothesis: stable URI vs signed/short-lived link is open (OQ-5).*
- `DocumentScope` — value object: *per-product / per-batch / per-shipment*. The only Shared component (web augments scope-mapping rules for completeness computation).
- `BatchReference` — value object, present when `DocumentScope = per-batch` (especially COAs). *Hypothesis: targets Catalog `ProductVariant` or a future Inventory aggregate (OQ-3).*
- `RequiredDocumentSet` — value object; per-product list of required `DocumentType`s. *Hypothesis: lives in this aggregate as the Catalog-side view of "what compliance requires"; alternative placement in Catalog is OQ-2.*

The transactional invariants in §4 (INV-1 … INV-7) all hold inside this boundary, including the read-only-mirror state-machine invariant on `DocumentStatus` and the cross-aggregate invariant on `RequiredDocumentSet`.

### Outside the Aggregate

Explicitly **not** inside the Documents & Compliance aggregate:

- `Product`, `ProductVariant`, `Specification` — owned by the **Catalog** aggregate; referenced by `ProductReference` (and by `BatchReference` if Batch lives there per OQ-3).
- `Customer`, `User` — owned by the **Customer Account** aggregate; not referenced directly from documents (entitlement checks happen at Capability layer, not at document level).
- `SampleRequest`, `SampleStatus` — owned by the **Sample Lifecycle** aggregate; referenced by `SampleReference` for sample-COAs and dispatch notices.
- `Order`, `OrderLine`, `BatchAssignment` — owned by the **Order Management** aggregate; referenced by `OrderReference` for dispatch-notice and per-order document bundles.
- `Workspace`, `ProductList` — owned by **Workspaces / Saved Lists**; referenced by `WorkspaceReference` for the CC-3 *Document Bulk Download per Workspace* cross-context capability (P3 / Future).
- `Price`, `Contract`, `MOQ` — owned by **Pricing & Contracts**; not referenced from documents in V1.

### Discussion

**`DocumentVersion` — Entity or Value Object?**
- *Pro Entity (default)*: each version has independent identity (v1, v2, v3 are not interchangeable even if the rest of the metadata matches); supersession events are entity-level transitions, not VO replacements; the web caches and references specific versions over time, which requires identity; download logs target a specific version, which needs identity.
- *Contra Entity*: if NET7 models versions as immutable stamps on the Document (no separate identity, just an `IssueDate` value), promoting them to web-side Entity adds bookkeeping that the source system does not require; the web could treat each version as a derived VO from `(DocumentId, IssueDate)`.
- **Provisional decision**: Entity. Versioning is identity-bearing in compliance domains (regulatory cite-ability, "we relied on version X at time Y"). Revisit per `OQ-1` after `erp-discovery.md` §3.c clarifies how NET7 represents versions.

**`RequiredDocumentSet` — inside Documents aggregate, or as Product metadata in Catalog?**
- *Pro inside Documents (default)*: keeps Catalog free of compliance-specific metadata; the Documents aggregate is the natural home of "what a compliant product needs"; web-side completeness logic (CAP-008) is a Documents-domain concern that consumes Catalog product references.
- *Contra inside Documents*: arguments for Catalog placement — required-document set is intrinsic to the product itself (organic certified product *needs* a Bio-Cert by definition); placing it in Documents creates a cross-aggregate read on every product detail render (CAP-003); Catalog already aggregates product metadata.
- **Provisional decision**: inside Documents aggregate. The Catalog aggregate already carries `Certification` as a VO; `RequiredDocumentSet` is the inverse mapping (which document *types* are required given the product's certifications). `INV-6` (cross-aggregate) acknowledges the eventual-consistency relationship to Product master. Revisit per `OQ-2` if catalogue-side rendering requires synchronous completeness.

**`BatchReference` placement — Catalog `ProductVariant` or own Inventory aggregate?**
- *Pro Catalog ProductVariant*: variants and batches are tightly related (a batch is a production run of a specific variant); already in scope for the Catalog aggregate.
- *Contra Catalog ProductVariant*: batches have their own lifecycle (production, quality control, dispatch, recall) that does not naturally belong in a product catalogue; recall scenarios cut across orders and require an Inventory-style aggregate; per-batch availability and quantities are not Catalog concerns.
- **Pro own Inventory aggregate**: clear separation of "what we sell" (Catalog) from "what we have on hand" (Inventory); aligns with the catalog `OQ-4` AvailabilityIndicator placement question.
- **Provisional decision**: defer. The placement ties into multiple open questions: catalog `OQ-1` (Variant lifecycle independence), catalog `OQ-4` (Availability placement), and this aggregate's `OQ-3`. For Documents purposes, `BatchReference` is a typed handle that the consumer resolves against whichever aggregate ends up owning Batches. Revisit per `OQ-3` after Sprint-2 ERP boundary review.

## 4. Internal Structure

### Schema

| Name                    | Type          | Owner       | Notes                                                                                       |
|-------------------------|---------------|-------------|---------------------------------------------------------------------------------------------|
| `Document`              | Entity (Root) | ERP-defined | Identity is `DocumentId`; root of the aggregate; carries `DocumentStatus`.                  |
| `DocumentId`            | Value Object  | ERP-defined | NET7-issued. *Hypothesis: stable across version transitions.*                               |
| `DocumentType`          | Value Object  | ERP-defined | SDS / COA / TDS / Allergen Statement / Bio-Cert / Kosher / Halal / REACH-Reg / FDA-Reg.     |
| `DocumentVersion`       | Entity        | ERP-defined | *Hypothesis: Entity for versioning semantics (OQ-1).* Versions are append-only.             |
| `DocumentStatus`        | Value Object  | ERP-defined | State machine (read-only-mirror): *Active → Expired* (auto by ExpiryDate) or *Active → Superseded*. |
| `IssueDate`             | Value Object  | ERP-defined | Date the version was issued by the issuing authority.                                       |
| `ExpiryDate`            | Value Object  | ERP-defined | Optional. SDS does not expire; certificates do.                                             |
| `DocumentLanguage`      | Value Object  | ERP-defined | EN / DE (extensible). *Hypothesis: web fallback EN if DE absent (OQ-4).*                    |
| `DocumentFileReference` | Value Object  | ERP-defined | URI / handle to binary in NET7-DMS. *Hypothesis: stable vs signed/short-lived (OQ-5).*      |
| `DocumentScope`         | Value Object  | Shared      | *per-product / per-batch / per-shipment*. Web augments scope-mapping for completeness.      |
| `BatchReference`        | Value Object  | ERP-defined | Present when scope = per-batch (esp. COAs). *Hypothesis: target placement (OQ-3).*          |
| `RequiredDocumentSet`   | Value Object  | ERP-defined | Per-product list of required `DocumentType`s. *Hypothesis: inside Documents (OQ-2).*        |

### Invariants

Transactional invariants — must hold at every commit of the aggregate (where "commit" here means the read-side update path: ingest from NET7 into the web mirror):

- **INV-1** (structural). `Document` references at least one `Product` via `ProductReference`. A document not bound to any product cannot be discovered through Catalog UX.
- **INV-2** (structural). Per-batch documents (typically COAs) must reference a `Batch` via `BatchReference`. A per-batch-scoped document without a batch handle is malformed and rejected at ingest.
- **INV-3** (state-machine, read-only-mirror). `DocumentStatus` follows the lifecycle *Active → Expired* (automatic transition driven by `ExpiryDate`) or *Active → Superseded* (when a newer `DocumentVersion` is published in NET7). The web mirror reflects these transitions but never originates them; web-side mutation of `DocumentStatus` is a violation. Status history is append-only.
- **INV-4** (structural). Each `DocumentVersion` has a unique `IssueDate` within its parent `Document`. Two versions cannot share an issue date; collisions on ingest indicate either an ERP data error or a versioning misunderstanding (escalate, do not silently dedup).
- **INV-5** (behavioural). All fields are read-only on the web side; the entire aggregate is a read-only mirror. Local mutation is a violation; the only legal write path is *re-ingest* from NET7 via `Net7DocumentAdapter` (analog to `catalog.md` `INV-6`).
- **INV-6** (cross-aggregate). `RequiredDocumentSet` is eventually consistent with the Product master in the **Catalog** aggregate. A change to Product certification or category in Catalog should propagate to the required-document set within an acceptable lag. *Hypothesis: lag of < 5 minutes acceptable for V1; validate via `erp-discovery.md` §3.f.*
- **INV-7** (structural). `Document` instances with `DocumentStatus = Expired` or `DocumentStatus = Superseded` are not surfaceable as "current" in completeness checks (CAP-008). They remain accessible via direct fetch (audit, history) but never count toward the required-document tally.

### Consistency Rules

- **Transactional (inside aggregate)**: ingest of a Document plus its versions, status, scope, and batch reference commits atomically on the web side. INV-1 through INV-5 and INV-7 hold at commit time; status transitions in the read-only mirror are stamped from NET7 events and validated against INV-3 before being persisted to the local view.
- **Eventual (cross-aggregate)**: `INV-6` is explicitly eventual — `RequiredDocumentSet` lag relative to Catalog `Product` changes is bounded but non-zero. Document content (binary files) referenced via `DocumentFileReference` is fetched on demand from NET7-DMS; there is no guarantee that the cached metadata and the binary content are version-aligned to the second, but they will converge within the OQ-6 cache window.

## 5. Web/ERP Ownership and Adapter Boundary

Per-component ownership statement and adapter mediation. The Documents & Compliance aggregate is the platform's reference **read-only-mirror** aggregate: every component except `DocumentScope` is ERP-defined, and the adapter has no write path.

- `Document`, `DocumentId`, `DocumentType`, `DocumentVersion`, `DocumentStatus`, `IssueDate`, `ExpiryDate`, `DocumentLanguage`, `DocumentFileReference`, `BatchReference`, `RequiredDocumentSet` — **ERP-defined**. NET7 is source of truth; web reads through `Net7DocumentAdapter` and caches with TTL discipline (OQ-6).
- `DocumentScope` — **Shared**. ERP emits raw scope hints (per-product, per-batch, per-shipment); web augments with scope-mapping rules used by completeness logic.

### Net7DocumentAdapter

The single mediation layer for Documents & Compliance ↔ NET7 traffic. Read-only adapter; idempotent by construction. Builds on the read-only adapter pattern from `catalog.md` (Task 1.4.1).

- **Read**: document master, version history, metadata (type, status, language, issue/expiry dates), `DocumentFileReference` URIs, `RequiredDocumentSet` per Product.
- **Search-index sync**: ingest into the web-side document index for fast per-Product / per-Batch / per-Order document lookup. Drives CAP-008 *completeness indicator*, CAP-009 *single-document download*, and CAP-010 *bulk-download*. *Hypothesis: incremental ingest on status events; full re-ingest on schedule. Validate via `erp-discovery.md` §3.f.*
- **Write-back**: **none**. Documents are authored exclusively in NET7-DMS; the web is a read-only mirror. The adapter contract has no write surface — idempotency is by construction.
- **Status-sync**: read-only-mirror updates of `DocumentStatus` from NET7 (push from event/webhook vs scheduled pull, aligned to OQ-6 caching strategy). The web never transitions `DocumentStatus` locally; ingest-time validation against INV-3 catches malformed transitions and escalates rather than silently accepting.
- **Failure mode**: stale cache with explicit `last-synced-at` timestamp; expired-document fallback message ("documents temporarily unavailable, last sync at X") when sync lag exceeds a configured threshold; CAP-009 download falls back to a direct NET7-DMS link if the cached `DocumentFileReference` returns 404 (stale handle) — drives OQ-5 stable-vs-signed decision.

### Sprint-2 Hook

Input for the Sprint 2 ERP Boundary Design adapter contract. Open ERP capabilities to validate before contract sign-off:

- **Document master read API** (CAP-008, CAP-009) — REST? OData? File-based export? (`erp-discovery.md` §3.d.) Drives the read mechanism in `Net7DocumentAdapter`.
- **Per-batch document linkage** — how does NET7 expose `BatchReference` for per-batch-scoped documents (especially COAs)? (`erp-discovery.md` §3.c, §3.d.) Affects INV-2 enforcement.
- **`DocumentFileReference` semantics** — stable URI vs signed / short-lived link? (`erp-discovery.md` §3.b, §3.h.) Drives `OQ-5` and the failure-mode fallback.
- **`DocumentStatus` event stream** — does NET7 push *Expired* / *Superseded* transitions, or must web poll? (`erp-discovery.md` §3.f.) Drives the read-only-mirror sync mechanism.
- **Bulk-download capability** — does NET7 expose a multi-document API, or must the web orchestrate per-document pulls? (`erp-discovery.md` §3.h on rate-limits is directly relevant.) Drives CC-3 *Document Bulk Download per Workspace* feasibility (`OQ-7`).

## 6. Cross-Aggregate Relationships and Open Architecture Questions

### Cross-Aggregate ID-References

Documents & Compliance references other aggregates **by identifier only**. No object navigation crosses these boundaries; consumers fetch the related data via the target aggregate's repository.

- **`ProductReference`** → Catalog aggregate. Every Document is scoped to one or more Products; carries the Catalog `ProductIdentifier` and resolves there for product-side metadata (name, INCI, certifications).
- **`BatchReference`** → Catalog `ProductVariant` (current default) or an own Inventory aggregate (per `OQ-3`). Present only when `DocumentScope = per-batch`. Per-batch COAs are the canonical example; dispatch-shipment-notice batches also surface here.
- **`OrderReference`** → Order Management aggregate. Dispatch notices and per-order document bundles are referenced via Order. Documents that arise from order fulfilment (rather than product master) link back through `OrderReference`.
- **`SampleReference`** → Sample Lifecycle aggregate. Sample-COAs and sample-dispatch documents reference the originating `SampleRequest` via `SampleReference`; aligns with `sample.md` `DocumentReference` cross-aggregate ID.
- **`WorkspaceReference`** → Workspaces / Saved Lists aggregate. Used by the CC-3 *Document Bulk Download per Workspace* capability (`capability-map.md` §4 CC-3, V1 priority P3 / Future). Documents themselves are not workspace-bound; the cross-context flow aggregates documents *across* products in a workspace.

### Open Architecture Questions

These questions are the primary input to Sprint 2 ERP Boundary Design and may also be answered by `business-discovery.md` / `erp-discovery.md` interviews.

- **OQ-1**. `DocumentVersion` — Entity (current default, versioning semantics) or Value Object? Trigger: NET7 models versions as a stamp on the parent document with no separate identity. Validate via `erp-discovery.md` §3.c.
- **OQ-2** *(critical, cross-aggregate impact)*. `RequiredDocumentSet` placement — Documents aggregate (current default) or Catalog (product metadata)? Decision substantially affects Catalog aggregate retroactively. Validate via `business-discovery.md` §3.f and `erp-discovery.md` §3.d.
- **OQ-3**. `BatchReference` placement — Catalog `ProductVariant` or own Inventory aggregate? Coupled to catalog `OQ-1` (Variant lifecycle independence) and catalog `OQ-4` (Availability placement). Validate via `erp-discovery.md` §3.c.
- **OQ-4**. `DocumentLanguage` fallback — EN if DE absent (current default) or strict per-locale with no fallback? Trigger: legal/regulatory teams may require strict locale matching for certain document types (e.g., Allergen Statement in DE for German market). Validate via `business-discovery.md` §5.a (Quality Manager).
- **OQ-5**. `DocumentFileReference` semantics — stable URI vs signed / short-lived link? Drives caching strategy (OQ-6) and failure-mode fallback. Validate via `erp-discovery.md` §3.b, §3.h.
- **OQ-6**. Caching strategy — short-TTL (re-fetch on every read) vs hard-TTL with expiry-aware invalidation (cache until ExpiryDate or a status event invalidates)? Affects freshness of completeness checks (CAP-008) and download links (CAP-009).
- **OQ-7**. Bulk-download (CC-3) implementation — streaming-ZIP (per-doc fetch in flight) vs pre-bundled (server pre-builds an archive) vs scheduled async (queue + email link)? Decision depends on the bulk-download capability in NET7 and rate-limits (`erp-discovery.md` §3.h). V1 priority is P3 / Future.
- **OQ-8**. Document search index — per-Document (each document is a search target) or per-Product (with the document set as a facet)? Affects CAP-008 *completeness indicator* render path and CAP-009 single-doc lookup.

## 7. References

- [`catalog.md`](./catalog.md) — pattern source (read-only adapter, schema-table style, `INV-6` read-only enforcement).
- [`sample.md`](./sample.md) — state-machine sub-class source; here applied in *read-only-mirror* notation variant.
- [`customer-account.md`](./customer-account.md) — third aggregate; security-boundary and hybrid-ownership patterns introduced there are not relevant here.
- [`bounded-contexts.md` §4.c, §5](../bounded-contexts.md) — Documents & Compliance bounded context; `Catalog → Documents` Customer/Supplier edge (#2); `Order → Documents` edge (#8).
- [`capability-map.md` §3.c, §4 CC-3](../capability-map.md) — Document capabilities CAP-008 *completeness indicator*, CAP-009 *single-document download*, CAP-010 *bulk-download*; Cross-Context CC-3 *Document Bulk Download per Workspace* (P3 / Future); `Net7DocumentAdapter` naming.
- [`glossary.md`](../glossary.md) — `SDS`, `COA`, `TDS`, `Allergen Statement`, `Certificate`, `Bio-Zertifikat`, `Batch`, `Lot`, `EU-Bio-VO 2018/848`, `EU-Kosmetik-VO 1223/2009`, `LFGB`, `REACH`, `FDA`, `Kosher / Halal` definitions.
- [`AGENTS.md`](../../../AGENTS.md) — engineering charter; ERP Ownership Rules; ERP Integration Rules.
- [`procurement-hypotheses.md`](../../procurement-hypotheses.md) — §6 *Document Completeness*, §3 buyer decision criteria including documentation completeness, §5 Stage 3 *Documentation*.
- [`erp-discovery.md`](../../erp-discovery.md) — §3.b Auth (file-reference auth scope), §3.c Datenmodell (document entities, batch linkage), §3.d Lese-Operationen (document read), §3.f Sync-Strategie (status events), §3.h Performance + Limits (bulk-download rate-limits).
