# Catalog Aggregate — Sprint-2 Extension

**Baseline:** /docs/domain/aggregates/catalog.md (Sprint-1 sketch 1.4.1)
**Style:** Sprint-1 ADR-0002 Pattern 1 (7-H2 Template), Pattern 5 (Self-Report Discipline)
**Extension scope:** New components and INVs required by Sprint-2 mandate (public catalog visibility, list-price-anchor, sub-title pattern, sample-as-quantity refactor)
**Date:** 2026-05-11

---

## 1. Purpose

Document Catalog aggregate extensions required by the public-catalog mandate (Discovery NEW FACTS) and Sprint-2 ADRs (0006 Strangler-Fig, 0007 Modular Monolith, 0008 Content Strategy).

This document **extends** the Sprint-1 baseline. It does not replace it. On commit, the extensions merge into the master sketch (single-file-discipline preserved).

## 2. Methodology — Pattern 5 Self-Report Discipline

Sprint-1 ADR-0002 Pattern 5 requires explicit reporting of what's reused, what's new, and what's not relevant.

### 2.0 Reused from Sprint-1 baseline (1.4.1)

- Product as Aggregate Root
- Variant as internal child entity
- ProductIdentifier value object (Teilenummer, INCI, CAS)
- Sprint-1 INV-001 through INV-006 (structural and behavioural)
- NET7-as-Single-Source-of-Truth alignment (ADR-0003 Rule 1)
- Read-only-mirror notation (Sprint-1 Pattern 3)
- Adapter-driven projection from NET7 master

### 2.1 New components (added by this extension)

| Component | Origin | Notes |
|-----------|--------|-------|
| `PublicVisibilityFlag` | Visual discovery Screenshot 1 (NET7 WebService-flag "W"); OQ-049 | Maps NET7 WebService-flag to platform public-visibility decision. Default-fail-safe: not-public unless flag explicitly set. |
| `ABCClassification` | Visual discovery Screenshot 6 (ABC="A") | Catalog-level classification surfaced from NET7. V2 candidate for search-ranking boost; V1 stored but not surfaced in UI. |
| `EAN13` | Visual discovery Screenshot 6 (EAN-13 field) | Additional identifier alongside Teilenummer, INCI, CAS. Used for Schema.org `gtin13` property in product structured data. |
| `WarengruppeReference` | Visual discovery Screenshot 6 (WG=001-Öl) and Screenshot 9 (URL parameter `Warengruppen=WG=001`) | Catalog hierarchy. Drives sitemap-by-Warengruppe (OQ-050) and faceted search. |
| `ListPriceAnchor` | Visual discovery Screenshot 6 (Staffel-1 VK frei Haus 39,90); OQ-053 | Public-exposable price field. Read-only projection from Pricing aggregate; concrete strategy in pricing-sprint-2-extension §2.1. |
| `SubTitle` | Visual discovery Screenshot 9 ("für kosmetische Zwecke", "GLA 20 %") | Disambiguation marker for variants of the same raw material. Free-text from NET7 Sachmerkmal field or similar. |
| `SanitizedNarrativeHtml` | ADR-0007 §2.3 | NET7 `Webshop Beschreibung` HTML with inline `style="..."` attributes stripped. Result fed to frontend; frontend applies design-system styling. |
| `LocaleVariant` | OQ-047 | Per-locale projection of name, sub-title, narrative-html. V1: DE minimum; EN if OQ-047 resolves day-1. |

### 2.2 Sample-as-Quantity refactor (CRITICAL)

Sprint-1 baseline modeled `Sample` as a separate Aggregate-Sub-Type. Visual discovery Screenshot 8 refutes this: the "Muster" button is the **8th quantity-option** alongside the 7 commercial staffeln (1, 4.5, 9, 23, 115, 190, 760 kg + Muster).

**Decision:** Sample is a quantity-option on the Catalog Product, NOT a separate Catalog-Subtype. The Sample Aggregate (Sprint-1 1.4.2) handles the order-workflow lifecycle, but the catalog-level representation is one row: Product with a `SampleQuantityOption` flag.

This aligns the catalog-side data model with the user-visible reality (one product card, multiple quantity buttons).

// ASSUMPTION: every product visible publicly offers the "Muster" option. If not — some products are sample-disqualified — a `SampleEligibility` flag is required. Discovery question (extension of OQ-052 scope).

### 2.3 Components NOT relevant (explicitly out of scope)

- Payment fields — per ADR-0008, no payment in Catalog (sample-orders go via Sample Aggregate workflow without payment in V1)
- Customer-specific pricing in Catalog — per ADR-0006 module boundary, customer-specific pricing lives in `pricing` module, surfaced via cross-module call; Catalog only carries ListPriceAnchor for public display
- Multi-tenant account-hierarchies — Sprint-1 baseline rightly defers this; no change here
- Recommendation/related-product data — V2 scope, not Sprint-2

## 3. Aggregate Boundary

The Catalog aggregate boundary remains as Sprint-1 defined: Product as Root, Variants as child entities, identifiers and projections as value objects.

The extensions in §2.1 are all **fields on the existing Aggregate Root or value objects**. No new entity, no new aggregate-boundary crossing.

## 4. Internal Structure

### 4.1 Product (Aggregate Root) — extended

