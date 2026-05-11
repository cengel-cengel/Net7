# Customer-Account Aggregate — Sprint-2 Extension

**Baseline:** /docs/domain/aggregates/customer-account.md (Sprint-1 sketch 1.4.3)
**Style:** Sprint-1 ADR-0002 Pattern 1 (7-H2 Template), Pattern 5 (Self-Report Discipline)
**Extension scope:** GuestSession parallel concept for anonymous identity, conversion path to CustomerAccount, ADR-0006 `auth` module boundary
**Date:** 2026-05-11

---

## 1. Purpose

Document Customer-Account aggregate extensions and the introduction of a parallel **GuestSession** concept driven by the Sprint-2 anonymous-access mandate. GuestSession is NOT a CustomerAccount subtype — it is a separate concept living within the `auth` module scope (ADR-0006), with an explicit conversion path to CustomerAccount.

This document **extends** the Sprint-1 baseline (1.4.3) without modifying it. Sprint-1 modelled the CustomerAccount aggregate with its hybrid adapter pattern and security-boundary INVs; Sprint-2 layers anonymous-identity on top, careful to preserve Sprint-1's security guarantees.

## 2. Methodology — Pattern 5 Self-Report Discipline

### 2.0 Reused from Sprint-1 baseline (1.4.3)

