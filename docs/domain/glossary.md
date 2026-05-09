# Glossary — AOT Procurement Platform

## 1. Purpose

This glossary fixes the shared vocabulary of the AOT Digital Procurement Platform across two language registers: the English of the architecture documentation (`bounded-contexts.md`, `capability-map.md`, `AGENTS.md`) and the German of the discovery interviews (`business-discovery.md`, `erp-discovery.md`). The audience is engineering, discovery interviewers, and Sprint-2 ERP Boundary Design.

This is a **living document**. Sprint-1 follow-ups (Aggregate Sketches, Task 1.4 ff) and the Sprint-0 discovery streams (TopM technical, customer interviews) will surface new terms and refine existing definitions. Update this file whenever a term carries domain meaning that is not already documented here.

## 2. Conventions

- **Language**: English is the lemma. German translations are added where an established term exists; otherwise the DE field is `[no DE equivalent]` (DDD vocabulary) or omitted (international acronyms, regulations).
- **Capitalisation**: ProperCase for domain concepts (`Bounded Context`, `Sample Request`); ALL-CAPS for acronyms (`INCI`, `CAS`, `SDS`); regulation names follow their official spelling.
- **Cross-references**: short pointers — `bounded-contexts.md §4.a`, `capability-map.md CAP-002`, `business-discovery.md §3.b`. No URLs.
- **Hypothesis marking**: when a definition depends on unverified ERP behaviour or pending discovery, the line carries `Hypothesis:`.
- **Sortation**: §3 is alphabetical by EN lemma. §4 groups the same terms by category for targeted lookup, term names only.
- **Plural**: lemma is singular. Plural use is unmarked.

## 3. Glossary

### ACL (Anti-Corruption Layer)

- **DE**: [no DE equivalent]
- **Definition**: A defensive translation layer between two bounded contexts that prevents the upstream model from polluting the downstream model. On this platform, the web side wraps every NET7 read/write to translate ERP shape into UX-relevant contracts.
- **Used in**: `bounded-contexts.md` §5 #1, #4; `capability-map.md` CAP-002, CAP-006 boundary.

### Adapter

- **DE**: [no DE equivalent]
- **Definition**: An infrastructure component that mediates between the application core and an external system. All ERP I/O on this platform happens through adapters in `apps/api/src/infrastructure/erp/adapters/*`.
- **Used in**: `AGENTS.md` ERP Integration Rules; ADR 0001 §2.5; `capability-map.md` (`Net7ProductAdapter`, `Net7PricingAdapter`, …).

### Aggregate

- **DE**: [no DE equivalent]
- **Definition**: A DDD cluster of domain objects treated as a single unit for data changes. Each aggregate has a root entity and defines its consistency boundary. Per-context aggregate sketches land in Sprint-1 Task 1.4 ff.
- **Used in**: `AGENTS.md` Architecture Rules.

### Allergen Statement

- **DE**: Allergen-Erklärung
- **Definition**: A regulatory document declaring presence or absence of allergens in a raw material. Required for AOT food-segment customers and for cosmetics with allergen-disclosure obligations.
- **Used in**: `bounded-contexts.md` §4.c; `business-discovery.md` §3.f.

### Application

- **DE**: Anwendung
- **Definition**: The intended use case of a raw material — for example *emulsifier*, *anti-aging active*, *preservative*, *flavour*. Used as a primary search facet in the catalogue.
- **Used in**: `bounded-contexts.md` §4.a; `capability-map.md` CAP-002.

### ApplicationProject

- **DE**: Anwendungs-Projekt
- **Definition**: AOT-internal artefact that captures a buyer's project context (e.g., "Anti Aging Serum", "Functional Beverage") across sample requests, RFQs, and orders. Persists across the sample-to-RFQ transition.
- **Used in**: `bounded-contexts.md` §4.d, §4.h; `capability-map.md` CC-1.

### Batch

- **DE**: Charge
- **Definition**: A discrete production run of a raw material, identified by a batch / lot number. Each batch carries its own COA and may carry batch-specific certificates. NET7 is source of truth for batch traceability.
- **Used in**: `bounded-contexts.md` §4.c, §4.f; `AGENTS.md` ERP Ownership Rules.

### Bio-Zertifikat