Sprint-1 fields (unchanged): ProductId, Teilenummer, INCI, CAS, Bezeichnung, Status, Variants[]

Sprint-2 added fields:
- PublicVisibilityFlag (boolean, default false)
- ABCClassification (enum: A, B, C, undefined)
- EAN13 (string, optional)
- WarengruppeReference (id + display-name)
- ListPriceAnchor (decimal + currency, projection from Pricing)
- SubTitle (string per locale, optional)
- SanitizedNarrativeHtml (string per locale)
- LocaleVariant (collection keyed by locale)
- SampleQuantityOption (boolean, see §2.2)

### 4.2 Invariants (continuing INV-numbering from Sprint-1 baseline)

- **INV-007** (structural, new): if `PublicVisibilityFlag == true`, then `LocaleVariant` for at least one locale must be non-empty (no public product without name/narrative)
- **INV-008** (security-boundary, new): if `PublicVisibilityFlag == false`, the product MUST NOT appear in any public-tier projection (catalog read-model, search index, sitemap). Default-fail-safe required.
- **INV-009** (behavioural, new): SanitizedNarrativeHtml MUST NOT contain inline `style` attributes. Adapter enforces this on projection. If raw NET7 HTML contains styles, sanitization strips them before persistence to read-model.
- **INV-010** (cross-aggregate, new): `ListPriceAnchor` value is sourced from Pricing aggregate (computed via Pricing module's public-list-price rule, see pricing-sprint-2-extension §2.1). Catalog does not store the price-formula; it stores the resolved anchor at projection time.
- **INV-011** (state-machine, new): `PublicVisibilityFlag` transition rules — flag flip from false to true requires all visibility-prerequisites met (LocaleVariant present, ListPriceAnchor available). Flag flip from true to false is unconditional (allow rapid public-takedown).

## 5. Web/ERP Ownership Mapping

| Field | Owned by NET7 | Owned by Platform | Notes |
|-------|---------------|-------------------|-------|
| Teilenummer, INCI, CAS, EAN13 | ✓ | — | Identifiers, no change |
| ABCClassification | ✓ | — | Read-only mirror |
| WarengruppeReference | ✓ | — | Read-only mirror |
| Bezeichnung, SubTitle | ✓ | — | Read-only mirror per locale |
| Webshop Beschreibung HTML (raw) | ✓ | — | NET7 owns raw content |
| SanitizedNarrativeHtml | — | ✓ | Platform computes via adapter sanitization |
| PublicVisibilityFlag (decision) | Ambiguous | — | Sourced from NET7 WebService-flag, but platform applies fail-safe defaults — see OQ-049 |
| ListPriceAnchor (resolved) | — | ✓ | Computed by Pricing module, projected into Catalog |
| LocaleVariant projection | — | ✓ | Platform aggregates per-locale view from NET7 multi-language fields |

## 6. Cross-Aggregate References and OQs

### Cross-aggregate references (new in this extension)

- → Pricing aggregate (via Pricing module): `ListPriceAnchor` resolved
- → Sample aggregate (via Sample module): `SampleQuantityOption` flag enables Sample-Order workflow entry
- → Documents-Compliance aggregate (via Documents module): per-product scoped document list, scoping per documents-sprint-2-extension §2.1
- → Search module: search-index projection sourced from Catalog read-model on `ProductPublished` event

### Open Questions (from oq-master-sprint-2-extension)

- OQ-047 (i18n scope) — affects LocaleVariant projection
- OQ-049 (public visibility filter) — affects PublicVisibilityFlag derivation
- OQ-050 (sitemap policy) — affects WarengruppeReference usage
- OQ-052 (anonymous inventory visibility) — affects Catalog projection field-visibility
- OQ-053 (list-price anchor strategy) — affects ListPriceAnchor computation rule
- OQ-060 (brand-redesign migration trigger) — affects long-term lifespan of SanitizedNarrativeHtml approach

## 7. References

- /docs/domain/aggregates/catalog.md (Sprint-1 baseline 1.4.1)
- /docs/adr/0002-domain-patterns.md (Pattern 1 7-H2 template, Pattern 5 Self-Report Discipline)
- /docs/adr/0003-erp-boundary-rules.md (Rule 1 NET7-as-SoT, Rule 3 Idempotency for adapter projections)
- /docs/adr/0006-modular-monolith.md (catalog module boundary)
- /docs/adr/0007-content-strategy.md (hybrid content model, sanitization in V1)
- /docs/adr/0008-payment-strategy.md (no payment in catalog, Sample-as-Quantity refactor implication)
- /docs/domain/adapter-topology.md (Sprint-1 P4 — adapter profile for catalog projection)
- /docs/domain/aggregates/pricing-contracts.md (ListPriceAnchor source; see pricing-sprint-2-extension)
- /docs/domain/aggregates/sample.md (Sample-Quantity-Option workflow; see sample-sprint-2-extension)
- /docs/domain/aggregates/documents-compliance.md (document scoping; see documents-sprint-2-extension)
- /docs/domain/open-questions-master.md (OQ-047, OQ-049, OQ-050, OQ-052, OQ-053, OQ-060 — see oq-master-sprint-2-extension)
- Visual discovery: Screenshots 1, 3, 6, 7, 8, 9
