# Master Data Risk Register — AOT Procurement Platform

## 1. Purpose

This document is the risk register for ERP-master-data-quality risks that threaten procurement UX, compliance workflows, and V1-launch readiness on the AOT Digital Procurement Platform. It is a Sprint-1-Closeout artefact (P2 of 5+3 mandatory items) and a direct input to Sprint-2 *ERP Boundary Design* triage and the V1-pre-launch mitigation plan.

Per `AGENTS.md` framing: *bad master data kills procurement UX, not architecture*. The five Sprint-1 aggregate sketches assume usable ERP master data. This document tracks the risks if that assumption fails — concrete, AOT-domain-specific risks (organic raw materials, cosmetics + food ingredients) rather than generic data-quality categories.

The register frames *what could go wrong with master data*, *how likely it is*, *how bad it would be*, and *what we plan to do about it*. It does **not** record actual data-quality findings — those land here only after the first real-data pulls in Sprint 2/3 move a risk from `Hypothesis` to `Identified`.

## 2. How to Use

- **Risk Lifecycle**: every risk moves through *Hypothesis* → *Identified* → *In-Mitigation* → *Mitigated* → *Resolved*. Status starts at *Hypothesis* and advances when discovery (TopM-Engineer interviews, customer interviews) or a real-data pull confirms or revises the risk. *Resolved* is reserved for risks that are no longer plausible after data-cleanup or process-change.
- **Discovery trigger**: Sprint-2 TopM-discovery sessions (`erp-discovery.md` clusters) and the first real master-data pulls (Catalog, Pricing, Documents, Customer) in Sprint 2/3 are the primary validators.
- **Mitigation ownership** is tagged inline in the Mitigation column with one of three categories:
  - `[AOT-cleanup]` — AOT-internal master-data fixes (renewals, dedup, missing-data fills, naming conventions).
  - `[Web-fallback]` — web-side graceful degradation (validation, banners, fallback labels, expiry-aware filtering).
  - `[NET7-schema]` — NET7-side schema or process changes (rarer, requires TopM coordination).
  Many risks carry combined ownership (e.g., `[AOT-cleanup] + [Web-fallback]`).
- **Cross-link to OQ-master**: where a master-data risk overlaps with an open architecture question in [`open-questions-master.md`](../domain/open-questions-master.md), the Mitigation column carries the OQ reference. Resolution of the OQ may also resolve or reframe the risk.
- **Forward-link**: lifecycle governance for this register and the OQ register will be formalised in a future `docs/adr/0003-erp-boundary-rules.md` (Sprint-1-Closeout P5b) and `docs/adr/0004-open-question-governance.md` (Sprint-1-Closeout P5c).
- **Probability levels**: *High* = typical reality at mid-market ERP suppliers without strict master-data governance; *Medium* = plausible but not universal; *Low* = structurally unlikely but high-impact when it occurs.
- **Impact tags** (multi-tag permitted): *UX-degradation*, *Compliance-failure*, *Order-blocking*, *Trust-loss*, *Regulatory-exposure*.

## 3. Risk Register Table

18 risks across 6 domains. Sorted by Risk ID, which groups by domain.