- **DE**: (already in DE)
- **Definition**: Organic certification documenting compliance with `EU-Bio-VO 2018/848` or an accepted equivalent. Issued by an accredited certification body and versioned per validity period.
- **Used in**: `bounded-contexts.md` §4.c; `business-discovery.md` §3.f, §3.g.

### Botanical Name

- **DE**: botanischer Name
- **Definition**: The Latin binomial of a plant-derived raw material (e.g., *Lavandula angustifolia*). A search facet alongside INCI and common name.
- **Used in**: `bounded-contexts.md` §4.a; `capability-map.md` CAP-002.

### Bounded Context

- **DE**: [no DE equivalent]
- **Definition**: A DDD pattern — a clear linguistic and ownership boundary inside a software system. Each bounded context has its own ubiquitous language and aggregates. The AOT platform is decomposed into eight contexts (`bounded-contexts.md` §4).
- **Used in**: `bounded-contexts.md` throughout.

### Buyer Journey

- **DE**: Beschaffungs-Reise
- **Definition**: The end-to-end flow a strategic procurement buyer goes through, from product search to repeat procurement. Modelled as six stages in `procurement-hypotheses.md` §5; subject to discovery validation.
- **Used in**: `procurement-hypotheses.md` §5.

### CAS

- **DE**: (international)
- **Definition**: Chemical Abstracts Service number. A unique identifier for a chemical substance. Primary search facet for cosmetics and food ingredients.
- **Used in**: `bounded-contexts.md` §4.a; `capability-map.md` CAP-002.

### Certificate

- **DE**: Zertifikat
- **Definition**: A document attesting compliance with a specific regulation or standard (organic, kosher, halal, fair-trade). Versioned per validity period and per batch where applicable.
- **Used in**: `bounded-contexts.md` §4.c; `capability-map.md` CAP-008, CAP-009.

### COA (Certificate of Analysis)

- **DE**: (international)
- **Definition**: Per-batch document detailing analytical test results — purity, contaminants, active content, microbiology. Required for raw-material acceptance in regulated industries.
- **Used in**: `bounded-contexts.md` §4.c; `business-discovery.md` §3.f.

### Conformist

- **DE**: [no DE equivalent]
- **Definition**: A DDD relationship type: the downstream context conforms to the upstream model without translation. Used where ERP shape is acceptable directly.
- **Used in**: `bounded-contexts.md` §5 #3, #6, #9.

### Customer/Supplier

- **DE**: [no DE equivalent]
- **Definition**: A DDD relationship type: an upstream–downstream relationship where the supplier provides what the customer needs. Most ERP→Web edges on this platform are Customer/Supplier with ACL.
- **Used in**: `bounded-contexts.md` §5 #1, #2, #4, #8, #10.

### Customer-ID

- **DE**: (international)
- **Definition**: The stable identifier for a customer in NET7. Web auth must be bridged to the NET7 customer-ID before any pricing or contract read. *Hypothesis: ID survives merges and renames; validation pending (`erp-discovery.md` §3.c).*
- **Used in**: `capability-map.md` CAP-022; `erp-discovery.md` §3.c.

### Decision Criteria

- **DE**: Entscheidungs-Kriterien
- **Definition**: The factors a buyer weighs when choosing between suppliers — quality consistency, documentation completeness, lead-time reliability, supplier responsiveness, price, sustainability. Hypothesis-ranked in `procurement-hypotheses.md` §4.
- **Used in**: `procurement-hypotheses.md` §4.

### Domain Event

- **DE**: [no DE equivalent]
- **Definition**: A DDD pattern: an immutable record of something that happened in the domain (e.g., `SampleRequested`, `OrderPlaced`). Used to decouple bounded contexts and enable async workflows.
- **Used in**: `AGENTS.md` Architecture Rules.

### EU-Bio-VO 2018/848

- **DE**: (DE-specific)
- **Definition**: EU regulation on organic production and labelling. Defines what "organic" means and who can certify it. Drives the Bio-Zertifikat document type.
- **Used in**: `bounded-contexts.md` §4.c; `erp-discovery.md` §5.a.

### EU-Kosmetik-VO 1223/2009

- **DE**: (DE-specific)
- **Definition**: EU Cosmetics Regulation. Sets safety, labelling, and notification requirements for cosmetic products and ingredients. Drives SDS, allergen, and INCI requirements.
- **Used in**: `bounded-contexts.md` §4.c; `erp-discovery.md` §5.a.

### FDA

