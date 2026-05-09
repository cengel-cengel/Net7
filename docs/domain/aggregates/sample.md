# Sample Lifecycle Aggregate Sketch — AOT Procurement Platform

## 1. Purpose

This document is the DDD aggregate sketch for the **Sample Lifecycle** bounded context (`bounded-contexts.md` §4.d). It fixes the aggregate root, the entities and value objects inside the consistency boundary, the lifecycle state machine, the invariants that must hold transactionally, and the Web/ERP ownership at component level. Direct input for Sprint 2 ERP Boundary Design (sample workflow contract definition) and Sprint 3 code scaffold (entity / VO classes, state machine).

All structural choices in this document are **working hypotheses** until validated by the Sprint-0 ERP-discovery stream (`erp-discovery.md`) and the customer-discovery stream (`business-discovery.md`).

This is the second of five aggregate sketches for the P1 contexts. The 7-section structure follows the pattern template established in [`catalog.md`](./catalog.md) (Task 1.4.1). Two pattern extensions introduced here become reusable in subsequent sketches: the *state-machine* sub-class of invariants (§2, §4) and the *write-path adapter* shape (§5) — both apply to Customer Account, RFQ Lifecycle, and Order Management. Documents & Compliance and Pricing & Contracts remain read-only on the web side.

## 2. Methodology

- **Aggregate** = a cluster of related domain objects treated as a single unit for data changes. One Aggregate Root acts as the only entry point; external code never reaches inside the aggregate by reference.
- **Entity vs Value Object**: an Entity has identity that persists across state changes (e.g., a `SampleRequest` is the same request even after its status advances from *Requested* to *Shipped*). A Value Object is defined entirely by its attributes (e.g., `SampleStatus` is a stamped value within the lifecycle; two requests in the same status are interchangeable on that attribute).
- **Web / ERP / Shared ownership**: per `AGENTS.md` ERP Ownership Rules, NET7 owns master sample / order / fulfilment data. *ERP-defined* means the source of truth is NET7; *Shared* means web augments or holds local state alongside the ERP record; *Web-only* means the value lives entirely in the web database with no ERP counterpart.
- **Invariant typology**: *structural* (cardinality, presence) — must hold at every read; *behavioural* (transition rules) — enforced at write; *state-machine* (legal status transitions) — enforced at every status change, status-history immutable; *cross-aggregate* (depends on another aggregate) — eventually consistent only.
- **Hypothesis marking**: where ERP capability or domain semantics are unverified, the line carries `Hypothesis:` or `Open question:`. The §6 OQ-N register collects these for follow-up.
- **Cross-aggregate references**: aggregates reference each other **by ID only**. No object navigation crosses an aggregate boundary; reads of related data go through repositories or query services on the consumer side.

## 3. Aggregate Boundary

### Aggregate Root

**SampleRequest** is the Aggregate Root. Justification: every sample-related operation — submission, status tracking, decision capture, conversion to RFQ — is anchored to a single sample request that carries the lifecycle state. Status changes, decision feedback, and conversion outcomes only make sense as attributes of one specific `SampleRequest`; there is no meaningful sample-domain operation that does not start from a request reference.

### Inside the Aggregate

Components inside the Sample Lifecycle aggregate (transactional consistency boundary):

- `SampleRequest` — the root entity; identity stable across status transitions.
- `SampleRequestId` — value object wrapping the request identifier issued by NET7 on first persist.
- `SampleStatus` — value object stamped onto the request as the lifecycle advances (state-machine domain).
- `RequestedQuantity` — value object pairing numeric amount and unit (g, kg, ml, sample-pack).
- `RequestedDeliveryDate` — value object with a future-dated target.
- `ApplicationProject` — value object with a project-name string. *Hypothesis: VO inside Sample (OQ-3); could be reframed as a cross-aggregate reference to Workspaces.*
- `SampleProductReference` — value object referencing a Catalog Product by `ProductIdentifier`. *Hypothesis: cardinality is 1..N; OQ-2 contemplates 1:1.*
- `DeliveryAddress` — value object. *Hypothesis: defaulted from Customer Account, override permitted (OQ-7).*
- `DecisionFeedback` — entity, web-only. Recorded after status reaches *Tested*. *Hypothesis: inside Sample (OQ-1); could be its own aggregate if NET7 ingests feedback.*
- `ConversionOutcome` — value object. Records the Sample → RFQ conversion event (RFQ-ID, conversion timestamp, triggering user). *Hypothesis: VO is sufficient; promote to Entity if conversion gains its own state machine.*

