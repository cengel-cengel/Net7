# Capability Map — AOT Procurement Platform

## 1. Purpose

This document enumerates the discrete user capabilities of the AOT Digital Procurement Platform, organised by the eight bounded contexts defined in [`bounded-contexts.md`](./bounded-contexts.md). Each capability carries an explicit Web/ERP boundary, a V1 priority, and discovery hooks that anchor it to the Sprint-0 interview clusters. The audience is engineering and the input target is the Sprint-2 ERP Boundary Design.

All capabilities, ownership classifications, and priorities in this document are **working hypotheses** until validated through `business-discovery.md` and `erp-discovery.md` interviews.

## 2. Methodology

- **Capability = what the user can do.** Capabilities are user-facing, imperative, and verb-led ("Search products by INCI", not "ProductSearchService"). No data-model fields or method signatures appear here — that belongs to the Aggregate Sketches in Task 1.4 ff.
- **Web Role / ERP Role / Boundary.** Each capability is decomposed into what the web platform owns, what the ERP **TopM NET7** owns, and which adapter mediates between them. ERP ownership follows `AGENTS.md` ERP Ownership Rules: products, prices, stock, contracts, batch traceability, documents, and order history live in NET7; the web platform owns UX, sessions, search, dashboards, and workflows.
- **Hypotheses are explicit.** Where ERP capability is unverified, the role line carries `Hypothesis:` or `Open question:`. Capability priority and boundary classification are themselves hypotheses pending discovery.
- **Bounded-context anchoring.** Every capability sits in exactly one primary context (`bounded-contexts.md` §4.a–§4.h) and may touch others. Cross-context workflows live in §4 Cross-Context Capabilities below.
- **V1 priorities** follow `procurement-hypotheses.md` §9: P1 = product search, product details, documents, sample requests; P2 = RFQ workflow, dashboard, reorder; P3/Future = personalisation, recommendations, analytics.

## 3. Capability Inventory by Context

Capabilities are numbered `CAP-NNN` continuously across all sub-sections. Each capability uses the same eight-field schema: ID, Name, Bounded Context, Web Role, ERP Role, Boundary, V1 Priority, Discovery Hooks.

### a. Catalog

- **ID**: CAP-001
- **Name**: Browse and filter the product catalogue.
- **Bounded Context**: `bounded-contexts.md` §4.a Catalog.
- **Web Role**: Render category navigation, faceted filters (segment, certification, application, origin), and SEO-friendly listing pages.
- **ERP Role**: Provide master product data and metadata that drives faceting (per `AGENTS.md` ERP Ownership Rules). *Hypothesis: facet values come fully from ERP master data without web-side enrichment.*
- **Boundary**: `Net7ProductAdapter` — read-only, served from a web-side index/cache for latency.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `business-discovery.md` §3.a, §3.b — typical browsing patterns.
  - `erp-discovery.md` §3.c, §3.d — exposed entities and filter mechanisms.
  - `erp-discovery.md` §3.f — sync/cache strategy.

---

- **ID**: CAP-002
- **Name**: Smart search by INCI / CAS / botanical name / application.
- **Bounded Context**: `bounded-contexts.md` §4.a Catalog.
- **Web Role**: Operate a search index supporting INCI, CAS, botanical, common name, application, and certification facets; type-ahead, ranking, synonym handling.
- **ERP Role**: Source of truth for searchable fields. *Hypothesis: NET7 exposes INCI/CAS as queryable structured fields, not free-text.*
- **Boundary**: `Net7ProductAdapter` for ingestion into the web search index; the index itself lives in web infrastructure.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §6 *Smart Search*.
  - `business-discovery.md` §3.b — actual search behaviour.
  - `erp-discovery.md` §3.d, §3.f — searchable fields and sync strategy.

---