- **DE**: (international)
- **Definition**: U.S. Food and Drug Administration. Relevant for AOT customers exporting to or selling in the U.S. market. Drives parallel-track compliance requirements alongside EU regulations.
- **Used in**: `erp-discovery.md` §5.a.

### INCI

- **DE**: (international)
- **Definition**: International Nomenclature of Cosmetic Ingredients. The standardised naming system for cosmetic ingredient declarations on labels. Primary search facet on the AOT catalogue.
- **Used in**: `capability-map.md` CAP-002; `procurement-hypotheses.md` §6.

### Kosher / Halal

- **DE**: (international)
- **Definition**: Religious certification documenting compliance with Jewish (kosher) or Islamic (halal) dietary law. Surfaced as an optional document type on products targeting those markets.
- **Used in**: `bounded-contexts.md` §4.c; `capability-map.md` CAP-009.

### Lead Time

- **DE**: Lieferzeit
- **Definition**: The elapsed time between order placement and delivery. A primary procurement KPI and decision criterion, especially volatile for natural raw materials with seasonality.
- **Used in**: `procurement-hypotheses.md` §4, §5 Stage 2.

### LFGB

- **DE**: (DE-specific)
- **Definition**: Lebensmittel- und Futtermittelgesetzbuch — the German food and feed code. Drives food-grade compliance for AOT food-segment customers in DACH.
- **Used in**: `bounded-contexts.md` §4.c; `erp-discovery.md` §5.a.

### Lot

- **DE**: Charge
- **Definition**: Synonym for *Batch*. See **Batch**.
- **Used in**: `bounded-contexts.md` §4.c, §4.f.

### Mixed Buyer

- **DE**: gemischter Einkäufer
- **Definition**: A buyer profile that combines strategic and tactical procurement responsibilities. One of the three buyer-type options in the discovery interview profile (`business-discovery.md` §2).
- **Used in**: `business-discovery.md` §2.

### MOQ

- **DE**: (international)
- **Definition**: Minimum Order Quantity. Per-product or per-customer minimum required for an order to be accepted. Set by ERP contract terms.
- **Used in**: `bounded-contexts.md` §4.b; `capability-map.md` CAP-007.

### NET7

- **DE**: (international)
- **Definition**: The TopM ERP product family AOT uses. Single source of truth for products, prices, contracts, stock, documents, batches, and order history per `AGENTS.md` ERP Ownership Rules.
- **Used in**: `AGENTS.md` throughout; `erp-discovery.md` throughout.

### Net7-Adapter

- **DE**: (international)
- **Definition**: The web-side infrastructure layer wrapping every NET7 read/write. Concrete adapters identified in `capability-map.md`: `Net7ProductAdapter`, `Net7PricingAdapter`, `Net7DocumentAdapter`, `Net7SampleAdapter`, `Net7RfqAdapter`, `Net7OrderAdapter`, `Net7CustomerAdapter`. *Hypothesis: this list is sufficient for V1; revisit during Sprint 2 ERP Boundary Design.*
- **Used in**: `capability-map.md`; ADR 0001 §2.5.

### Open Host Service

- **DE**: [no DE equivalent]
- **Definition**: A DDD relationship type: a context publishes a stable, well-documented protocol for any downstream consumer. Listed in the §5 vocabulary of `bounded-contexts.md` but not currently in use on this platform.
- **Used in**: `bounded-contexts.md` §5 (vocabulary list).

### Order

- **DE**: Auftrag
- **Definition**: A confirmed customer order, derived from an accepted Quote. Persisted and processed by NET7; surfaced read-only on the web dashboard.
- **Used in**: `bounded-contexts.md` §4.f; `capability-map.md` CAP-018, CAP-019.

### Order Line

- **DE**: Auftragsposition
- **Definition**: A single line item within an Order, referencing one product or variant with quantity, price, and optionally a batch assignment.
- **Used in**: `bounded-contexts.md` §4.f.

### Organic

- **DE**: Bio / organisch
- **Definition**: AOT's primary positioning. In this glossary specifically: certified organic per `EU-Bio-VO 2018/848` or an accepted equivalent. AOT's catalogue is exclusively organic raw materials.
- **Used in**: `AGENTS.md` Project Context.

### Origin

- **DE**: Herkunft
- **Definition**: The geographic source of a raw material — country, region, sometimes cooperative. A search facet and decision criterion, especially for fair-trade and traceability.
- **Used in**: `bounded-contexts.md` §4.a; `capability-map.md` CAP-002.