- CustomerAccount as Aggregate Root
- AuthenticationCredential as platform-only concept (Sprint-1 INVs: never propagated to NET7 — ADR-0003 Rule 4)
- Hybrid adapter pattern: customer-master read from NET7; AuthenticationCredential platform-only
- Sprint-1 INVs governing customer-profile-sync (opt-in, never automatic, per Sprint-1 OQ-021)
- Security-boundary INVs (Sprint-1's "first cross-aggregate INV-tag" recorded for Customer-Account)

### 2.1 New components

| Component | Origin | Notes |
|-----------|--------|-------|
| `GuestSession` (parallel concept) | Mandate "anonymous discovery" + Sample-Order anonymous path | NEW concept, see §3 below. Lives in `auth` module (ADR-0006). NOT a CustomerAccount subtype. |
| `GuestSessionId` | derived | UUID, ephemeral, no PII initially. Becomes PII-bearing only on sample-submit. |
| `GuestSessionConversionTrigger` | OQ-052 derivative | Defines events that cause GuestSession → CustomerAccount conversion (operational, manual at AOT-Sales discretion in V1) |
| `UserRoleAssignment` (V1 basic, V2 hierarchies) | Mandate "B2B authenticated experience" | Sprint-1 baseline alluded to roles; Sprint-2 commits to V1 basic role-set: Buyer (default), Admin (account self-service). V2 adds Approver and parent/subsidiary hierarchies. |
| `IdentityBridgeMapping` | ADR-0006 `auth` module + `customer` module boundary | Platform-side mapping from `auth.UserId` to `customer.CustomerAccountId` (NET7 Kundenstamm reference). Sprint-1 hybrid adapter pattern realized as this explicit mapping. |

### 2.2 Components NOT relevant (out of scope V1)

- Multi-tenant account hierarchies (parent company / subsidiaries) — V2
- SSO (SAML, OIDC for enterprise customers) — V2
- API tokens for customer-ERP integration — V2
- Approval workflows (multi-step purchase approval) — V2
- Punch-out integration (SAP Ariba, Coupa) — V2
- Granular RBAC beyond Buyer/Admin — V2

## 3. GuestSession — Parallel Concept

GuestSession is a deliberately separate aggregate from CustomerAccount, with strict boundary discipline.

### 3.1 GuestSession Structure

Fields:
- GuestSessionId (UUID, platform-generated)
- CreatedAt timestamp
- ExpiresAt timestamp (default 30 days from creation, refreshable on activity)
- Locale (DE | EN, set on first interaction)
- CartState (transient sample-cart contents; replaceable across sessions)
- ContactDetails (email, name) — populated only on sample-submit, not on browse
- ShippingAddress — populated only on sample-submit
- ConversionState (browsing | cart-active | submitted | converted-to-customer-account | expired)
- AuditTrail (creation, conversion events; minimal — only what's needed for sample-submit traceability)

### 3.2 Constitutional Rules

- **GuestSession is Web-only.** It NEVER appears in NET7 unless converted.
- **No NET7-write triggered solely by GuestSession existence.** Only on sample-submit does a SampleOrder traverse to NET7 (per sample-sprint-2-extension §2.1).
- **GuestSession holds NO authentication credentials.** It is identity-less by design; conversion to CustomerAccount creates credentials in a separate flow.
- **DSGVO compliance**: minimal data collection (no PII until contact-detail-entry), explicit expiry, deletion-on-expiry, data-export-on-request handled by separate operational tool.

// ASSUMPTION: 30-day TTL is conservative default; configurable per Sprint-2 Phase-0 review with AOT-Compliance.

### 3.3 GuestSession-to-CustomerAccount Conversion Path

Two conversion paths exist:

**Path A — Self-service registration (post-sample-submit)**
- User submits sample-request as GuestSession
- Confirmation email includes optional account-creation link
- User clicks → completes account registration → CustomerAccount created in NET7 (via outbound adapter)
- GuestSession marked `converted-to-customer-account`; ContactDetails carry over to CustomerAccount; conversion logged in AuditTrail

**Path B — Operational conversion (AOT-Sales-driven)**
- AOT-Sales receives sample-request notification (via existing Bestellanfrage workflow)
- Sales decides to onboard customer
- Customer created in NET7 manually (existing AOT process)
- AOT-Sales or Innendienst sends customer the account-activation link
- Customer activates → CustomerAccount login enabled

Both paths preserve the constitutional rule: NET7-write happens only with operational consent, not automatically from GuestSession activity.

## 4. CustomerAccount Aggregate Extensions

The Sprint-1 CustomerAccount aggregate boundary is preserved. New fields added:

### 4.1 New fields on CustomerAccount (Aggregate Root)

| Field | Notes |
|-------|-------|
| `UserRoleAssignment[]` | V1: single role (Buyer) per user assigned to a CustomerAccount; Admin role for self-service. V2 expands. |
| `OriginGuestSessionReference` (nullable) | If CustomerAccount was created via conversion-from-GuestSession Path A, this references the originating GuestSessionId for analytics. Cleared after retention window. |
| `PriceClassAssignmentReference` | Reference to Pricing-Contracts aggregate (Sprint-1 baseline). No change to the assignment mechanism. |

### 4.2 Invariants (continuing INV-numbering from Sprint-1 baseline)

- **INV-(continue from Sprint-1)** (security-boundary, new): GuestSession data MUST NOT enter NET7 except via the explicit SampleOrder push (sample-sprint-2-extension §2.1) or via the GuestSession-to-CustomerAccount conversion path (§3.3). Direct sync of GuestSession state to NET7 is prohibited.
- (state-machine, new): GuestSession state-transitions are platform-only. NET7 has no GuestSession concept; NET7-write only occurs at sample-submit or at customer-creation, never at intermediate GuestSession transitions.
- (cross-aggregate, new): IdentityBridgeMapping is the SOLE mapping between `auth.UserId` and `customer.CustomerAccountId`. No other module reaches across modules to associate identity — they call `auth` module's public interface (ADR-0006).
- (security-boundary, new): AuthenticationCredential (Sprint-1 ADR-0003 Rule 4 constitutional) NEVER appears in CustomerAccount Aggregate Root. Credentials live in `auth` module, mapped via IdentityBridgeMapping. Sprint-1 baseline alignment preserved.

// ASSUMPTION: precise INV numbering coordinates with Sprint-1 file on merge. Numbering placeholder here.

## 5. Web/ERP Ownership Mapping

| Field | Owned by NET7 | Owned by Platform | Notes |
|-------|---------------|-------------------|-------|
| CustomerAccountId (NET7 Kundennummer) | ✓ | — | NET7 master |
| Customer master data (name, address, tax-id, etc.) | ✓ | — | NET7 master (per Sprint-1 baseline) |
| AuthenticationCredential | — | ✓ | Platform-only (Sprint-1 constitutional) |
| `auth.UserId` (platform identity) | — | ✓ | Platform-only |
| IdentityBridgeMapping (UserId ↔ CustomerAccountId) | — | ✓ | Platform-only |
| UserRoleAssignment | — | ✓ | Platform-only (V1) |
| GuestSession entirely | — | ✓ | Web-only concept |
| PriceClassAssignmentReference value | ✓ | — | Reference to NET7-owned PriceClass assignment |
| Customer-profile-sync (opt-in) | Hybrid | Hybrid | Sprint-1 baseline pattern preserved; opt-in only |

## 6. Cross-Aggregate References and OQs

### Cross-aggregate references

- → Pricing-Contracts aggregate: PriceClassAssignmentReference (Sprint-1 baseline)
- → Sample aggregate: BuyerIdentity binding (CustomerAccountReference or GuestSessionReference, per sample-sprint-2-extension §2.1)
- → RFQ aggregate (Sprint-2 V1 scope): authenticated B2B RFQ submissions reference CustomerAccount
- → Documents-Compliance aggregate: customer-specific document scope (documents-sprint-2-extension §2.1)
- → `auth` module (ADR-0006): IdentityBridgeMapping consumed by all other modules needing identity

### Open Questions (from oq-master-sprint-2-extension)

- OQ-052 (anonymous inventory visibility) — affects GuestSession Catalog interaction
- OQ-055 (NET7 pricing-API fallback for authenticated users) — affects CustomerAccount price-display
- OQ-058 (anti-fraud measures) — affects GuestSession sample-submit validation

## 7. References

- /docs/domain/aggregates/customer-account.md (Sprint-1 baseline 1.4.3, first cross-aggregate INV-tag recorded here)
- /docs/adr/0002-domain-patterns.md (Pattern 1, Pattern 5 Self-Report Discipline)
- /docs/adr/0003-erp-boundary-rules.md (Rule 4 Authentication-Credential-Containment — constitutional)
- /docs/adr/0005-strangler-fig.md (migration umbrella; CustomerAccount migration in Phase-3/Phase-4)
- /docs/adr/0006-modular-monolith.md (`auth` and `customer` module boundary; IdentityBridgeMapping concept)
- /docs/adr/0008-payment-strategy.md (no payment-coupling to CustomerAccount in V1)
- /docs/domain/aggregates/sample.md (BuyerIdentity polymorphic across CustomerAccount and GuestSession; see sample-sprint-2-extension)
- /docs/domain/aggregates/pricing-contracts.md (PriceClassAssignmentReference; see pricing-sprint-2-extension)
- /docs/domain/aggregates/documents-compliance.md (customer-specific document scope; see documents-sprint-2-extension)
- /docs/domain/open-questions-master.md (OQ-052, OQ-055, OQ-058 — see oq-master-sprint-2-extension)
- Visual discovery: Screenshot 4 (current B2B-portal session-based identity), Screenshot 7 (Kundenadresse + PriceClass-assignment)