- **ID**: CAP-003
- **Name**: View product detail with specifications and certifications.
- **Bounded Context**: `bounded-contexts.md` §4.a Catalog (with read-only excursions into §4.b Pricing & §4.c Documents).
- **Web Role**: Render the product detail page — specs, applications, INCI/CAS, origin, certifications, available documents, customer-specific price, and document-completeness indicator.
- **ERP Role**: Provide master data, the customer-specific price slice, and document inventory. No web-side pricing logic, per `AGENTS.md`.
- **Boundary**: `Net7ProductAdapter` (master), `Net7PricingAdapter` (price slice), `Net7DocumentAdapter` (document index).
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §5 Stage 1.
  - `business-discovery.md` §3.b, §3.f — what info buyers expect.
  - `erp-discovery.md` §3.c, §3.d — entity shape and read mechanisms.

---

- **ID**: CAP-004
- **Name**: Compare products side-by-side.
- **Bounded Context**: `bounded-contexts.md` §4.a Catalog.
- **Web Role**: Allow the buyer to assemble a comparison set and render side-by-side comparison of MOQ, origin, lead time, certifications, packaging, and availability.
- **ERP Role**: Source of comparison fields. *Hypothesis: all comparison fields are available per product without aggregation.*
- **Boundary**: `Net7ProductAdapter` — pure read.
- **V1 Priority**: P2. *Open question: `procurement-hypotheses.md` §6 treats comparison as a core UX hypothesis, but §9 does not list compare under V1 P1. Default P2 here; revisit after buyer interviews.*
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §6 *Product Comparison*.
  - `business-discovery.md` §3.b — does the buyer actively compare on a portal today?
  - `erp-discovery.md` §3.d — fields available per product.

---

- **ID**: CAP-005
- **Name**: Discover comparable alternatives when a product is unavailable.
- **Bounded Context**: `bounded-contexts.md` §4.a Catalog.
- **Web Role**: When availability is "Made-to-order" or "Low", surface comparable alternatives based on application, certification, and origin similarity.
- **ERP Role**: Provide availability signal and product taxonomy. *Hypothesis: similarity inference happens web-side from ERP metadata; NET7 does not maintain an "alternatives" entity.*
- **Boundary**: `Net7ProductAdapter`; web-side similarity logic.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §6 *Alternatives*.
  - `erp-discovery.md` §3.d — availability granularity.
  - `business-discovery.md` §3.c, §3.g — buyer expectation when unavailable.

### b. Pricing & Contracts

- **ID**: CAP-006
- **Name**: View customer-specific price for a product.
- **Bounded Context**: `bounded-contexts.md` §4.b Pricing & Contracts.
- **Web Role**: Render the customer's contracted price on product detail and order/RFQ flows. Read-only display; no calculation web-side.
- **ERP Role**: Compute and serve the customer-specific price (contract application, pricing tier, volume). `AGENTS.md` hard rule: pricing logic stays in ERP.
- **Boundary**: `Net7PricingAdapter` — synchronous read at request time. *Hypothesis: latency acceptable for UX; otherwise a short-TTL snapshot is needed.*
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §7 ERP Assumptions.
  - `erp-discovery.md` §3.b, §3.d — auth scope and read mechanism.
  - `erp-discovery.md` §3.f — freshness expectations.

---

- **ID**: CAP-007
- **Name**: View contract terms (MOQ, payment terms, validity).
- **Bounded Context**: `bounded-contexts.md` §4.b Pricing & Contracts.
- **Web Role**: Render MOQ, payment terms, and contract validity for a product/customer combination, scoped to the authenticated session.
- **ERP Role**: Source of contract metadata. *Hypothesis: contract terms are exposed alongside price in the same read path.*
- **Boundary**: `Net7PricingAdapter`.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `business-discovery.md` §3.h — supplier-relationship signals.
  - `erp-discovery.md` §3.c, §3.d.

### c. Documents & Compliance

- **ID**: CAP-008
- **Name**: See document-completeness indicator on product / order.
- **Bounded Context**: `bounded-contexts.md` §4.c Documents & Compliance.
- **Web Role**: Aggregate the per-product or per-order required-vs-available document set; render a "8/8 documents available" indicator.
- **ERP Role**: Provide the document inventory per product / batch / order. *Hypothesis: the required-document set is an industry/segment convention, not ERP metadata; web defines required sets.*
- **Boundary**: `Net7DocumentAdapter` for inventory; web-side completeness rules.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §6 *Document Completeness*.
  - `business-discovery.md` §3.f — required-doc set per buyer.
  - `erp-discovery.md` §3.d — doc inventory access.