### Partnership

- **DE**: [no DE equivalent]
- **Definition**: A DDD relationship type: two contexts are co-developed with shared success goals. Used on Sample → RFQ and RFQ → Order edges in `bounded-contexts.md` §5 #5, #7.
- **Used in**: `bounded-contexts.md` §5 #5, #7.

### Pricing Tier

- **DE**: Preis-Stufe
- **Definition**: A NET7 customer attribute that determines which contract prices apply. *Hypothesis: tier is set per customer in NET7 master data, not derived from volume on the web side; validation pending (`erp-discovery.md` §3.c, §3.d).*
- **Used in**: `bounded-contexts.md` §4.b, §4.g.

### Procurement KPI

- **DE**: Beschaffungs-KPI
- **Definition**: A measurable indicator AOT customers use to evaluate their procurement function — typical examples: lead time, scrap rate, supplier risk score, total cost of ownership.
- **Used in**: `business-discovery.md` §3.a.

### Quote

- **DE**: Angebot
- **Definition**: A formal price/availability offer from AOT to the buyer in response to an RFQ. Carries a validity window. NET7 is the source of truth for quote generation. *Hypothesis: NET7 quote-creation may be a back-office workflow; web-API exposure pending (`erp-discovery.md` §3.e).*
- **Used in**: `bounded-contexts.md` §4.e; `capability-map.md` CAP-016, CAP-017, CAP-018.

### REACH

- **DE**: (international)
- **Definition**: EU Regulation on Registration, Evaluation, Authorisation and Restriction of Chemicals (1907/2006). Drives SDS content and chemical-substance traceability for AOT cosmetics ingredients.
- **Used in**: `bounded-contexts.md` §4.c; `erp-discovery.md` §5.a.

### Repository

- **DE**: [no DE equivalent]
- **Definition**: A DDD pattern: an abstraction that mediates between the domain and data-mapping layers, presenting a collection-like interface over persistent aggregates. Naming convention `<Domain>Repository` (`CLAUDE.md` §5).
- **Used in**: `CLAUDE.md` §5; `AGENTS.md` Architecture Rules.

### RFQ

- **DE**: Angebotsanfrage
- **Definition**: Request for Quote. A formal request from a buyer for AOT to issue a price/availability offer for one or more products. Distinct from Sample (no physical material) and Order (no commitment).
- **Used in**: `bounded-contexts.md` §4.e; `capability-map.md` CAP-015 ff.

### RFQ Workflow

- **DE**: Angebotsanfrage-Prozess
- **Definition**: The end-to-end flow from RFQ submission through quote receipt, comparison, and either acceptance (→ Order) or expiry. Stage 5 of the buyer journey in `procurement-hypotheses.md` §5.
- **Used in**: `procurement-hypotheses.md` §5 Stage 5; `capability-map.md` CC-1.

### Sample

- **DE**: Muster (also: Probe)
- **Definition**: A small physical quantity of a raw material provided to a buyer for technical evaluation prior to commercial procurement. Distinct from Sample Request (the digital ask) and Sample Order (the fulfilment record).
- **Used in**: `bounded-contexts.md` §4.d.

### Sample Order

- **DE**: Muster-Auftrag
- **Definition**: The fulfilment record on the AOT side that ships physical sample material. *Hypothesis: may be modelled as a special order class in NET7; validation pending (`erp-discovery.md` §3.c, §3.e).*
- **Used in**: `bounded-contexts.md` §4.d; `erp-discovery.md` §3.c.

### Sample Request

- **DE**: Muster-Anfrage
- **Definition**: The buyer's request to receive a sample, with project context, application, requested delivery date, and optional comment. Submitted via the web platform; fulfilled by NET7.
- **Used in**: `bounded-contexts.md` §4.d; `capability-map.md` CAP-011, CAP-012, CAP-013.

### SDS (Safety Data Sheet)

- **DE**: (international)
- **Definition**: Regulatory document declaring hazard, handling, and safety information for a chemical substance. Required for AOT cosmetics ingredients per REACH and `EU-Kosmetik-VO 1223/2009`.
- **Used in**: `bounded-contexts.md` §4.c; `capability-map.md` CAP-009.

### Specification

