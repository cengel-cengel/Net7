# Documents-Compliance Aggregate — Sprint-2 Extension

**Baseline:** /docs/domain/aggregates/documents-compliance.md (Sprint-1 sketch 1.4.4)
**Style:** Sprint-1 ADR-0002 Pattern 1 (7-H2 Template), Pattern 5 (Self-Report Discipline)
**Extension scope:** DocumentScope mapping from NET7 visibility flags, default-fail-safe rule, anonymous-access document scoping
**Date:** 2026-05-11

---

## 1. Purpose

Document Documents-Compliance aggregate extensions required by the public-catalog anonymous-access mandate. Visual discovery confirmed NET7 has visibility flags (WebService-flag "W" and B2B-Dokumentenanzeige-Dropdown, Screenshot 1) that must be mapped to platform DocumentScope classifications.

This document **extends** the Sprint-1 baseline (1.4.4) without modifying it. Sprint-1 modelled Documents with read-only-mirror notation and DMS-versioning awareness; Sprint-2 adds explicit scope-classification to support anonymous access without accidentally leaking B2B-confidential documents.

## 2. Methodology — Pattern 5 Self-Report Discipline

### 2.0 Reused from Sprint-1 baseline (1.4.4)

- Document as Aggregate Root (with DocumentVersion as child entity)
- Read-only-mirror notation (Sprint-1 Pattern 3) — NET7 DMS is the System of Record
- DocumentVersion lifecycle (Sprint-1 INVs: status, prüfintervall, workflow-chain)
- Document-Customer scope mapping (Sprint-1 baseline: customer-specific document visibility)
- Audit trail (Sprint-1 baseline: regulatory query, download events)

### 2.1 New components

| Component | Origin | Notes |
|-----------|--------|-------|
| `DocumentScope` (classification) | Visual discovery Screenshot 1 (WebService-flag W + B2B-Dokumentenanzeige-Dropdown); OQ-051 | Classifies each document as Public / B2B / Internal / Customer-Specific. Drives Web-tier visibility decisions. |
| `DocumentScopeMappingRule` | OQ-051 | Rule for translating NET7 flags into DocumentScope. **Default-fail-safe = B2B-only** if mapping ambiguous. Prevents accidental public leak. |
| `PublicDocumentProjection` | ADR-0005 Phase-2 (public document access) | Read-model projection of Documents where DocumentScope == Public. Edge-cacheable. |
| `AnonymousAccessAuditEntry` | DSGVO and audit consideration | New audit-trail entry-type for anonymous public-document downloads. Captures IP + timestamp + DocumentVersionId, retention per DSGVO policy. |

### 2.2 DocumentScope — four-tier classification

| Scope | Visible To | NET7 Source Signal (anticipated) | Examples (per visual discovery) |
|-------|-----------|----------------------------------|--------------------------------|
| **Public** | Anonymous + Authenticated | NET7 WebService-flag W AND B2B-Dokumentenanzeige set to public-equivalent | Screenshot 5: AOT-MSDS DE/EN (regulatory, suitable for public catalog), Allergen Statement |
| **B2B** | Authenticated B2B (any) | NET7 WebService-flag W AND B2B-Dokumentenanzeige set to B2B-equivalent | Detailed COA, Flowchart |
| **Customer-Specific** | Authenticated B2B (matching customer-account binding) | NET7 customer-specific document attachment | Customer-specific compliance documents, custom specs |
| **Internal** | Not visible on Web at all | NET7 WebService-flag absent OR explicit internal | Internal QM documents, supplier docs |

// ASSUMPTION: precise mapping between B2B-Dokumentenanzeige-Dropdown values and the four-tier classification requires Discovery confirmation from AOT-Compliance and TopM-Eng. Default-fail-safe rule (B2B-only unless clearly public) absorbs ambiguity safely.

### 2.3 Default-Fail-Safe Rule (constitutional)

If a document's NET7 flags are ambiguous or the mapping rule produces an undefined result, the platform defaults to:
- DocumentScope = **B2B**
- Public visibility = **false**

This is constitutional: accidental B2B-leak risk is asymmetric with accidental over-restriction. Over-restriction is correctable (extend mapping, re-classify document). Public leak is unrecoverable (search engines cache, third parties archive).

Sprint-1 ADR-0003 Rule 2 (Schema-Drift-Containment) and Rule 4 (Authentication-Credential-Containment) align with this fail-safe pattern.

### 2.4 Components NOT relevant (out of scope V1)

- CMS-managed documents (V2, per ADR-0007 §2.6 regulatory documents stay in NET7 DMS)
- Document signing workflow (out of scope entirely)
- Document expiration auto-takedown (V1: surface "expired" status, V2: auto-hide if compliance requires)
- Customer-document upload (customers uploading documents to platform — V2 scope at earliest)

## 3. Aggregate Boundary

Sprint-1 baseline aggregate identity preserved: Document remains the Aggregate Root with DocumentVersion as child entity. DocumentScope is a new field on the Aggregate Root (per DocumentVersion if scope varies by version — see §4.2 INV).

PublicDocumentProjection is a read-model projection, not a separate aggregate. It exists in the `documents` module (ADR-0006).

## 4. Internal Structure

### 4.1 Document (Aggregate Root) — extended

Sprint-1 fields (unchanged): DocumentId, DocumentNumber (DMS-Nr), Title, DocumentType (SPEC | MSDS | COA | FC | ALL | other), ProductReferences[], DocumentVersion[], CurrentVersionId