| Risk ID | Domain        | Description                                                                                            | Impact                                            | Probability | Mitigation                                                                                                     | Status     |
|---------|---------------|--------------------------------------------------------------------------------------------------------|---------------------------------------------------|-------------|----------------------------------------------------------------------------------------------------------------|------------|
| MDR-001 | Catalog       | Missing INCI names on cosmetics-segment products. Breaks INCI-search facet (CAP-002), blocks regulatory completeness on product detail. | UX-degradation, Compliance-failure                | High        | `[AOT-cleanup]` master-data fill against INCI master list; `[Web-fallback]` show "INCI pending" badge instead of empty field. | Hypothesis |
| MDR-002 | Catalog       | Missing CAS numbers, especially on plant-extract and natural-blend products without single-substance identity. | UX-degradation                                    | Medium      | `[AOT-cleanup]` per-product CAS review; `[Web-fallback]` allow CAS-N/A label for legitimately CAS-less products. | Hypothesis |
| MDR-003 | Catalog       | Duplicate articles — same product surfaces under multiple SKUs (legacy migrations, naming variations, packaging sub-SKUs). | UX-degradation, Trust-loss                        | Medium      | `[AOT-cleanup]` dedup project against canonical product master; `[Web-fallback]` web-side dedup heuristic on INCI+Origin until clean. | Hypothesis |
| MDR-004 | Catalog       | Specification incompleteness — purity bands, particle size, active content fields blank on a non-trivial fraction of products. | UX-degradation                                    | Medium      | `[AOT-cleanup]` per-product spec audit; `[Web-fallback]` "specification on request" CTA instead of empty fields. | Hypothesis |
| MDR-005 | Documents     | Broken document links — `DocumentFileReference` URI returns 404 (signed-link expiry, file-storage migrations, manual archival). | Compliance-failure, Order-blocking                | High        | `[Web-fallback]` 3-stage chain: cached URL → fresh fetch → direct NET7-DMS link; `[AOT-cleanup]` per-doc link audit. Couples to OQ-029 (URL semantics). | Hypothesis |
| MDR-006 | Documents     | Document-Language gaps — DE/EN coverage uneven; SDS or COA available only in one locale for non-trivial product sets. | UX-degradation, Compliance-failure (non-DE markets) | High        | `[Web-fallback]` EN-fallback when DE absent (per OQ-028); `[AOT-cleanup]` translation backlog prioritised by Customer-segment. Couples to OQ-028. | Hypothesis |
| MDR-007 | Documents     | Expired certificates not flagged — `ExpiryDate` past but certificate still surfaces as "current" in completeness checks (CAP-008). | Compliance-failure, Regulatory-exposure           | High        | `[Web-fallback]` expiry-aware filtering at read-time; `[AOT-cleanup]` cert-renewal process and 30-day-expiry warning to AOT-Sales. | Hypothesis |
| MDR-008 | Documents     | Allergen-statement missing for food-segment products. Required by `LFGB` and EU food-labelling rules; absence is regulatory exposure for AOT customers selling into Germany. | Compliance-failure, Regulatory-exposure           | High        | `[AOT-cleanup]` food-segment allergen-statement audit; `[Web-fallback]` block food-segment product display until allergen-statement linked. | Hypothesis |
| MDR-009 | Documents     | Document-versioning conflicts — duplicate `IssueDate` per Document, broken version history, ingest-time INV-4 violations. | Audit-failure, Compliance-failure                 | Medium      | `[NET7-schema]` ensure unique IssueDate enforcement on NET7 side; `[AOT-cleanup]` history-cleanup; `[Web-fallback]` escalate INV-4 ingest violations rather than silently dedup. | Hypothesis |
| MDR-010 | Pricing       | Missing customer-specific prices for products that are otherwise active in catalog. CAP-018 *Place Order* fails for these products. | Order-blocking, UX-degradation                    | High        | `[AOT-cleanup]` per-customer-per-product price audit; `[Web-fallback]` "request quote" path instead of "place order" when ProductPrice missing. | Hypothesis |
| MDR-011 | Pricing       | Stale prices — `ProductPrice` not updated for many months despite product or contract changes; `PriceValidity` expired or absent. | Trust-loss, Order-friction                        | Medium      | `[Web-fallback]` `last-updated` banner per product; `[AOT-cleanup]` quarterly price-review cycle; couples to OQ-038 (future-dated prices). | Hypothesis |
| MDR-012 | Pricing       | Currency-mix inconsistencies — international customers with multi-currency contracts show inconsistent or missing currency on individual `ProductPrice` rows. | UX-degradation, Order-friction                    | Medium      | `[AOT-cleanup]` per-contract currency audit; `[Web-fallback]` per-Price currency rendering (per OQ-036). Couples to OQ-036. | Hypothesis |
| MDR-013 | Customer      | Outdated customer contacts — primary email bounces, phone disconnects, contact has left the company. Auth flows (CAP-022) and notification flows fail. | Trust-loss, Auth-flow-failure                     | High        | `[Web-fallback]` bounce-detection on outbound email + flag user as `Locked` until refreshed; `[AOT-cleanup]` periodic contact-verification outreach. | Hypothesis |
| MDR-014 | Customer      | Customer-ID instability across mergers, renames, or successor-account creation. Breaks identity-bridging and cross-aggregate ID-references. | Cross-aggregate-data-loss, Auth-flow-failure      | Medium      | `[NET7-schema]` Customer-ID stability commitment; `[Web-fallback]` reconciliation tooling for known-rename events. Couples to OQ-020 (UserId source). | Hypothesis |
| MDR-015 | Sample/Batch  | Missing batch references on shipped samples. Sample-COAs cannot be linked to the physical sample the buyer received; traceability fails. | Compliance-failure, Traceability-failure          | High        | `[AOT-cleanup]` sample-dispatch process discipline (batch stamped at pick); `[Web-fallback]` "batch-info pending" placeholder with explicit ETA when missing. Couples to OQ-027 (BatchReference placement). | Hypothesis |
| MDR-016 | Sample/Batch  | Sample-COA missing — sample shipped without per-batch certificate of analysis, especially food-segment.                | Compliance-failure                                | Medium      | `[AOT-cleanup]` sample-dispatch QA gate (no ship without COA); `[Web-fallback]` block "Tested" status transition until COA linked. | Hypothesis |
| MDR-017 | Sample/Batch  | Sample-to-product cross-reference broken — `SampleProductReference` points to retired or merged Product. CC-1 conversion fails. | Sample-conversion-failure                         | Low         | `[Web-fallback]` reconciliation against Catalog at conversion time; `[AOT-cleanup]` retired-product reverse-link table. | Hypothesis |
| MDR-018 | Cross-cutting | NET7 sync-lag exceeds business threshold — status events (Sample, Order, ContractStatus, PriceValidity) reach the web mirror late or not at all. | UX-degradation, Financial-inconsistency (Pricing) | Low         | `[Web-fallback]` `last-synced-at` banner + business-process-blocking when threshold exceeded (per `pricing-contracts.md` §5 failure-mode chain); `[NET7-schema]` push-event-stream from NET7. Couples to Group-3 Status-Sync Mechanism Family in OQ-master. | Hypothesis |

