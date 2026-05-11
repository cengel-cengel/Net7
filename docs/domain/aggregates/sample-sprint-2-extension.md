# Sample Aggregate — Sprint-2 Extension

**Baseline:** /docs/domain/aggregates/sample.md (Sprint-1 sketch 1.4.2)
**Style:** Sprint-1 ADR-0002 Pattern 1 (7-H2 Template), Pattern 5 (Self-Report Discipline)
**Extension scope:** MAJOR REFACTOR — Sample-as-Quantity-Option, PaymentBasedSample removed (per ADR-0008), GuestSession trigger for anonymous path
**Date:** 2026-05-11

---

## 1. Purpose

This extension records a **significant refactor** of the Sprint-1 Sample Aggregate driven by two Sprint-2 evidence sources:

1. **Visual discovery (Screenshot 8)** — "Muster" is the 8th quantity-option button on the catalog product page, alongside the 7 commercial-staffel-quantities (1, 4.5, 9, 23, 115, 190, 760 kg). Sample is NOT a separate catalog entity; it's a quantity-option on the existing Product.

2. **ADR-0008 (Payment Deferred)** — V1 ships Sample-Order workflow without online payment integration. Sprint-1's `PaymentBasedSample` aggregate subtype is removed from V1 scope.

This extension supersedes (with explicit acknowledgement) the Sprint-1 subtype-decomposition. The aggregate identity (Sample as workflow-Aggregate) remains; the catalog-side representation and the payment-coupling are revised.

## 2. Methodology — Pattern 5 Self-Report Discipline

### 2.0 Reused from Sprint-1 baseline (1.4.2)

- SampleOrder as Aggregate Root
- BuyerIdentity value object (referencing CustomerAccount when authenticated)
- SampleOrderLifecycle state-machine (Sprint-1 INVs governing transitions)
- Outbound-write-path adapter pattern (Sprint-1 ADR-0003 Rule 3 Idempotency Mandate)
- NET7 as destination of SampleOrder persistence (Bestellanfrage workflow)

### 2.1 Refactored components

| Component | Sprint-1 | Sprint-2 | Origin |
|-----------|----------|----------|--------|
| Sample-Catalog representation | Separate Sample-Aggregate-Subtype | Quantity-option `SampleQuantityOption` on Catalog Product | Visual discovery Screenshot 8 |
| PaymentBasedSample subtype | Defined in baseline | **REMOVED** for V1 | ADR-0008 |
| RequestBasedSample subtype | Defined in baseline | **Promoted to single SampleOrder type** for V1 | ADR-0008 simplification |
| Sample-Order trigger | Authenticated B2B only (Sprint-1) | Anonymous + Authenticated, both via GuestSession or CustomerAccount | Mandate "anonymous discovery" + Screenshot 4 (current Bestellanfrage workflow) |

### 2.2 New components

| Component | Origin | Notes |
|-----------|--------|-------|
| `GuestSessionReference` | Mandate "anonymous discovery"; OQ-052 | When BuyerIdentity is not a CustomerAccount, SampleOrder references a GuestSession (see customer-account-sprint-2-extension §2.1). GuestSession holds contact details and shipping address. |
| `SampleRequestSource` | OQ-058 | Track origin (public-catalog vs B2B-portal vs API future) — supports fraud-monitoring and conversion analytics |
| `AntiFraudGuard` | OQ-058 | V1 measures: per-IP rate-limit, per-email rate-limit, optional captcha on submission. // ASSUMPTION: specific rate-limits set during Sprint-2 Phase-0 based on AOT-Sales sample-volume expectations. |
| `PaymentReference` (placeholder, null in V1) | ADR-0008 §2.3 | Field reserved on Aggregate Root for V2 payment introduction. Always null in V1. Preserves V2 reversibility. |
| `ValidationMetrics` (instrumentation, not aggregate state) | ADR-0008 §2.2; OQ-056 | Volume, conversion-rate, AOT-Sales-operational-load signals captured for ADR-0008 revisit trigger |

### 2.3 Components NOT relevant (removed or deferred)

- PaymentBasedSample subtype — V2 candidate per ADR-0008 §4.3 revisit trigger
- PSP-token, payment-state, refund-workflow — entire V2 scope
- Sample-pricing decisions (free vs €5 vs other) — operational/business decision, not aggregate concern; AOT handles via existing Bestellanfrage process

## 3. Aggregate Boundary

Sprint-1 baseline aggregate identity preserved: SampleOrder is its own Aggregate (separate from Catalog Product, separate from CustomerContract Pricing). It owns the order-lifecycle, the buyer-binding, and the NET7-write coordination.

What changed:
- The aggregate no longer has subtypes (no Payment/Request split in V1)
- BuyerIdentity now polymorphic across CustomerAccount and GuestSessionReference

## 4. Internal Structure

### 4.1 SampleOrder (Aggregate Root) — refactored

Fields:
- SampleOrderId
- BuyerIdentity (one of: CustomerAccountReference OR GuestSessionReference)
- LineItems[] (each LineItem = ProductId + Quantity=Muster + optional notes)
- ShippingAddress (sourced from BuyerIdentity at order-submission)
- Status (Sprint-1 state-machine: Requested → InProcessing → Fulfilled → Closed | Cancelled)
- SampleRequestSource (public-catalog | b2b-portal | api-future)
- AntiFraudGuard signals (rate-limit-state, captcha-completion)
- PaymentReference (always null in V1)
- NET7OutboundIdempotencyToken (Sprint-1 ADR-0003 Rule 3)
- AuditTrail (Sprint-1 baseline; extended to log GuestSession-conversion events)

