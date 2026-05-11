# Pricing-Contracts Aggregate — Sprint-2 Extension

**Baseline:** /docs/domain/aggregates/pricing-contracts.md (Sprint-1 sketch 1.4.5)
**Style:** Sprint-1 ADR-0002 Pattern 1 (7-H2 Template), Pattern 5 (Self-Report Discipline)
**Extension scope:** Public-list-price anchor, price-visibility tiers, fallback policy alignment
**Date:** 2026-05-11

---

## 1. Purpose

Document Pricing-Contracts extensions required by the public-catalog mandate, ADR-0006 (Modular Monolith) pricing-module isolation, and the new authenticated-vs-anonymous price-disclosure tiering.

This document **extends** the Sprint-1 baseline (1.4.5) without modifying it. Sprint-1 modelled per-customer contract pricing exhaustively; Sprint-2 layers public-tier pricing on top.

## 2. Methodology — Pattern 5 Self-Report Discipline

### 2.0 Reused from Sprint-1 baseline (1.4.5)

- CustomerContract as Aggregate Root
- PriceClass value object (Sprint-1 verified: 5 columns × 7 staffeln per Screenshot 6)
- PriceClassAssignment per CustomerAccount (Screenshot 7 verified: Christine = "4-Einstandspreis")
- Sprint-1 INV-001 through INV-013 (state-machine, behavioural, security-boundary)
- ADR-0003 Rule 5 (Read-Replica-Fallback-Discipline) for NET7 pricing-API unavailability
- Pattern 5 Self-Report Discipline (continued)
- 13/0/0 invariant distribution noted in Sprint-1

### 2.1 New components

| Component | Origin | Notes |
|-----------|--------|-------|
| `PublicListPrice` | OQ-053; visual discovery Screenshot 6 | Public-anchor price computed from Staffel-1 row of a designated PriceClass column. // ASSUMPTION: default column is "VK frei Haus" — overridable per OQ-053 resolution. |
| `PriceVisibilityRule` | OQ-054 | Per-tier rule controlling what each user-type sees. See §2.2 below. |
| `PriceContext` (response field) | ADR-0006 module boundary | Pricing-module response includes `priceContext: "public" | "customer" | "rfq-required"` so frontend rendering is unambiguous. |
| `PriceFallbackBehaviour` | OQ-055; Sprint-1 ADR-0003 Rule 5 | Fallback when NET7 unavailable — concrete behaviour list per user-type (see §2.3) |
| `RFQTrigger` | Visual discovery Screenshot 6 ("A.Anfrage" at 190+ kg) | Pricing returns RFQTrigger for tiers marked "A.Anfrage"; downstream RFQ module captures the request. |

### 2.2 PriceVisibilityRule — tier matrix

| User Type | Visible Price | "A.Anfrage" Tier Behaviour | Source |
|-----------|---------------|---------------------------|--------|
| Anonymous | PublicListPrice (Staffel-1 of designated column) | Show "Auf Anfrage — Login für Angebot" + Login-CTA | OQ-053, OQ-054 |
| Authenticated B2B (customer-price-class assigned) | Per-customer price (full staffel access) | RFQ-CTA (anonymous Login-CTA replaced by RFQ-form entry) | Sprint-1 INV per Pricing baseline |
| Authenticated B2B (no price-class — new customer awaiting NET7 onboarding) | PublicListPrice + banner indicating pending pricing-setup | RFQ-CTA | // ASSUMPTION: edge-case in customer-onboarding; resolved during Customer-Account discovery |

// ASSUMPTION: only one PublicListPrice per product surface in V1 (single anchor). V2 could expose "from €X for Staffel-1, up to €Y for Staffel-5" range display if SEO/UX testing indicates value (OQ-053 follow-up).

### 2.3 PriceFallbackBehaviour — when NET7 unavailable

Per Sprint-1 ADR-0003 Rule 5, fallback must be explicit, not implicit. Behaviour per user-type:

| User Type | Fallback | UX Affordance |
|-----------|----------|---------------|
| Anonymous | Cached PublicListPrice (last-known projection from Catalog read-model) — projected daily, so worst-case 24-hour staleness | No visible degradation (cached price is the normal-case data path for anonymous) |
| Authenticated B2B | Stale customer-price (from cache, if cached within TTL window) OR PublicListPrice fallback OR error-state — choice per OQ-055 resolution | Banner: "Pricing aktualisiert sich gerade — angezeigt: zuletzt bekannter Preis" or equivalent |

// ASSUMPTION: customer-pricing cache TTL is short (5-60 min). Exact value derived from NET7 SLA discovery — see Sprint-1 OQ-coupling for NET7 performance OQs.

### 2.4 Cacheability rules (constitutional, derived from ADR-0006 module boundary)

- **PublicListPrice IS edge-cacheable** (same value for all anonymous users); daily refresh on `ListPriceUpdated` projection
- **Per-customer price is NEVER edge-cacheable** (cross-user leak risk); short-TTL in-memory cache acceptable per customer-id key