---

- **ID**: CAP-009
- **Name**: Download a single document (SDS / COA / TDS / certificate).
- **Bounded Context**: `bounded-contexts.md` §4.c Documents & Compliance.
- **Web Role**: Resolve customer entitlement, fetch the document URL or stream from ERP, surface the file to the user, and track download events for analytics.
- **ERP Role**: Source and serve the document. *Hypothesis: document URLs may be stable references or signed/short-lived links — verification needed.*
- **Boundary**: `Net7DocumentAdapter`.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `business-discovery.md` §3.f — current download pain.
  - `erp-discovery.md` §3.b, §3.d, §3.h — auth, read, URL lifecycle.

---

- **ID**: CAP-010
- **Name**: Bulk-download all required documents for a product or order.
- **Bounded Context**: `bounded-contexts.md` §4.c Documents & Compliance.
- **Web Role**: Aggregate the required document set, package it as a ZIP / PDF bundle, deliver it to the customer.
- **ERP Role**: Provide the individual documents on demand. *Hypothesis: bundling happens web-side; NET7 has no native bulk-export.*
- **Boundary**: `Net7DocumentAdapter` (per-doc pulls); web-side bundling.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.f — actual demand for bulk vs single.
  - `erp-discovery.md` §3.h — rate-limits relevant to bulk pulls.

### d. Sample Lifecycle

- **ID**: CAP-011
- **Name**: Request a sample with project context.
- **Bounded Context**: `bounded-contexts.md` §4.d Sample Lifecycle.
- **Web Role**: Render the request form (sample quantity, application, project name, requested delivery date, comment); submit to ERP.
- **ERP Role**: Accept and persist the sample request, route to internal fulfilment. *Hypothesis: NET7 has either a native sample-request entity or treats samples as a special order class.*
- **Boundary**: `Net7SampleAdapter` — write side; idempotent submission required.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §5 Stage 4.
  - `business-discovery.md` §3.d.
  - `erp-discovery.md` §3.c, §3.e.

---

- **ID**: CAP-012
- **Name**: Track sample request status.
- **Bounded Context**: `bounded-contexts.md` §4.d Sample Lifecycle.
- **Web Role**: Display current sample status (requested / in fulfilment / dispatched / delivered) on the customer dashboard.
- **ERP Role**: Provide status updates as fulfilment progresses. *Hypothesis: status events available via polling or webhook; sync mechanism open.*
- **Boundary**: `Net7SampleAdapter` — read; sync strategy is an open question (`erp-discovery.md` §3.f).
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `business-discovery.md` §3.d.
  - `erp-discovery.md` §3.f — sync mechanism.

---

- **ID**: CAP-013
- **Name**: Convert a sample request to an RFQ.
- **Bounded Context**: `bounded-contexts.md` §4.d Sample Lifecycle + §4.e RFQ Lifecycle (cross-context — see §4 below).
- **Web Role**: Carry sample data (product, quantity, project context) into the RFQ creation form, eliminating duplicate entry.
- **ERP Role**: *Open question: web orchestrates two ERP write operations (read sample → write RFQ), or NET7 has a native conversion transition.*
- **Boundary**: `Net7SampleAdapter` (read sample) + `Net7RfqAdapter` (write RFQ). Boundary depends on the open question above.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §5 Stage 5.
  - `erp-discovery.md` §3.e.

---

- **ID**: CAP-014
- **Name**: Submit decision feedback after sample testing.
- **Bounded Context**: `bounded-contexts.md` §4.d Sample Lifecycle.
- **Web Role**: Capture buyer / product-developer decision (accept / reject / inconclusive) with optional notes; route to AOT sales.
- **ERP Role**: *Open question: does NET7 accept sample-decision feedback writes today, or is feedback email-only?*
- **Boundary**: Open. `Net7SampleAdapter` write if API exists; otherwise web-only persistence + email notification.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.d.
  - `erp-discovery.md` §3.e.

### e. RFQ Lifecycle