### 4.2 Invariants (continuing INV-numbering from Sprint-1 baseline)

- **INV-007** (state-machine, refactored): SampleOrder.Status state-machine unchanged from Sprint-1, but the entry-event "Requested" can originate from anonymous GuestSession-triggered submission OR authenticated CustomerAccount submission. Single state-machine, two entry-paths.
- **INV-008** (security-boundary, new): PaymentReference MUST be null for all V1 SampleOrders. Adapter rejects any SampleOrder with non-null PaymentReference on inbound; outbound to NET7 omits the field entirely. Enforces ADR-0008.
- **INV-009** (structural, new): if BuyerIdentity is GuestSessionReference, the GuestSession MUST have at minimum: contact email, shipping address. SampleOrder creation fails fast if either is missing.
- **INV-010** (cross-aggregate, new): SampleOrder submission MUST consume Catalog Product visibility check — only Products with `SampleQuantityOption == true` (catalog-sprint-2-extension INV-relevant) AND `PublicVisibilityFlag == true` (for anonymous path) are eligible.
- **INV-011** (behavioural, new): AntiFraudGuard rate-limit triggers reject SampleOrder submission with explicit error-state; the SampleOrder is NOT persisted, NOT pushed to NET7. // ASSUMPTION: rejection telemetry feeds back into ValidationMetrics for capacity planning.
- **INV-012** (state-machine, refactored): GuestSession-triggered SampleOrder MAY trigger CustomerAccount creation in NET7 if AOT-Sales operationally converts the lead — this is a manual workflow at AOT, not an automated aggregate transition. Aggregate does not enforce conversion.

## 5. Web/ERP Ownership Mapping

| Field | Owned by NET7 | Owned by Platform | Notes |
|-------|---------------|-------------------|-------|
| SampleOrderId (platform-generated) | — | ✓ | Platform-issued UUID; NET7 may add its own Auftragsnummer |
| BuyerIdentity (CustomerAccount path) | ✓ (Customer master) | — | NET7 Kundenstamm |
| BuyerIdentity (GuestSession path) | — | ✓ | Platform-only; not in NET7 unless converted |
| ShippingAddress | Hybrid | Hybrid | If CustomerAccount: NET7 master; if GuestSession: platform-only |
| LineItems (ProductId resolution) | ✓ | — | NET7 master |
| Status state-machine | — | ✓ | Platform-side; NET7 has its own Bestellanfrage status that is mapped/projected |
| AntiFraudGuard state | — | ✓ | Platform-only; not exposed to NET7 |
| AuditTrail | — | ✓ | Platform-only |
| NET7OutboundIdempotencyToken | — | ✓ | Per Sprint-1 ADR-0003 Rule 3 |

## 6. Cross-Aggregate References and OQs

### Cross-aggregate references

- → Catalog aggregate: SampleQuantityOption eligibility check (catalog-sprint-2-extension INV-relevant per §2.2 Sample-as-Quantity refactor)
- → Customer-Account aggregate: BuyerIdentity binding when authenticated (customer-account-sprint-2-extension §2.0)
- → GuestSession (new concept in customer-account-sprint-2-extension §2.1): BuyerIdentity binding when anonymous
- → Documents-Compliance: not directly coupled in V1 (sample-shipment regulatory documents handled by NET7 workflow)
- → RFQ module (Sprint-2 V1 scope): no direct aggregate-coupling; both modules consume Catalog independently

### Open Questions (from oq-master-sprint-2-extension)

- OQ-056 (concrete validation thresholds for payment-revisit)
- OQ-057 (AOT-Sales review cadence)
- OQ-058 (anti-fraud measures for anonymous sample-requests)
- OQ-052 (anonymous-inventory-visibility — affects Catalog Product eligibility-display)

## 7. References

- /docs/domain/aggregates/sample.md (Sprint-1 baseline 1.4.2)
- /docs/adr/0002-domain-patterns.md (Pattern 1, Pattern 3 read-only-mirror, Pattern 5 Self-Report Discipline)
- /docs/adr/0003-erp-boundary-rules.md (Rule 3 Idempotency Mandate, Rule 4 Authentication-Credential-Containment)
- /docs/adr/0005-strangler-fig.md (migration umbrella; Sample-Order in Phase-3 scope)
- /docs/adr/0006-modular-monolith.md (`sample` module boundary)
- /docs/adr/0008-payment-strategy.md (DIRECT driver of this refactor; payment-deferred)
- /docs/domain/aggregates/catalog.md (Sample-as-Quantity-Option refactor; see catalog-sprint-2-extension §2.2)
- /docs/domain/aggregates/customer-account.md (GuestSession concept; see customer-account-sprint-2-extension)
- /docs/domain/open-questions-master.md (OQ-052, OQ-056, OQ-057, OQ-058 — see oq-master-sprint-2-extension)
- Visual discovery: Screenshot 4 (current Bestellanfrage workflow), Screenshot 8 ("Muster" quantity button)