The transactional invariants in §4 (INV-1 … INV-7) all hold inside this boundary, including the state-machine invariants on `SampleStatus`, `DecisionFeedback`, and `ConversionOutcome`.

### Outside the Aggregate

Explicitly **not** inside the Sample Lifecycle aggregate:

- `Product` — owned by the **Catalog** aggregate; referenced by `ProductIdentifier`. The sample request never holds Product attributes directly (name, INCI, certification); these are read from Catalog at render time.
- `Customer`, `Contact`, default `DeliveryAddress` — owned by the **Customer Account** aggregate; referenced by ID. The sample request stamps a snapshot of the chosen address but does not own customer master data.
- `RFQRequest`, `Quote` — owned by the **RFQ Lifecycle** aggregate; referenced by ID via `ConversionOutcome`. The conversion event is recorded inside Sample, but the RFQ itself lives in its own consistency boundary.
- `Document` (sample COA, dispatch notice, sample-side compliance docs) — owned by the **Documents & Compliance** aggregate; referenced by ID per status milestone (e.g., a *Shipped* status may carry a dispatch-notice DocumentReference).
- `Workspace` — owned by **Workspaces / Saved Lists**; referenced indirectly via `ApplicationProject` if the project name resolves to a saved workspace. *Hypothesis: indirect via name string for V1; OQ-3.*
- `Order` (post-RFQ commercial order) — owned by **Order Management**; not directly referenced from the sample.

### Discussion

**Multi-Product per SampleRequest, or one Sample per Product?**
- *Pro multi-product (default)*: matches `procurement-hypotheses.md` §5 Stage 4 fields (sample quantity, application, project name, delivery date) which are typically project-bound, not product-bound; reduces user friction; one `ApplicationProject` covers one buyer task.
- *Contra multi-product*: each product has its own fulfilment timeline; status transitions may diverge per product (one shipped, another back-ordered); modelling as one request forces a composite status which is harder to reason about.
- **Provisional decision**: multi-product is permitted via `SampleProductReference` (1..N). State-machine applies to the overall request; per-product fulfilment detail surfaces via the Documents aggregate (per-shipment notices). Revisit per OQ-2 after `erp-discovery.md` §3.c clarifies how NET7 models sample fulfilment cardinality.

**DecisionFeedback — child entity inside Sample, or its own aggregate?**
- *Pro inside (default)*: feedback is meaningless without the parent SampleRequest; its existence is gated by INV-4 (only after *Tested*); single-writer pattern (one feedback per sample) avoids contention; lifecycle naturally bound.
- *Contra inside*: if AOT later treats feedback as a first-class artefact (multi-rater, aggregated insights, NET7 ingest), it should be its own aggregate with its own consistency rules; if Product Developers and Quality Managers contribute separately, the single-feedback assumption breaks.
- **Provisional decision**: inside Sample, single feedback per request, web-only persistence. Revisit per OQ-1 after `business-discovery.md` §3.d clarifies decision-channel reality and `erp-discovery.md` §3.e clarifies whether NET7 accepts feedback writes (CAP-014).

**ApplicationProject — VO inside Sample, or cross-reference to Workspaces?**
- *Pro VO inside (default)*: project name is a string label; copying it onto each sample request is trivial; no cross-aggregate read on common reads.
- *Contra VO inside*: if the same project drives multiple samples, multiple RFQs, and a saved workspace, the project name should be a stable identifier in the Workspaces aggregate, and Sample should reference it. String-based linkage breaks if a workspace is renamed.
- **Provisional decision**: VO with a free-text label for V1. If `business-discovery.md` confirms project-level workflows are common (workspace as primary user-organising concept), refactor to `WorkspaceReference` in 1.4.3 / 1.4.4. Tracked as OQ-3.

## 4. Internal Structure

### Schema