- **ID**: CAP-015
- **Name**: Submit an RFQ.
- **Bounded Context**: `bounded-contexts.md` §4.e RFQ Lifecycle.
- **Web Role**: Render RFQ form (product lines, quantity, target lead time, payment-term preference, comment); submit to ERP.
- **ERP Role**: Accept the RFQ and generate an official quote. *Hypothesis: quote generation is a back-office workflow today; a write API may not exist yet.*
- **Boundary**: `Net7RfqAdapter` — *open question whether a write API exists*.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §5 Stage 5.
  - `erp-discovery.md` §3.e.

---

- **ID**: CAP-016
- **Name**: Compare quotes side-by-side.
- **Bounded Context**: `bounded-contexts.md` §4.e RFQ Lifecycle.
- **Web Role**: Allow the buyer to compare multiple quotes (price, MOQ, lead time, terms, validity).
- **ERP Role**: Source of quote data per RFQ.
- **Boundary**: `Net7RfqAdapter` — read.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.e.
  - `erp-discovery.md` §3.d.

---

- **ID**: CAP-017
- **Name**: Track RFQ status and quote validity.
- **Bounded Context**: `bounded-contexts.md` §4.e RFQ Lifecycle.
- **Web Role**: Display RFQ state (submitted / quoted / accepted / expired) and quote-validity countdown.
- **ERP Role**: Provide RFQ + quote state.
- **Boundary**: `Net7RfqAdapter`.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.e.
  - `erp-discovery.md` §3.f.

### f. Order Management

- **ID**: CAP-018
- **Name**: Place an order from an accepted quote.
- **Bounded Context**: `bounded-contexts.md` §4.f Order Management.
- **Web Role**: Buyer accepts a quote → web sends order placement to ERP and surfaces confirmation.
- **ERP Role**: Persist the order, trigger fulfilment, return order ID. *Hypothesis: NET7 supports quote-to-order transition (web-orchestrated or ERP-native).*
- **Boundary**: `Net7OrderAdapter` — idempotent write.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §9 — dashboard / reorder.
  - `erp-discovery.md` §3.e.

---

- **ID**: CAP-019
- **Name**: View order history.
- **Bounded Context**: `bounded-contexts.md` §4.f Order Management.
- **Web Role**: Render order history (date range, status, line items) for the authenticated customer.
- **ERP Role**: Provide order history scoped to the customer.
- **Boundary**: `Net7OrderAdapter` — read.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.h.
  - `erp-discovery.md` §3.d.

---

- **ID**: CAP-020
- **Name**: Track delivery status.
- **Bounded Context**: `bounded-contexts.md` §4.f Order Management.
- **Web Role**: Display delivery state (in production / shipped / in transit / delivered) with carrier reference where available.
- **ERP Role**: Source of delivery events. *Hypothesis: detail level open — carrier + tracking-id only, or staged events.*
- **Boundary**: `Net7OrderAdapter`.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.h.
  - `erp-discovery.md` §3.d.

---

- **ID**: CAP-021
- **Name**: Reorder from history.
- **Bounded Context**: `bounded-contexts.md` §4.f Order Management + §4.a Catalog + §4.b Pricing & Contracts (cross-context).
- **Web Role**: Pre-fill an RFQ or direct order based on a previous order; surface current price and availability for those SKUs.
- **ERP Role**: Provide current price and availability for the previously ordered SKUs.
- **Boundary**: `Net7OrderAdapter` (history), `Net7ProductAdapter` (current state), `Net7PricingAdapter` (current price).
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `procurement-hypotheses.md` §5 Stage 6.
  - `business-discovery.md` §3.h.

### g. Customer Account

- **ID**: CAP-022
- **Name**: Authenticate (sign in / out).
- **Bounded Context**: `bounded-contexts.md` §4.g Customer Account.
- **Web Role**: Run the auth flow (mechanism TBD — OAuth, email-link, SSO), establish session, scope access for downstream calls.
- **ERP Role**: Map the web identity to the ERP customer ID for downstream calls. *Hypothesis: identity bridging happens at the adapter layer; NET7 does not host the auth flow itself.*
- **Boundary**: Auth provider (web infrastructure) + `Net7CustomerAdapter` for the identity bridge.
- **V1 Priority**: P1.
- **Discovery Hooks**:
  - `erp-discovery.md` §3.b — auth mechanisms NET7 supports.
  - `business-discovery.md` §3.b — multi-contact handling expectations.

