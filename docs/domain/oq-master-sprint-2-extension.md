# Open Questions Master — Sprint-2 Extension

**Baseline:** /docs/domain/open-questions-master.md (Sprint-1, OQ-001 through OQ-046)
**Extension scope:** Sprint-2 mandate (Discovery NEW FACTS) introduces new question clusters not addressed by Sprint-1.
**Governance:** /docs/adr/0004-open-question-governance.md (Open Question Governance Rules 1-6, Triplet-Closer pattern)
**Date:** 2026-05-11

This document lists OQ-047 through OQ-061. They extend the Sprint-1 OQ-master without modifying it. On commit they merge into the master file (single file maintained going forward).

---

## Section 1 — New OQ Clusters

Six new clusters emerge from Sprint-2 mandate analysis:

| Cluster | OQ Range | Topic |
|---------|----------|-------|
| C1 | OQ-047, OQ-048 | SEO indexing strategy and ownership |
| C2 | OQ-049, OQ-050 | Public catalog visibility scope |
| C3 | OQ-051, OQ-052 | Anonymous product access (documents + inventory) |
| C4 | OQ-053, OQ-054, OQ-055 | Pricing exposure policy |
| C5 | OQ-056, OQ-057, OQ-058 | Payment readiness validation |
| C6 | OQ-059, OQ-060, OQ-061 | CMS and content ownership |
| C7 | OQ-062 | Infrastructure / hosting strategy (added 2026-05-11 housekeeping; migrated from strategic-foundation §12 OQ-59) |

---

## Section 2 — OQ Triage Table (Sprint-1 8-column format)

