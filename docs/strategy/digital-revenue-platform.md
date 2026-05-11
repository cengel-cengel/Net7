# AOT Digital Revenue Platform — Strategy

**Status:** Constitutional — supersedes any prior framing that conflicts with the five principles below.
**Date:** 2026-05-11
**Decided by:** Human (System Architect, on behalf of AOT)
**Authors:** Human + Claude (chat) + Claude Code
**Hierarchy:** This document sits ABOVE the ADRs. ADRs operationalize these principles; this document frames the business intent that ADRs must serve.

---

## §1 Purpose

This document codifies AOT's strategic intent for the new digital platform. Sprint-1 (domain modelling) and Sprint-2 Foundation (architectural decisions) produced significant deliberation material — strategic-foundation.md, 8 ADRs, 5 aggregate sketches + 5 extensions, 61 OQs, MDR-risk-register. This document is the **constitutional layer** that frames that work: WHY the platform exists and WHAT outcome it produces.

If any prior artifact appears to conflict with the principles below, this document wins and the conflicting artifact must be reconciled.

---

## §2 The Core Business Decision

**AOT builds ONE digital revenue platform.**

**Goal: demand generation + procurement enablement.**

The platform is not a "website modernization." It is not a "B2B portal upgrade." It is not a "SEO project." It is a single commercial system with two pillars: it generates new commercial demand (anonymous discovery → conversion) and it enables existing customer procurement workflows (authenticated B2B operations).

These two pillars share one platform, one brand, one data foundation, one engineering codebase. The customer experience does not surface this duality.

---

## §3 The Five Architecture Principles

### Principle 1 — Customer never sees website vs portal separation

The end-user — anonymous or authenticated, lead or established customer — experiences ONE digital surface. Navigation, design, terminology, brand are consistent. Authentication is a transition WITHIN the experience, not a transition BETWEEN systems.

**Implication:** No "website.aot.de" vs "portal.aot.de" split. No jarring UI swap on login. One design system, two rendering modes (per ADR-0005 Strangler-Fig migration target).

**Implication for migration:** Strangler-Fig Pattern (ADR-0005) builds the new platform alongside legacy. When legacy is sunset, the unified experience is what remains.

### Principle 2 — NET7 stays system-of-record only

NET7 is authoritative for: product master, prices, inventory, DMS, certifications, customer accounts, order history, batch traceability, ERP workflows.

NET7 is NOT authoritative for: experience, SEO, content, sessions, search, marketing, analytics, conversion-tracking, public-tier presentation.

**Implication:** No experience-layer logic leaks into NET7. No frontend developers write NET7 logic. No NET7 developers reach into web code. Adapter discipline (Sprint-1 ADR-0003) is constitutional.

**Implication for V1:** Web platform consumes NET7 via adapters only. Outbound writes (sample-requests, customer profile updates) are queued, idempotent, isolated.

### Principle 3 — Public discovery layer drives SEO and demand

The public-tier exists for one purpose: **commercial demand generation**. It captures anonymous traffic, converts it through SEO-driven discovery, provides enough context (product information, documents, sample-request) to qualify the lead, and hands it to the procurement layer (or to AOT-Sales for high-touch follow-up).

**Implication:** SEO performance is a primary KPI for the public-tier. Page-load, structured-data, content-quality, link-architecture are commercial concerns, not technical-niceties.

**Implication:** Conversion-tracking is mandatory. Anonymous → sample-request → CustomerAccount-creation conversions must be measurable. ADR-0008 (Payment Deferred) validation metrics depend on this instrumentation.

### Principle 4 — Authenticated layer handles procurement logic

Logged-in B2B-customers see a procurement-grade experience: customer-specific pricing (per NET7 contract assignment), order history, document access (B2B-scoped + customer-specific), RFQ submission, account self-service.

**Implication:** The authenticated layer is a serious commercial tool, not a "members area." Multi-user accounts (V1 basic, V2 hierarchies), approval workflows (V2), procurement-API integration (V2 cXML/OCI) are roadmap items, not afterthoughts.

**Implication:** Per ADR-0007 (Pricing-Disclosure constitutional in pricing-sprint-2-extension §2.4), customer-pricing is NEVER edge-cached. Authentication is a real boundary, not a cosmetic one.

### Principle 5 — V1 launches with curated hero portfolio

V1 does NOT publish all ~800 AOT raw materials publicly. V1 launches with a **curated hero portfolio** — a strategic subset of products that:
- Represent AOT's commercial strengths (volume, margin, USP)
- Have content-grade material ready (high-quality images, narrative, application guides, complete documentation)
- Anchor SEO targeting for AOT's positioning (Bio cosmetic oils, food-grade specialty oils, etc.)
- Justify marketing investment per product