---

- **ID**: CAP-023
- **Name**: Manage profile (contacts, addresses).
- **Bounded Context**: `bounded-contexts.md` §4.g Customer Account.
- **Web Role**: Render profile UI; submit changes.
- **ERP Role**: *Open question: are profile writes supported via NET7 API, or only via the TopM portal? If portal-only, this capability is read-only on the web.*
- **Boundary**: `Net7CustomerAdapter`.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `erp-discovery.md` §3.e — write-operations inventory.
  - `business-discovery.md` §3.h.

---

- **ID**: CAP-024
- **Name**: Manage account preferences.
- **Bounded Context**: `bounded-contexts.md` §4.g Customer Account.
- **Web Role**: Web-only preferences (notification settings, default project context, list ordering).
- **ERP Role**: None — web-only state.
- **Boundary**: Web database; no adapter.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.b — preference patterns.

### h. Workspaces / Saved Lists

- **ID**: CAP-025
- **Name**: Create and manage a workspace.
- **Bounded Context**: `bounded-contexts.md` §4.h Workspaces / Saved Lists.
- **Web Role**: Create, rename, archive, and delete workspaces (project-named buckets such as "Anti Aging Serum"). Pure web concept.
- **ERP Role**: None.
- **Boundary**: Web database; no adapter.
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.b, §3.h.
  - `procurement-hypotheses.md` §6 *Saved Lists*.

---

- **ID**: CAP-026
- **Name**: Save products to a workspace.
- **Bounded Context**: `bounded-contexts.md` §4.h Workspaces / Saved Lists.
- **Web Role**: Add and remove products from a workspace; reference Catalog product identifiers.
- **ERP Role**: None.
- **Boundary**: Web database; product IDs sourced from Catalog (Conformist relationship to §4.a in `bounded-contexts.md`).
- **V1 Priority**: P2.
- **Discovery Hooks**:
  - `business-discovery.md` §3.h.
  - `procurement-hypotheses.md` §6 *Saved Lists*.

## 4. Cross-Context Capabilities

Workflows that span two or more bounded contexts. Each carries explicit boundary-ownership questions because the orchestration seam is where Web/ERP responsibility is least obvious.

**CC-1. Sample-to-RFQ Conversion**

- **Contexts involved**: Sample Lifecycle (§4.d) + RFQ Lifecycle (§4.e) + Customer Account (§4.g).
- **Workflow**: After a sample is delivered and approved, the buyer triggers conversion in the dashboard. The web platform carries `SampleRequest` data — product, quantity, project context — into the RFQ creation form, eliminating duplicate entry. The buyer reviews, adjusts, and submits as RFQ. References CAP-013 + CAP-015. *ApplicationProject* persists across the transition.
- **V1 Priority**: P2.
- **Open question — boundary ownership**: Does the web platform orchestrate two ERP write operations (read sample, write RFQ), or does NET7 expose a native sample-to-RFQ transition? Affects the *Partnership* classification of the §5 #5 edge in `bounded-contexts.md`. Validate via `erp-discovery.md` §3.e.

**CC-2. Reorder via Order History**

- **Contexts involved**: Order Management (§4.f) + Catalog (§4.a) + Pricing & Contracts (§4.b) + Customer Account (§4.g).
- **Workflow**: The buyer selects a past order from history. The web platform pulls current product data, current price, and current availability for the SKUs in that order. The buyer adjusts quantity or substitutes products and submits as a new RFQ or — depending on contract terms — a direct order. References CAP-021 + CAP-006 + CAP-015 (or CAP-018).
- **V1 Priority**: P2.
- **Open question — boundary ownership**: Is one-click reorder against legacy contract terms acceptable, or does AOT require a fresh RFQ for current price every time? Buyer expectation needs validation in `business-discovery.md` §3.h, alongside ERP capabilities in `erp-discovery.md` §3.d, §3.e.