| Name                     | Type          | Owner       | Notes                                                                                       |
|--------------------------|---------------|-------------|---------------------------------------------------------------------------------------------|
| `SampleRequest`          | Entity (Root) | Shared      | Identity is `SampleRequestId`; root of the aggregate; carries `SampleStatus`.               |
| `SampleRequestId`        | Value Object  | ERP-defined | Issued by NET7 on first persist. *Hypothesis: stable across status transitions.*            |
| `SampleStatus`           | Value Object  | ERP-defined | State machine: *Requested → Approved → Shipped → Delivered → Tested → Decided / Cancelled*. |
| `RequestedQuantity`      | Value Object  | Shared      | Numeric amount + unit (g, kg, ml, sample-pack). Web validates against catalogue MOQ.        |
| `RequestedDeliveryDate`  | Value Object  | Shared      | Future-dated target delivery date. Constrained by INV-7.                                    |
| `ApplicationProject`     | Value Object  | Shared      | Project-name string. *Hypothesis: VO inside Sample (OQ-3).*                                 |
| `SampleProductReference` | Value Object  | Shared      | Reference to Catalog `ProductIdentifier`. Cardinality 1..N per OQ-2.                        |
| `DeliveryAddress`        | Value Object  | ERP-defined | *Hypothesis: defaults from Customer Account, override permitted (OQ-7).*                    |
| `DecisionFeedback`       | Entity        | Web         | Web-only. Rating + comment + outcome enum. *Hypothesis: inside Sample (OQ-1).*              |
| `ConversionOutcome`      | Value Object  | Shared      | Records Sample → RFQ event (RFQ-ID, timestamp, user). *Hypothesis: VO; OQ-6 may promote.*   |

### Invariants

Transactional invariants — must hold at every commit of the aggregate:

- **INV-1** (structural). `SampleRequest` references at least one `Product` via `SampleProductReference`. A request without products has no meaning.
- **INV-2** (structural). `SampleRequest` belongs to exactly one `Customer` (via `CustomerReference` to Customer Account). Cross-customer sharing is not permitted at request level.
- **INV-3** (state-machine). `SampleStatus` transitions follow the defined lifecycle: *Requested → Approved → Shipped → Delivered → Tested → Decided*, with *Cancelled* reachable from any state before *Delivered*. Backward transitions are prohibited; status history is append-only.
- **INV-4** (state-machine). `DecisionFeedback` may exist only when `SampleStatus` is *Tested* or *Decided*. Attempting to attach feedback to an earlier status is a transition violation.
- **INV-5** (state-machine). `ConversionOutcome` may be set only when `SampleStatus` is *Tested* or *Decided*. A sample that has not been tested cannot be converted to an RFQ.
- **INV-6** (behavioural). ERP-sourced fields (`SampleRequestId`, `SampleStatus`, ERP-defaulted `DeliveryAddress`) are read-only on the web side. Local mutation is a violation; the only legal write paths are *create* (web → NET7) and *status-sync* (NET7 → web) via the adapter.
- **INV-7** (structural). `RequestedDeliveryDate` is greater than or equal to the request creation date. Past-dated requests are rejected at write time.

### Consistency Rules

- **Transactional (inside aggregate)**: status transitions, decision feedback creation, and conversion-outcome stamping must commit atomically. INV-1 through INV-7 hold at commit time; status-history append and the corresponding INV-3/-4/-5 evaluations occur in the same transaction.
- **Eventual (cross-aggregate)**: `SampleStatus` updates from NET7 are eventually consistent — there will be a delay between fulfilment events on the ERP side and dashboard reflection on the web. *Hypothesis: lag of < 5 minutes acceptable for V1; validate via `business-discovery.md` §3.d on buyer expectations and `erp-discovery.md` §3.f on sync mechanism.* The `ConversionOutcome → RFQRequest` linkage is eventually consistent across the Sample / RFQ aggregate boundary; CC-1 in `capability-map.md` §4 covers the orchestration.

## 5. Web/ERP Ownership and Adapter Boundary

Per-component ownership statement and adapter mediation.