**Probability distribution**: 8 High / 8 Medium / 2 Low = 18.
**Domain distribution**: 4 Catalog / 5 Documents / 3 Pricing / 2 Customer / 3 Sample-Batch / 1 Cross-cutting = 18.

## 4. Risk Categories Overview

High-level map by domain. Counts are total risks per domain (any probability/impact); see §5 for the V1-blocking subset.

- **Catalog (Product master)**: 4 risks — `MDR-001` INCI, `MDR-002` CAS, `MDR-003` Duplicates, `MDR-004` Specifications. Pattern: master-data completeness and naming hygiene.
- **Documents**: 5 risks — `MDR-005` Broken links, `MDR-006` Language gaps, `MDR-007` Expired certs, `MDR-008` Allergen, `MDR-009` Versioning. Pattern: regulatory-relevant content with expiry and version semantics.
- **Pricing**: 3 risks — `MDR-010` Missing prices, `MDR-011` Stale prices, `MDR-012` Currency-mix. Pattern: customer-specific price freshness and consistency.
- **Customer**: 2 risks — `MDR-013` Outdated contacts, `MDR-014` Customer-ID instability. Pattern: identity-and-contact freshness.
- **Sample/Batch**: 3 risks — `MDR-015` Missing batch refs, `MDR-016` Missing sample-COA, `MDR-017` Broken cross-ref. Pattern: traceability and lifecycle integrity.
- **Cross-cutting**: 1 risk — `MDR-018` Sync-lag. Pattern: adapter-level rather than per-aggregate.

## 5. Critical Risks for V1 Launch

Subset of risks that combine `Probability = High` with `Impact ∈ {Order-blocking, Compliance-failure, Regulatory-exposure}`. These are V1-launch-blockers unless mitigated. Six items:

### CR-1 — MDR-001: Missing INCI names

- **Why V1-blocking**: cosmetics-segment customers buy primarily by INCI; a non-trivial fraction of products without INCI is a credibility failure on the catalogue's primary search facet (CAP-002), and INCI absence breaks regulatory completeness on the product detail page.
- **Pre-V1-mitigation**: AOT-side INCI-master-list fill before launch (`[AOT-cleanup]`), combined with a web-side "INCI pending" badge for residual gaps (`[Web-fallback]`).

### CR-2 — MDR-005: Broken document links

- **Why V1-blocking**: SDS and COA are required documents at product detail and at order placement; broken links cause direct compliance and order-blocking failures. Couples to OQ-029 (`DocumentFileReference` URL semantics — stable vs signed/short-lived).
- **Pre-V1-mitigation**: web-side three-stage fallback chain (cached URL → fresh fetch → direct NET7-DMS link, per `documents-compliance.md` §5 failure-mode), combined with an AOT-side per-doc link audit before launch.