The remaining catalog continues to live in NET7, accessible via the authenticated B2B-tier (Principle 4) for established customers. Public discoverability for the long-tail is a V2 decision.

**Implication:** OQ-049 (which products public-visible) is reframed: not "all-vs-WebService-flag" but "which products are Hero." Hero-portfolio membership is a deliberate AOT-Sales + Marketing curation, not an automatic flag-derived projection.

**Implication:** Content-investment per Hero product is significant (high-quality copy, multi-language narrative, application stories, formulation guides). Per ADR-0007 (Hybrid Content Strategy), V1 still uses NET7 Webshop-Beschreibung sanitized — but Hero products may receive earlier CMS-migration investment.

**Implication for V1 success criteria:** Hero-portfolio conversion (anonymous → sample-request, anonymous → RFQ, anonymous → account-creation) — not all-catalog SEO ranking.

---

## §4 The Two Pillars

### Pillar A — Demand Generation

Goal: Convert anonymous web visitors into qualified commercial leads.

Key components:
- Public-tier catalog for the Hero Portfolio (Principle 5)
- SEO-optimized rendering, Schema.org structured data, sitemap, content velocity (V2 CMS for editorial layer per ADR-0007)
- Sample-request workflow (per ADR-0008, no payment in V1 — Bestellanfrage workflow)
- RFQ submission for commercial-volume tiers ("A.Anfrage" Staffeln in NET7 per Visual Discovery Screenshot 6)
- Lead-capture instrumentation (anonymous → contact details → CustomerAccount conversion path per customer-account-sprint-2-extension §3.3)

V1 success metrics:
- Hero-portfolio SEO performance (impressions, clicks, ranking)
- Sample-request volume (anonymous + authenticated)
- Sample-request → CustomerAccount-creation conversion rate
- RFQ-submission volume

### Pillar B — Procurement Enablement

Goal: Serve existing AOT-customers with a procurement-grade digital tool.

Key components:
- Authenticated B2B-tier with customer-specific pricing (per Sprint-1 Pricing aggregate + pricing-sprint-2-extension)
- Document portal (B2B-scoped + customer-specific per documents-sprint-2-extension §2.2)
- RFQ workflows (lightweight V1, full negotiation V2)
- Customer self-service (account, addresses V1; multi-user/approval V2)
- Order history (read-only from NET7)
- Future: procurement-API for customer-ERP integration (V2 cXML/OCI)

V1 success metrics:
- Migration from legacy portal to new B2B-tier (% of customer-base on new platform)
- Customer self-service activity (vs. AOT-Sales-mediated transactions)
- Customer satisfaction with new portal vs. legacy

---

## §5 The Curated Hero Portfolio — V1 Scope

### What "Hero" means