- `SampleRequest` — **Shared**. Master record persists in NET7 once created. Web holds session-side draft state until submission; after submission, web reads through the adapter. Mediated by `Net7SampleAdapter`.
- `SampleRequestId` — **ERP-defined**. NET7 issues on persist. Web may carry a client-generated correlation token for idempotency (see adapter section).
- `SampleStatus` — **ERP-defined**. State machine authority lives in NET7 fulfilment. *Hypothesis: web models the same state names but does not transition them locally (OQ-4).*
- `RequestedQuantity`, `RequestedDeliveryDate`, `ApplicationProject` — **Shared**. Captured on the web side at request creation; persisted onto the request in NET7.
- `SampleProductReference` — **Shared**. Web composes the reference at submission time from the Catalog browse / search context.
- `DeliveryAddress` — **ERP-defined**. Default sourced from Customer Account in NET7. *Hypothesis: per-request override is permitted but stamped onto the request as a snapshot, not back-propagated to Customer master (OQ-7).*
- `DecisionFeedback` — **Web**. Web-only persistence in V1. *Hypothesis: NET7 does not ingest feedback (CAP-014); revisit per OQ-1 / `erp-discovery.md` §3.e.*
- `ConversionOutcome` — **Shared**. Web records the conversion event with the resulting RFQ ID. *Hypothesis: NET7 may also stamp the conversion (OQ-6); if so, web becomes the conformist read.*

### Net7SampleAdapter

The single mediation layer for Sample Lifecycle ↔ NET7 traffic. First aggregate in the platform with a meaningful **write path** — the adapter pattern below is the template for Customer Account, RFQ Lifecycle, and Order Management writes.

- **Read**: sample-request fetch by ID, status updates (single + bulk for dashboard rendering), per-customer history.
- **Status-sync**: ingest NET7-side status events into the web view. *Open question: webhook / event push from NET7 vs scheduled pull (cron). Drives both freshness UX and adapter complexity. See OQ-5.*
- **Write-back**: sample-request creation (web → NET7). Translates the web-side request payload into NET7's expected shape; receives `SampleRequestId` and initial status on success.
- **Idempotency**: write-create is idempotent against a client-generated correlation token submitted with the request. *Hypothesis: NET7 either supports a request-side token natively or surfaces a deduplication semantic on natural keys (Customer + Product + Project + RequestedDeliveryDate). Validate via `erp-discovery.md` §3.e. If neither holds, the adapter implements a web-side dedup window with explicit failure-mode escalation.*
- **Failure mode**: write failures retry with the same correlation token; status-sync failures show a stale-banner with "last-synced-at" timestamp; partial failures (write succeeds, sync lags) are reconciled on the next sync cycle.

### Sprint-2 Hook

The above is the input for the Sprint 2 ERP Boundary Design adapter contract. Open ERP capabilities to validate before contract sign-off:

- Sample-request **write API** vs back-office-only workflow (CAP-011, `erp-discovery.md` §3.e). If write API does not exist, the entire `Net7SampleAdapter.write` path needs replacement (e.g., async hand-off to AOT sales).
- State-machine **ownership and event semantics** — does NET7 emit transition events, or must web poll? (OQ-4, OQ-5.)
- **Idempotency tokens** on writes — natively supported, or web-side dedup required? (Hypothesis above.)
- **DecisionFeedback channel** — does NET7 accept feedback writes (CAP-014), or is feedback purely a web-side artefact? (OQ-1.)

## 6. Cross-Aggregate Relationships and Open Architecture Questions

### Cross-Aggregate ID-References

Sample Lifecycle references other aggregates **by identifier only**. No object navigation crosses these boundaries; consumers fetch the related data via the target aggregate's repository.

- **`ProductReference`** → Catalog aggregate. Composed via `SampleProductReference` at request creation. Catalog renders product master data alongside the sample in dashboard views.
- **`CustomerReference`** → Customer Account aggregate. Stamps the owning customer on the request; INV-2 enforces uniqueness. The Customer Account aggregate provides identity bridging and contact resolution.
- **`DeliveryAddressReference`** → Customer Account aggregate. The default address handle. Per-request override is captured as a `DeliveryAddress` VO snapshot inside Sample; the reference is for default lookup only.
- **`DocumentReference`** → Documents & Compliance aggregate. Per-status milestone documents (e.g., dispatch notice on *Shipped*, sample COA on *Delivered*) are referenced from the request's status history.
- **`RFQReference`** → RFQ Lifecycle aggregate. Set at conversion time inside `ConversionOutcome`. Establishes the Partnership relationship from `bounded-contexts.md` §5 #5.