- **DE**: Spezifikation
- **Definition**: The set of agreed technical attributes a product or batch must meet — purity, particle size, active content. Often issued by AOT and accepted by the buyer; surfaces in product detail and COA validation.
- **Used in**: `bounded-contexts.md` §4.a; `business-discovery.md` §3.c.

### Strategic Buyer

- **DE**: strategischer Einkäufer
- **Definition**: The primary user persona — buyers responsible for long-term sourcing strategy, supplier relationships, and procurement risk management (per `AGENTS.md` User Personas).
- **Used in**: `AGENTS.md` User Personas; `business-discovery.md` §3.

### Tactical Buyer

- **DE**: taktischer Einkäufer
- **Definition**: A buyer profile focused on day-to-day operational procurement (purchase orders, delivery follow-up). Listed alongside Strategic and Mixed in the discovery interview profile.
- **Used in**: `business-discovery.md` §2.

### TDS (Technical Data Sheet)

- **DE**: (international)
- **Definition**: Document detailing technical and functional properties of a product — typical use rates, compatibility, processing recommendations. Read primarily by Product Developers.
- **Used in**: `bounded-contexts.md` §4.c; `business-discovery.md` §4.c.

### TopM

- **DE**: (international)
- **Definition**: The ERP vendor whose NET7 product family AOT uses. The Sprint-0 ERP-discovery stream targets TopM Technical and Account/Commercial personas.
- **Used in**: `erp-discovery.md` throughout.

### Variant

- **DE**: Variante
- **Definition**: A specific configuration of a base product — packaging size, granulation, certification level. *Hypothesis: NET7 may model variants as first-class entities or as Product attributes; open question (`erp-discovery.md` §3.c).*
- **Used in**: `bounded-contexts.md` §4.a; `capability-map.md` CAP-001.

### Workspace

- **DE**: Projekt-Liste
- **Definition**: A web-only, customer-scoped, project-named bucket of saved products (e.g., "Anti Aging Serum", "Functional Beverage"). Not represented in NET7 — pure web concept owned by the platform.
- **Used in**: `bounded-contexts.md` §4.h; `capability-map.md` CAP-025, CAP-026.

## 4. Categories Index

Same terms grouped by category for targeted lookup. Term names only; see §3 for definitions.

**Domain (Procurement & Products)** — 22 terms
- Allergen Statement, Application, Batch, Botanical Name, CAS, Certificate, COA (Certificate of Analysis), INCI, Kosher / Halal, Lot, MOQ, Order, Order Line, Origin, Quote, RFQ, Sample, Sample Order, Sample Request, SDS (Safety Data Sheet), Specification, TDS (Technical Data Sheet), Variant.

**Architecture (DDD & Systems)** — 10 terms
- ACL (Anti-Corruption Layer), Adapter, Aggregate, Bounded Context, Conformist, Customer/Supplier, Domain Event, Open Host Service, Partnership, Repository.

**Compliance & Regulatory** — 6 terms
- Bio-Zertifikat, EU-Bio-VO 2018/848, EU-Kosmetik-VO 1223/2009, FDA, LFGB, REACH.

**AOT-Specific** — 6 terms
- ApplicationProject, Mixed Buyer, Organic, Strategic Buyer, Tactical Buyer, Workspace.

**ERP (TopM NET7)** — 5 terms
- Customer-ID, NET7, Net7-Adapter, Pricing Tier, TopM.

**Procurement Process** — 5 terms
- Buyer Journey, Decision Criteria, Lead Time, Procurement KPI, RFQ Workflow.

Total: 22 + 10 + 6 + 6 + 5 + 5 = **54 terms** + Quote = **55**.

## 5. References

- [`bounded-contexts.md`](./bounded-contexts.md) — eight bounded contexts as the structural backbone.
- [`capability-map.md`](./capability-map.md) — capability inventory and adapter naming.
- [`AGENTS.md`](../../AGENTS.md) — engineering charter; ERP Ownership Rules and ERP Integration Rules.
- [`procurement-hypotheses.md`](../procurement-hypotheses.md) — UX, ERP, and V1-priority hypotheses.
- [`business-discovery.md`](../business-discovery.md) — German-language buyer / product-developer / quality-manager interview guide.
- [`erp-discovery.md`](../erp-discovery.md) — German-language TopM technical / account-commercial interview guide.
- [`ROADMAP.md`](../../ROADMAP.md) — sprint plan; Sprint 1 Domain Modeling, Sprint 2 ERP Boundary Design.