| OQ ID | Source | Aggregate | Question | Dependency | Risk | Owner | Status |
|-------|--------|-----------|----------|------------|------|-------|--------|
| OQ-047 | ADR-0007 §2.2; mandate "public SEO pages" | Catalog (Cross-cutting) | Are SEO target markets DE-only for V1, or DE+EN from day-1? | Determines i18n scope for catalog projection, content strategy, translation workflow | M | AOT-Mgmt + Marketing | Open |
| OQ-048 | ADR-0007 §2.5; strategic-foundation §11 Ownership-Gap OG-1 | Cross-cutting | Who owns ongoing SEO performance — Marketing internal, Engineering, agency, joint team? | Determines team composition, V2 hiring/contracting, content-velocity assumptions | M | AOT-Mgmt | Open |
| OQ-049 | Mandate "public SEO pages"; Screenshot 1 (NET7 WebService-flag W) | Catalog | Which products are visible publicly — all `aktiv` products, or filtered via NET7 WebService-flag, or other criterion? | Determines Catalog adapter visibility-filter logic, public sitemap scope | H | AOT-Sales + AOT-PM | Open |
| OQ-050 | OQ-049 derivative | Catalog | What is the public sitemap policy — flat sitemap of all visible products, sitemap-by-Warengruppe, or other? | Determines crawl budget management, indexing prioritization | L | Engineering + Marketing | Open |
| OQ-051 | ADR-0005 §2.6 Phase 2; Sprint-1 Documents-Compliance Aggregate; Screenshot 1 (B2B-Dokumentenanzeige-Dropdown) | Documents-Compliance | What is the DocumentScope mapping rule from NET7's B2B-Dokumentenanzeige flag — Public / B2B / Internal / Customer-Specific defaults? Default-fail-safe should be B2B-only to prevent accidental public leak. | Determines documents adapter scope-classification logic, audit risk profile | H | AOT-Compliance + Legal | Open |
| OQ-052 | Mandate "one-stop experience"; Screenshot 8 (live Lagerbestand 4900.21 kg visible to authenticated user) | Catalog | Should anonymous users see inventory state? If yes — boolean "available" or exact quantity? Authenticated users currently see exact (Screenshot 8). | Determines Catalog projection field-level visibility rules | M | AOT-Sales | Open |
| OQ-053 | ADR-0007 §2.1; Screenshot 6 (7-staffel pricing) | Pricing-Contracts | What is the public-list-price anchor strategy — Staffel-1 VK frei Haus (highest published), "ab €X" range display, or other? | Determines Public-List-Price computation, Schema.org Product price field, frontend price-display rendering | H | AOT-Mgmt + Sales | Open |
| OQ-054 | Screenshot 6 ("A.Anfrage" rows at 190+ kg) | Pricing-Contracts | For "A.Anfrage" pricing tiers: anonymous user sees what — Login-CTA, RFQ-CTA, hidden tier, or "Auf Anfrage" placeholder? | Determines pricing-disclosure UX for high-volume tiers, RFQ flow entry-points | M | AOT-Sales + Engineering | Open |
| OQ-055 | Strategic-foundation §5.2 AR-3; Sprint-1 ADR-0003 Rule 5 (Read-Replica-Fallback-Discipline) | Pricing-Contracts | When NET7 pricing-API is unavailable, what is the customer-pricing fallback policy — stale-cached-price-with-banner, public-list-price-with-banner, error-state, or other? | Determines availability behaviour during NET7 incidents; alignment with Sprint-1 fallback rule | M | AOT-Mgmt + Engineering | Open |
| OQ-056 | ADR-0008 §2.2; strategic-foundation §1.2 | Sample | What are the concrete validation thresholds that trigger payment re-decision — monthly sample-volume number, conversion-rate target, AOT-Sales operational-load signal? Sprint-2 Phase-0 must instrument measurement; thresholds need numbers. | Determines metric instrumentation requirements; sets the bar for ADR-0008 revisit | M | AOT-Sales + AOT-Mgmt | Open |
| OQ-057 | ADR-0008 §4.3 | Sample | What is the AOT-Sales review cadence for sample-order volume — monthly, quarterly, ad-hoc? Who chairs the review? | Governance for ADR-0008 revisit trigger | L | AOT-Sales | Open |
| OQ-058 | ADR-0008 §4.2 sample-fraud risk | Sample | What anti-fraud measures are required for anonymous sample-requests in V1 — rate-limiting per IP/email, contact-validation, captcha, ProtonMail-style throwaway-email handling? | Engineering scope for V1 Sample-module; cost vs friction trade-off | M | AOT-Sales + Engineering | Open |
| OQ-059 | ADR-0007 §2.5; strategic-foundation §11 Ownership-Gap OG-2 | Cross-cutting (V2) | Who authors CMS content when introduced in V2 — existing AOT-team augmented, new editorial hire, external agency, hybrid? AOT does not currently appear to have a dedicated content/marketing role. | Determines V2 readiness, hiring/contracting plan, CMS vendor selection criteria | M | AOT-Mgmt | Open |
| OQ-060 | ADR-0007 §2.3; visual discovery Screenshot 3 (#D20072 inline-styled HTML) | Catalog | What is the brand-redesign migration trajectory — V1 sanitize-only is brittle; at what trigger does NET7-HTML migrate to CMS-managed? Brand-redesign initiative would force the question. | Triggers V2 CMS introduction; defines acceptable lifespan of sanitization-only approach | M | AOT-Mgmt + Marketing | Open |
| OQ-061 | OQ-047 derivative; ADR-0007 §2.5 | Catalog (Cross-cutting) | Multi-language content strategy — human-translation, auto-translation (DeepL or equivalent), hybrid for non-regulatory vs regulatory? | Determines translation workflow, editorial role scope, V2 CMS feature requirements | M | AOT-Mgmt + Marketing | Open |
| OQ-062 | ADR-0009 §1, §2.7 Gate G1; migrated from strategic-foundation §12 OQ-59 (2026-05-11 housekeeping) | Cross-cutting (Infrastructure) | Hosting strategy — cloud (which provider — Vercel + Railway proposed in ADR-0009, AWS/GCP/Azure alternatives, on-premise, hybrid)? AOT-IT-owned decision; gates ADR-0009 status transition from Proposed to Accepted. | Determines Phase-0 infrastructure choice; affects tech-stack lock-in, cost projections, AOT-IT operational responsibilities | H | AOT-IT | Open |

---

## Section 3 — Coupling to Sprint-1 Aggregates

Each new OQ couples to one or more Sprint-1 aggregate sketches. Couplings are explicit so aggregate updates remain traceable. Sprint-1 OQ-Aggregate-Coupling pattern continues.

| OQ ID | Primary Aggregate | Secondary Aggregate(s) | Sprint-1 INV/Component Affected |
|-------|-------------------|------------------------|-------------------------------|
| OQ-047 | Catalog | Documents-Compliance | New: i18n-projection-field-set |
| OQ-048 | Cross-cutting | — | Ownership-only (no INV change) |
| OQ-049 | Catalog | — | New: PublicVisibilityFlag (catalog-sprint-2-extension §2.1) |
| OQ-050 | Catalog | — | Adapter-output: sitemap-projection |
| OQ-051 | Documents-Compliance | Catalog | New: DocumentScope-field (documents-sprint-2-extension §2.1) |
| OQ-052 | Catalog | — | Projection field-visibility rule |
| OQ-053 | Pricing-Contracts | Catalog | New: PublicListPrice (pricing-sprint-2-extension §2.1) |
| OQ-054 | Pricing-Contracts | — | New: PriceVisibilityRule (pricing-sprint-2-extension §2.2) |
| OQ-055 | Pricing-Contracts | Customer-Account | INV update: fallback-behaviour-state-machine |
| OQ-056 | Sample | — | Measurement-instrumentation (no INV) |
| OQ-057 | Sample | — | Governance (no INV) |
| OQ-058 | Sample | Customer-Account | New: Anti-fraud guard (sample-sprint-2-extension §2.3) |
| OQ-059 | Cross-cutting | — | V2 only |
| OQ-060 | Catalog | — | V2 trigger |
| OQ-061 | Catalog | Documents-Compliance | V2 scope |
| OQ-062 | Cross-cutting | — | ADR-0009 Gate G1 (infrastructure) |

---

## Section 4 — Triplet-Closer Pattern (Sprint-1 ADR-0004 Rule 6)

Per Sprint-1 governance, each closed OQ must produce a triplet: (a) decision, (b) artifact update, (c) test/validation. The Sprint-2 OQs follow the same closure discipline. No closure entries yet — all OQs are Open status.

// ASSUMPTION: First closures expected during Sprint-2 Phase-0 Discovery sessions with AOT-Sales, AOT-Compliance, and AOT-Mgmt.

---

## Section 5 — Discovery Anchoring

Each OQ is anchored to either:
- A Sprint-2 ADR (0006 through 0009)
- A visual discovery screenshot (1 through 9)
- A strategic-foundation §section
- A Sprint-1 artifact

This preserves Sprint-1 ADR-0004 Rule 2 (Discovery-Anchoring Mandate) — no orphan OQs.

---

## Section 6 — Risk Distribution

| Risk Level | Count | OQs |
|------------|-------|-----|
| H (High) | 4 | OQ-049, OQ-051, OQ-053, OQ-062 |
| M (Medium) | 10 | OQ-047, OQ-048, OQ-052, OQ-054, OQ-055, OQ-056, OQ-058, OQ-059, OQ-060, OQ-061 |
| L (Low) | 2 | OQ-050, OQ-057 |

High-risk OQs (OQ-049, OQ-051, OQ-053, OQ-062) are Sprint-2 Phase-0 blocking — OQ-049/051/053 shape Catalog projection, Documents scope, and Pricing exposure respectively; OQ-062 gates ADR-0009 status transition (Proposed → Accepted). All must be answered before V1 implementation can fully begin.

---

## Section 7 — Merge-into-Master Instruction

On commit, this extension document is merged into `/docs/domain/open-questions-master.md` as follows:
1. OQ rows OQ-047 through OQ-062 append to the master triage table
2. Cluster summary (§1) merges into master TOC (now C1-C7)
3. Coupling (§3) merges into master coupling table
4. Risk distribution (§6) updates master risk summary
5. This extension file is then deleted (single-master-discipline preserved per Sprint-1 ADR-0004 Rule 1)

// ASSUMPTION: merge performed by Carlos or via Claude-Code in Sprint-2 Phase-0 commit. Until merged, both files coexist.