### Open Architecture Questions

These questions are the primary input to Sprint 2 ERP Boundary Design and may also be answered by `business-discovery.md` / `erp-discovery.md` interviews.

- **OQ-1**. `DecisionFeedback` — inside Sample (current default) or own aggregate? Triggers for re-evaluation: NET7 accepts feedback writes; multiple roles (Product Developer + Quality Manager) contribute independently; feedback drives downstream analytics. Validate via `business-discovery.md` §3.d, §4.b and `erp-discovery.md` §3.e.
- **OQ-2**. Multi-product per `SampleRequest` (current default, 1..N) vs strict one-sample-per-product? Triggers: NET7 fulfils samples per product line individually (separate dispatch, separate COA per product); composite-status modelling becomes painful. Validate via `erp-discovery.md` §3.c, §3.e.
- **OQ-3**. `ApplicationProject` — value object inside Sample (current default) or cross-aggregate reference to Workspaces? Trigger: customer feedback indicates project-level workflows are central; renaming a workspace must propagate. Validate via `business-discovery.md` §3.b, §3.h.
- **OQ-4**. `SampleStatus` state machine — ERP-defined (web is conformist) or web-modelled with status mapping? Trigger: NET7 status vocabulary differs from buyer-facing UX (e.g., NET7 has "in pick" / "QC hold" stages that buyers shouldn't see). May require a status-translation layer in the adapter. Validate via `erp-discovery.md` §3.c, §3.f.
- **OQ-5**. Status synchronisation — push from NET7 (event / webhook) vs scheduled pull on the web side. Drives both freshness UX (CAP-012) and adapter complexity. Validate via `erp-discovery.md` §3.f.
- **OQ-6** *(critical, CC-1)*. `ConversionOutcome` — does the web orchestrate two ERP write operations (read sample, write RFQ), or does NET7 expose a native sample-to-RFQ transition? Decision affects the *Partnership* classification of `bounded-contexts.md` §5 #5 and the RFQ aggregate sketch in 1.4.x. Validate via `erp-discovery.md` §3.e.
- **OQ-7**. `DeliveryAddress` override — per-request optional override (current default) or always default from Customer Account with no override? Trigger: AOT logistics rules may require addresses verified at customer level only. Validate via `business-discovery.md` §3.d and `erp-discovery.md` §3.e.
- **OQ-8**. `SampleRequest` visibility — strictly user-bound (only the requesting contact sees their requests) or shared within the Customer-Org (any authorised contact sees all requests)? Affects auth-scope on read paths. Validate via `business-discovery.md` §3.b on multi-contact handling.

## 7. References

- [`catalog.md`](./catalog.md) — pattern template (7-section structure, methodology, schema-table style); also the `ProductReference` target.
- [`bounded-contexts.md` §4.d, §5](../bounded-contexts.md) — Sample Lifecycle bounded context; Partnership relationship #5 (Sample ↔ RFQ).
- [`capability-map.md` §3.d, §4 CC-1](../capability-map.md) — Sample capabilities CAP-011 … CAP-014; Cross-Context CC-1 Sample-to-RFQ Conversion; `Net7SampleAdapter` naming.
- [`glossary.md`](../glossary.md) — `Sample`, `Sample Request`, `Sample Order`, `ApplicationProject`, `Decision Criteria`, `Workspace` definitions.
- [`AGENTS.md`](../../../AGENTS.md) — engineering charter; ERP Ownership Rules, ERP Integration Rules.
- [`procurement-hypotheses.md`](../../procurement-hypotheses.md) — §5 Stage 4 *Sample Request*, §5 Stage 5 *RFQ* (conversion).
- [`business-discovery.md`](../../business-discovery.md) — §3.d Sample-Request-Workflow heute (Buyer); §4.b Sample-Testing-Prozess (Product Developer).
- [`erp-discovery.md`](../../erp-discovery.md) — §3.c Datenmodell, §3.e Schreib-Operationen, §3.f Sync-Strategie.