This rule is constitutional. Violations risk cross-customer price-leak and DSGVO/competitive-data-exposure.

### 2.5 Components NOT relevant (out of scope)

- Payment-driven pricing (per ADR-0008 deferred) — no payment-tier pricing in V1
- Multi-currency conversion — V1 EUR only (per strategic-foundation §9.1)
- Subscription / volume-commitment discount — V2 scope
- Dynamic pricing (demand-based) — out of scope entirely

## 3. Aggregate Boundary

Sprint-1 baseline aggregate boundary unchanged. PublicListPrice is a **projection** computed from existing CustomerContract data (specifically the un-customer-assigned base PriceClass rows). It is not a new entity within Pricing.

PriceVisibilityRule is module-level policy logic, executed by the `pricing` module (ADR-0006). It is not stored on the aggregate.

## 4. Internal Structure

### 4.1 Projections from CustomerContract data

The Pricing module exposes two projection types from the same underlying CustomerContract aggregate:

- **PublicListPriceProjection**: per-product, computed from Staffel-1 VK frei Haus of the "default public" PriceClass. Cached at edge.
- **CustomerPriceProjection**: per-(customer, product, quantity) tuple, computed real-time via NET7 adapter or short-TTL cache. NOT edge-cacheable.

Both projections share the same Sprint-1 invariants on the underlying CustomerContract structure.

### 4.2 Invariants (continuing INV-numbering from Sprint-1 baseline)

- **INV-014** (security-boundary, new): per-customer price MUST NOT appear in any edge-cached, CDN-cached, or anonymous-accessible response. Pricing module response containing `priceContext: "customer"` MUST set Cache-Control: private + no-store headers.
- **INV-015** (state-machine, new): when NET7 pricing-API unavailable, fallback transition follows the rule in §2.3 — anonymous → cached PublicListPrice (no visible degradation), authenticated → policy-driven (banner mandatory if degraded)
- **INV-016** (cross-aggregate, new): RFQTrigger raised by Pricing module MUST be consumable by RFQ module without further aggregate-internal data exposure — only ProductId, RequestedQuantity, CustomerAccountReference (if authenticated) cross the boundary
- **INV-017** (behavioural, new): PublicListPrice projection updates on `ListPriceUpdated` NET7 event MUST trigger Catalog read-model refresh (cross-module event coupling) — eventual consistency target <60s

## 5. Web/ERP Ownership Mapping

| Field | Owned by NET7 | Owned by Platform | Notes |
|-------|---------------|-------------------|-------|
| All Staffel × PriceClass values | ✓ | — | NET7 master |
| PriceClass-to-Customer assignment | ✓ | — | NET7 master (Screenshot 7) |
| MwSt / Brutto computation | ✓ | — | NET7 rule (Brutto = Netto × 1.07 per Screenshot 6) |
| PublicListPrice (projection) | — | ✓ | Platform computes from NET7 data per OQ-053 rule |
| PriceVisibilityRule policy | — | ✓ | Platform-side rule, not in NET7 |
| Cache invalidation on update | — | ✓ | Platform subscribes to NET7 events |
| RFQTrigger emission | — | ✓ | Platform-side, fed by NET7 "A.Anfrage" marker |

## 6. Cross-Aggregate References and OQs

### Cross-aggregate references

- → Catalog aggregate: `ListPriceAnchor` (catalog-sprint-2-extension §2.1) is the Catalog-side projection of PublicListPrice
- → Customer-Account aggregate: customer-pricing requires customer-account-id and is gated by authentication context (see customer-account-sprint-2-extension)
- → RFQ aggregate (Sprint-2 V1 scope): RFQTrigger initiates RFQ workflow

### Open Questions (from oq-master-sprint-2-extension)

- OQ-053 (public-list-price anchor strategy)
- OQ-054 (anonymous "A.Anfrage" UX)
- OQ-055 (NET7-unavailable fallback policy)

## 7. References

- /docs/domain/aggregates/pricing-contracts.md (Sprint-1 baseline 1.4.5, 13/0/0 INV-distribution)
- /docs/adr/0002-domain-patterns.md (Pattern 1, Pattern 5 Self-Report Discipline)
- /docs/adr/0003-erp-boundary-rules.md (Rule 4 Authentication-Credential-Containment, Rule 5 Fallback-Discipline — both directly applied here)
- /docs/adr/0006-modular-monolith.md (`pricing` module boundary; cross-module API contract)
- /docs/adr/0008-payment-strategy.md (no payment-tier pricing in V1)
- /docs/domain/aggregates/catalog.md (Catalog ListPriceAnchor consumer; see catalog-sprint-2-extension)
- /docs/domain/aggregates/customer-account.md (customer-pricing gated by authentication; see customer-account-sprint-2-extension)
- /docs/domain/open-questions-master.md (OQ-053, OQ-054, OQ-055 — see oq-master-sprint-2-extension)
- Visual discovery: Screenshots 6 (Staffelpreise-Tabelle), 7 (PriceClass-assignment dropdown), 8 (Christine's view at Einstandspreis), 4 (no payment-tier)