A Hero product in V1 meets all of:
1. **Commercially significant** — material to AOT revenue (top-50 by volume or top-25 by margin contribution — // ASSUMPTION: thresholds to be set by AOT-Sales)
2. **Content-ready** — has high-quality images, complete narrative, current SDS/COA/TDS, application-context, certification claims
3. **Strategic positioning** — anchors a key AOT positioning (e.g., Bio cosmetic oils, food-grade specialty oils, INCI-listed actives, Upcycling-Beauty-narrative ingredients per Visual Discovery Screenshot 5)
4. **Curation-approved** — explicitly added to the Hero list by AOT-Sales + Marketing

### What "Hero" does NOT mean

- Hero status is NOT "all aktiv products" or "all products with NET7 WebService-flag"
- Hero status is NOT automatic from any NET7-side classification — it is a Web-platform-side decision recorded in the platform
- Hero status is NOT permanent — products enter and leave the Hero list as commercial strategy evolves

### Selection mechanism — V1

// ASSUMPTION: V1 implementation tracks Hero-status as a platform-side flag (not in NET7). AOT-Sales + Marketing maintain the list via a simple admin tool (could be as basic as a configured list in code or a database table managed via API).

// ASSUMPTION: ~30-80 Hero products at V1 launch — sufficient for SEO breadth without exceeding content-investment capacity. Exact count is an AOT business decision.

### Catalog-Visibility-Filter — V1

The public catalog renders ONLY Hero products. The B2B-authenticated catalog renders ALL aktiv products (existing operational behaviour, preserved).

This sharpens catalog-sprint-2-extension §2.1 `PublicVisibilityFlag`:
- V1: `PublicVisibilityFlag = isHeroPortfolioMember(productId)` — a platform-side decision, not a NET7-flag projection
- V2 (post-launch evaluation): broaden criteria if content-pipeline scales

### Long-tail catalog — V1 path

For non-Hero products:
- Remain in NET7 unchanged
- Accessible via authenticated B2B-tier (existing portal behaviour migrated to new platform per ADR-0005 Phase 3)
- NOT indexed by search engines (not in sitemap, not in public-tier routing)
- Future re-evaluation in V2 based on V1 learnings

---

## §6 Reconciliation with Sprint-2 Foundation

### What this update CONFIRMS

- ADR-0005 Strangler-Fig: NET7 stays backend-of-record, new experience layer external — Principle 2 reinforces
- ADR-0006 Modular Monolith: 8-module domain structure remains correct — Principle 4 procurement-logic aligns with `customer`/`pricing`/`rfq` modules
- ADR-0007 Content Strategy (Hybrid): V1 sanitization of NET7 HTML correct — Principle 5 Hero-portfolio adds: Hero products may receive CMS-migration investment earlier than long-tail
- ADR-0008 Payment Deferred: aligns — Principle 3 demand-generation gets validation data before payment infrastructure
- Sprint-1 Pricing Aggregate: customer-specific pricing in NET7 is authoritative — Principle 2 confirms
- documents-sprint-2-extension Default-Fail-Safe (B2B-only): Principle 1 unified experience requires conservative public exposure — confirmed

### What this update REFINES

- **OQ-049** is reframed: from "public-visibility-filter" to "Hero-portfolio-membership". Question to AOT-Sales becomes: "Which 30-80 products are V1 Hero candidates?"
- **catalog-sprint-2-extension §2.1 PublicVisibilityFlag** semantic update: V1 means "Hero membership", not "NET7 WebService-flag projection"
- **strategic-foundation.md §9.1 V1 Scope** "Public catalog with SEO" gets sharpened: "Public Hero-portfolio catalog with SEO"
- **Discovery Phase-0 Brief §5 OQ-049 question battery** can be tightened: Hero-curation-process is now the question, not visibility-rule

### What this update INTRODUCES

- **Hero Portfolio** as a V1 concept that did not exist in prior Sprint-2 Foundation
- **Demand Generation** as an explicit business KPI pillar — previously implicit
- **Procurement Enablement** as the second pillar, explicitly co-equal with demand-generation
- **Hero-curation ownership** as a new business-process (see §8 below)
- **Constitutional hierarchy** with this strategy document above ADRs

---

## §7 Impact on Open Questions

| OQ | Pre-Strategy Status | Post-Strategy Status |
|----|---------------------|---------------------|
| OQ-047 (SEO target markets DE-only or DE+EN V1) | Open | Open — but reframed: hero-portfolio content investment determines i18n scope |
| OQ-048 (SEO ownership) | Open | Open — but elevated priority (demand-generation is now a primary pillar) |
| OQ-049 (Public visibility filter) | H-Risk Open | **Reframed:** Hero-portfolio curation, not flag-projection. New question: who curates the Hero list, what criteria? |
| OQ-050 (Sitemap policy) | Open | Simplified: sitemap = Hero-portfolio products only V1 |
| OQ-051 (DocumentScope mapping) | H-Risk Open | Open — unchanged |
| OQ-052 (Anonymous inventory visibility) | Open | Open — but only relevant for Hero-portfolio products V1 |
| OQ-053 (Public list-price anchor) | H-Risk Open | Open — unchanged scope, decision applies to Hero-portfolio only V1 |
| OQ-054 (A.Anfrage UX) | Open | Open — unchanged |
| OQ-055 (NET7-unavailable pricing fallback) | Open | Open — unchanged |
| OQ-056-058 (Payment validation, fraud) | Open | Open — unchanged |
| OQ-059-061 (CMS, translation) | Open | Open — but Hero-portfolio narrows V2 CMS scope |
| **NEW OQ-062** | — | Who owns Hero-portfolio curation (AOT-Sales? Marketing? joint?), what is the add/remove workflow, what is the V1 launch list? |

OQ-062 is now H-Risk Phase-0 blocking, replacing OQ-049 in that role.

---

## §8 New Ownership Territory

The Hero-portfolio introduces business-process ownership that did not exist before:

| Question | Owner | Cadence |
|----------|-------|---------|
| Who curates the Hero list? | AOT-Sales + Marketing (joint, // ASSUMPTION) | Quarterly review V1 |
| Who approves add/remove? | // ASSUMPTION: Sales-Lead has final approval | Per-request |
| Who owns Hero content quality? | Marketing (with QM regulatory review) | Per-product onboarding |
| Who tracks Hero conversion performance? | // ASSUMPTION: Sales + Engineering joint (Sales sees commercial impact, Engineering provides instrumentation) | Monthly |
| Who triggers V2 broadening (more products public)? | AOT-Mgmt based on V1 learnings | Post-V1 review |

These assumptions are starting points — Discovery confirms or refines them.

---

## §9 V1 Scope Implications

### What V1 INCLUDES (sharpened from strategic-foundation.md §9.1)

- Public-tier Catalog **for Hero portfolio only** (NOT all 800 products)
- SEO content investment for ~30-80 Hero products
- Sample-request workflow (per ADR-0008, no payment)
- RFQ submission (per ADR-0006 `rfq` module)
- B2B-authenticated catalog **for all aktiv products** (existing operational behaviour migrated)
- Customer-specific pricing for authenticated users
- Document portal (Public + B2B + customer-specific scopes per documents-sprint-2-extension)
- Customer self-service (basic V1 per customer-account-sprint-2-extension)
- Conversion-tracking instrumentation (anonymous → sample → CustomerAccount)

### What V1 EXCLUDES (post-strategy explicit)

- Public catalog for long-tail (non-Hero) products
- Multi-language full coverage (DE primary; EN for Hero products if OQ-047 says yes)
- CMS for Hero-product narratives (V1.5 candidate; V1 ships sanitized NET7 HTML)
- Multi-tenant account hierarchies (V2)
- SSO (V2)
- Procurement-API for customer-ERP integration (V2 cXML/OCI)
- Approval workflows (V2)
- Subscription/auto-reorder (V2)
- Recommendation engine (V2 — needs Hero conversion data first)

### V1 success criteria (sharpened)

Demand Generation pillar:
- Hero-portfolio indexing rate >80% within 4 weeks of launch
- Sample-request rate from anonymous traffic measurable and trend-able
- Anonymous → CustomerAccount-conversion >1% (// ASSUMPTION threshold, refine with AOT-Sales)
- Lighthouse Performance/SEO scores >90 for Hero-portfolio product pages

Procurement Enablement pillar:
- Top-20 customers migrated to new B2B-tier within Phase 4 window
- Customer-specific pricing accuracy 100% sample-tested
- Daily Active Users on new portal matching legacy baseline
- Customer-pricing-API p95 response time within SLA

---

## §10 What This Does NOT Change

To prevent over-interpretation of this update:

- **Sprint-1 artifacts are unchanged.** This is a strategy refinement, not a Sprint-1 retraction.
- **The 8 modules from ADR-0006** are unchanged. The same Catalog, Pricing, Customer, Documents, Sample, RFQ, Auth, Search modules ship V1.
- **The NET7 adapter discipline** from Sprint-1 ADR-0003 is unchanged.
- **The Strangler-Fig migration order** from ADR-0005 §2.6 is unchanged. Public catalog (Phase 1) renders Hero-portfolio; long-tail B2B-tier (Phase 3) renders all aktiv products.
- **The hybrid content strategy** from ADR-0007 is unchanged. Hero products simply receive earlier content-investment, not different content model.
- **Payment deferral** from ADR-0008 is unchanged.
- **Default-Fail-Safe** in documents from documents-sprint-2-extension §2.3 is unchanged and now even more important (public exposure is smaller, but each Hero product's documents need correct scoping).

---

## §11 References

- /docs/architecture/strategic-foundation.md (predecessor deliberation; this strategy doc is the constitutional layer above it)
- /docs/adr/0005-strangler-fig.md (Principle 2 alignment)
- /docs/adr/0006-modular-monolith.md (Principle 4 module mapping)
- /docs/adr/0007-content-strategy.md (Principle 5 Hero-content-investment)
- /docs/adr/0008-payment-strategy.md (Principle 3 demand-generation validation enables)
- /docs/domain/aggregates/catalog-sprint-2-extension.md §2.1 (PublicVisibilityFlag semantic update)
- /docs/domain/aggregates/pricing-contracts-sprint-2-extension.md §2.4 (constitutional cacheability — confirmed)
- /docs/domain/aggregates/documents-compliance-sprint-2-extension.md §2.3 (Default-Fail-Safe — confirmed even more important)
- /docs/domain/open-questions-master-sprint-2-extension.md (OQ-049 reframe, OQ-062 NEW)
- /home/claude/discovery/aot-discovery-phase-0-brief.md (Discovery Phase-0 Brief — needs §5 OQ-049 update to reflect Hero-portfolio reframe)
- AGENTS.md (engineering charter — Principle 2 alignment, NET7 isolation)
- Visual Discovery: Screenshots 6 (Staffel pricing), 8 (public product page), 9 (catalog listing pattern), 5 (Upcycling-Beauty narrative — Hero-positioning anchor)

---

## §12 Constitutional Status

This document is the **highest-level frame** for the platform. ADRs operate WITHIN these principles. Aggregate sketches operationalize ADRs. Implementation realizes aggregates.

If a future ADR or implementation choice would violate any of the 5 principles, the principle wins. The ADR must be revised, or this strategy must be explicitly revised by a successor document.

**Revision-policy:** This strategy may only be revised by a successor strategy document, explicitly named, dated, and acknowledging the change. The Five Principles are not modified via ADR.