Sprint-2 added fields:
- DocumentScope (per Document — see §4.2 INV for per-version variation)
- ScopeSource (the NET7-flag-set that determined the scope, for audit traceability)
- LastScopeReview (timestamp + reviewer-id if available)

### 4.2 Invariants (continuing INV-numbering from Sprint-1 baseline)

- **INV-(continue from Sprint-1)** (security-boundary, new): DocumentScope MUST be evaluated PER DocumentVersion, not per Document. A document's older version may have been Public; a newer version may be B2B-only if regulatory content changed. // ASSUMPTION: NET7's B2B-Dokumentenanzeige flag operates per-version; if per-Document only, scope is derived per-Document. Validated during Sprint-2 Phase-0 adapter implementation.
- (security-boundary, new): When DocumentScopeMappingRule produces ambiguous or undefined output, default to DocumentScope = B2B, Public visibility = false. **Constitutional fail-safe.**
- (state-machine, new): DocumentScope transition (e.g., document reclassified from Public to B2B) MUST trigger:
  - Removal from PublicDocumentProjection within edge-cache invalidation window (target <60s)
  - Search-index update (document no longer indexed if scope demotes from Public)
  - AuditTrail entry with scope-change reason
- (cross-aggregate, new): PublicDocumentProjection consumption by Catalog module surfaces a subset of documents on the public product page. The set is computed via INNER JOIN of (Product, Document, DocumentScope==Public). Catalog does not re-derive scope; it consumes the documents-module projection.
- (behavioural, new): AnonymousAccessAuditEntry MUST be recorded for every anonymous download of a Public document. Retention per DSGVO policy (// ASSUMPTION: 6-month default, configurable post-Discovery).
- (behavioural, new): Documents with DocumentScope == Internal MUST NOT appear in any Web-tier projection (public OR B2B). The platform pretends they do not exist. Sprint-1 Documents baseline's customer-specific scope is preserved at finer granularity.

## 5. Web/ERP Ownership Mapping

| Field | Owned by NET7 | Owned by Platform | Notes |
|-------|---------------|-------------------|-------|
| Document master (DMS-Nr, content, versions) | ✓ | — | NET7 DMS (Screenshot 1) |
| Workflow-chain (bearbeitet/geprüft/freigegeben) | ✓ | — | NET7 DMS, surfaced as read-only mirror |
| Prüfintervall, Wiedervorlage | ✓ | — | NET7 DMS |
| WebService-flag, B2B-Dokumentenanzeige | ✓ | — | NET7 source signals |
| DocumentScope (classification) | Hybrid | Hybrid | Sourced from NET7 flags via DocumentScopeMappingRule, but the rule and the resulting classification are platform-side. Platform applies fail-safe defaults. |
| PublicDocumentProjection (read-model) | — | ✓ | Platform read-model |
| Download URLs (public, CDN-signed) | — | ✓ | Platform-generated; NET7 file content is the source asset |
| AnonymousAccessAuditEntry | — | ✓ | Platform-only |
| ScopeChange events | — | ✓ | Platform observes NET7 source-signal changes and publishes scope-change events |

## 6. Cross-Aggregate References and OQs

### Cross-aggregate references

- → Catalog aggregate: PublicDocumentProjection consumed by Catalog Product view (per product, scoped documents listed)
- → Customer-Account aggregate: customer-specific document scope binding (Sprint-1 baseline preserved)
- → `documents` module (ADR-0006): owns scope-derivation rule and projection
- → `search` module (ADR-0006): consumes PublicDocumentProjection for document-search-index

### Open Questions (from oq-master-sprint-2-extension)

- OQ-051 (DocumentScope mapping rule — concrete, primary driver of this extension)
- OQ-049 (Catalog public visibility) — interaction: a Catalog Product marked public may have B2B-only documents; the product page shows public documents only, B2B documents hidden until authentication

## 7. References

- /docs/domain/aggregates/documents-compliance.md (Sprint-1 baseline 1.4.4, read-only-mirror notation introduced here)
- /docs/adr/0002-domain-patterns.md (Pattern 1, Pattern 3 read-only-mirror, Pattern 5 Self-Report Discipline)
- /docs/adr/0003-erp-boundary-rules.md (Rule 1 NET7-as-SoT, Rule 2 Schema-Drift-Containment — alignment with fail-safe default)
- /docs/adr/0005-strangler-fig.md (migration umbrella; Documents in Phase-2 — public-scoped documents — and Phase-3 — full B2B scope)
- /docs/adr/0006-modular-monolith.md (`documents` module boundary)
- /docs/adr/0007-content-strategy.md (regulatory documents remain in NET7 DMS; not migrated to CMS)
- /docs/domain/aggregates/catalog.md (Catalog consumes PublicDocumentProjection per Product; see catalog-sprint-2-extension)
- /docs/domain/aggregates/customer-account.md (customer-specific document scope; see customer-account-sprint-2-extension)
- /docs/domain/open-questions-master.md (OQ-049, OQ-051 — see oq-master-sprint-2-extension)
- /docs/risks/master-data-risk-register.md (MDR-005 Stale-Doc-Risk and related — direct alignment with scope-change INV)
- Visual discovery: Screenshot 1 (NET7 DMS detail, WebService-flag, B2B-Dokumentenanzeige), Screenshot 2 (multi-version document list), Screenshot 5 (public product page document section)