**CC-3. Document Bulk Download per Workspace**

- **Contexts involved**: Workspaces / Saved Lists (§4.h) + Documents & Compliance (§4.c) + Catalog (§4.a) + Customer Account (§4.g).
- **Workflow**: From a workspace ("Anti Aging Serum"), the buyer triggers a bulk-document download. The web platform iterates the workspace's product references, aggregates the required-document set per product, bundles all documents, and delivers a ZIP/PDF to the buyer. References CAP-025 + CAP-026 + CAP-010.
- **V1 Priority**: P3 / Future. Workspaces themselves are P2; combined cross-context demand needs explicit validation before V1 commitment.
- **Open question — boundary ownership**: Web-side bundling is the default assumption (NET7 has no bulk-export — see `erp-discovery.md` §3.h). Real demand for project-scoped bulk download is unverified — does buyer organisation actually run sourcing per project, or per single product? Validate via `business-discovery.md` §3.f, §3.b.

## 5. Open Architecture Questions

Capabilities with the highest Web/ERP boundary uncertainty — these are the primary input to Sprint 2 ERP Boundary Design.

- **CAP-002 search index ownership**: push from NET7 vs. scheduled pull on the web side. Affects sync architecture for the entire catalogue surface.
- **CAP-004 compare priority**: P1 vs P2. `procurement-hypotheses.md` §6 treats it as core UX, but §9 puts comparison outside V1 P1. Default P2 here; revisit after buyer interviews.
- **CAP-006 pricing read latency**: synchronous per-request from NET7 vs cached short-TTL snapshot. Trade-off freshness against UX; depends on `erp-discovery.md` §3.h latency findings.
- **CAP-013 sample-to-RFQ conversion**: web-orchestrated vs ERP-native transition (see CC-1).
- **CAP-014 decision-feedback channel**: does NET7 accept feedback writes, or is the channel email-only?
- **CAP-015 RFQ submission**: does NET7 expose a quote-creation API today, or is quote generation back-office only?
- **CAP-018 order placement**: quote-to-order is ERP-native or web-orchestrated.
- **CAP-023 profile writes**: API or portal-only? If portal-only, web becomes read-only on this capability.
- **Cross-cap — customer-ID stability**: does NET7 customer-ID survive merges, renames, or successor accounts? Affects every authenticated capability (CAP-006, -007, -011, -015, -018, -019, -020, -022, -023).

Hooks for follow-up tasks:

- **Task 1.3 — Ubiquitous Language Glossary**: Each capability uses terminology that needs reconciling between AOT-internal usage, industry standard (INCI, CAS, COA, SDS, TDS), and NET7-internal terms.
- **Task 1.4 ff — Aggregate Sketches**: Each P1 capability needs its consistency boundary mapped, especially CAP-003 (multi-source render), CAP-006 (pricing freshness), CAP-008 (completeness aggregation), CAP-011 (sample submission idempotency).
- **Sprint 2 — ERP Boundary Design**: This map is the direct input for adapter contract definition (`Net7ProductAdapter`, `Net7PricingAdapter`, `Net7DocumentAdapter`, `Net7SampleAdapter`, `Net7RfqAdapter`, `Net7OrderAdapter`, `Net7CustomerAdapter`).

## 6. References

- [`bounded-contexts.md`](./bounded-contexts.md) — eight bounded contexts as the structural backbone.
- [`AGENTS.md`](../../AGENTS.md) — engineering charter; ERP Ownership Rules and ERP Integration Rules.
- [`procurement-hypotheses.md`](../procurement-hypotheses.md) — §5 Buyer Journey, §6 UX Hypotheses, §9 V1 Feature Priorities.
- [`business-discovery.md`](../business-discovery.md) — buyer-behaviour discovery (Buyer, Product Developer, Quality Manager).
- [`erp-discovery.md`](../erp-discovery.md) — ERP-capability discovery (TopM Technical, Account/Commercial).
- [`ROADMAP.md`](../../ROADMAP.md) — Sprint 1 — Domain Modeling; Sprint 2 — ERP Boundary Design.