### CR-3 — MDR-007: Expired certificates not flagged

- **Why V1-blocking**: expired Bio-, kosher-, halal-, or audit-issued certificates surfacing as "current" exposes AOT customers to regulatory non-compliance. The compliance failure is downstream of AOT but originates in master-data hygiene.
- **Pre-V1-mitigation**: web-side expiry-aware filtering at read time (per `documents-compliance.md` `INV-7`), combined with an AOT-side cert-renewal process and a 30-day-expiry warning to AOT-Sales.

### CR-4 — MDR-008: Allergen-statement missing for food-segment products

- **Why V1-blocking**: required by `LFGB` (and EU food-labelling rules); absence on food-segment products is a regulatory exposure for AOT customers selling into Germany. Specifically risks the customer's labelling compliance, which is a high-trust dependency on AOT.
- **Pre-V1-mitigation**: AOT-side food-segment allergen-statement audit (every food-segment product must have a current allergen-statement linked) plus a web-side hard block on food-segment product display until the allergen-statement is linked.

### CR-5 — MDR-010: Missing customer-specific prices for active products

- **Why V1-blocking**: `CAP-018` *Place Order* depends on a `ProductPrice` for `(Customer, Product)`; absence blocks the order flow entirely. A product showing in catalog but unorderable is a worse experience than not showing at all.
- **Pre-V1-mitigation**: AOT-side per-customer-per-product price audit before launch, combined with a web-side fallback that swaps the *Place Order* CTA for *Request Quote* when `ProductPrice` is missing — degrades gracefully into the RFQ flow rather than dead-ending.

### CR-6 — MDR-015: Missing batch references on shipped samples

- **Why V1-blocking**: sample-COAs cannot be linked to the physical sample without a batch reference; traceability fails at the most credibility-sensitive moment (the buyer testing a sample). Couples to OQ-027 (BatchReference placement) which itself is a Critical-Path OQ for Sprint 2.
- **Pre-V1-mitigation**: AOT-side sample-dispatch process discipline (batch stamped at pick, no exception); web-side "batch-info pending" placeholder with explicit ETA; OQ-027 resolution determines whether `BatchReference` lives in Catalog `ProductVariant` or a future Inventory aggregate.

## 6. References

- [`../../AGENTS.md`](../../AGENTS.md) — engineering charter; ERP Ownership Rules. Source of the *"bad master data kills procurement UX, not architecture"* framing that motivates this register.
- [`../procurement-hypotheses.md`](../procurement-hypotheses.md) — §3 *Decision Criteria* (documentation completeness as primary buyer driver), §6 *Document Completeness* (UX hypothesis), §6 *Availability Confidence* (data-quality dependency).
- [`../erp-discovery.md`](../erp-discovery.md) — ERP-side discovery clusters: §3.c Datenmodell, §3.d Lese-Operationen, §3.f Sync-Strategie. Primary validator of the *Hypothesis* status.
- [`../business-discovery.md`](../business-discovery.md) — Buyer-side data-pain interview clusters: §3.c Schmerzpunkte, §3.d Sample-Workflow, §3.f Dokumentations-Bedürfnisse, §3.h Lieferanten-Beziehungen.
- [`../domain/aggregates/catalog.md`](../domain/aggregates/catalog.md), [`sample.md`](../domain/aggregates/sample.md), [`customer-account.md`](../domain/aggregates/customer-account.md), [`documents-compliance.md`](../domain/aggregates/documents-compliance.md), [`pricing-contracts.md`](../domain/aggregates/pricing-contracts.md) — per-domain master-data context.
- [`../domain/open-questions-master.md`](../domain/open-questions-master.md) — Cross-link target. Several MDR risks couple to specific OQs (`MDR-005` ↔ OQ-029, `MDR-006` ↔ OQ-028, `MDR-011` ↔ OQ-038, `MDR-012` ↔ OQ-036, `MDR-014` ↔ OQ-020, `MDR-015` ↔ OQ-027, `MDR-018` ↔ Group-3 Status-Sync Mechanism Family).
- *Future*: `docs/adr/0003-erp-boundary-rules.md` (Sprint-1-Closeout P5b) and `docs/adr/0004-open-question-governance.md` (Sprint-1-Closeout P5c) will formalise risk-and-OQ-lifecycle governance and ADR-promotion paths.
