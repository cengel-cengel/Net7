# ADR 0008 — Payment Strategy (Deferred)

| Status     | **Accepted**                          |
|------------|---------------------------------------|
| Date       | 2026-05-11                            |
| Decided by | Human (System Architect)              |
| Authors    | Human + Claude (chat) + Claude Code   |

## 1. Context

The platform mandate (Sprint-1 Closeout + Discovery NEW FACTS) mentions online payment for anonymous sample orders as a desired capability. The strategic-foundation analysis (Sprint-2 prep, §1.2) challenged this assumption, observing:

- Sample fulfillment is physical (shipping product samples). Sample-handling logistics overhead exists independent of payment.
- Payment introduces operational scope: PSP integration, DSGVO compliance for guest-payment data, fraud handling, refund workflow, dispute management.
- Sample-pricing (visual discovery, Screenshot 4 shows €5,00 sample) is likely below cost — a cost-recovery gesture rather than a profit center.
- Current AOT workflow handles sample-requests as "Bestellanfrage" (request-for-quote-style) without online payment (Screenshot 4 shows "Bestellanfrage abschicken" button, not a payment-checkout).

Visual discovery confirms the current state is **request-based, not payment-based**. The "anonymous payment for samples" is a business hypothesis stated in the mandate, not a validated user need.

Sprint-1 ADR-0003 (ERP-Boundary-Rules) Rule 4 (Authentication-Credential-Containment) implies that payment data must never enter NET7 — payment lives entirely in the new platform tier. Adding a payment module to V1 expands scope and adds PCI-DSS compliance considerations.

This ADR codifies a deliberate deferral: validate demand before building payment infrastructure.

## 2. Decision

**Payment integration is deferred until RFQ/sample-conversion data validates demand.**

### 2.1 V1 Sample-Order Workflow Without Payment

V1 ships sample-order capability **without online payment**:
- Anonymous user can request a sample from the public catalog (Sprint-1 Sample Aggregate 1.4.2 workflow, refactored — see sample-sprint-2-extension.md)
- User provides contact details and shipping address
- Sample-request is pushed to NET7 via the outbound adapter (Sprint-1 ADR-0003 Rule 3 Idempotency Mandate)
- AOT handles the sample-request operationally — confirms, fulfills, invoices, charges via existing channels (current "Bestellanfrage" workflow)
- No PSP integration, no card-data handling, no payment-module in the modular monolith (ADR-0006)

### 2.2 Validation Metrics Before Payment Re-Decision

Payment integration is reconsidered when measurable signals indicate demand justifies the build cost. Candidate validation metrics:

- **Volume threshold**: anonymous sample-requests reach a sustained monthly rate (e.g., consistently >X/month for 3 consecutive months) where manual operational overhead becomes the bottleneck
- **Conversion rate**: meaningful fraction of anonymous sample-requests convert to authenticated B2B accounts or to commercial orders (validating that anonymous-access drives pipeline)
- **AOT-Sales operational feedback**: sample-handling becomes a significant burden on Sales/Innendienst capacity, or sample-fraud (repeated requests, non-genuine inquiries) becomes a real cost

// ASSUMPTION: specific volume thresholds (e.g., "50 samples/month") cannot be set without baseline data. Sprint-2 Phase-0 establishes the measurement infrastructure; V1 ship establishes the baseline; quarterly review establishes whether the threshold has been crossed.

### 2.3 Reversibility — Payment-Ready Architecture

V1 architecture remains payment-ready without including payment:
- Sample Aggregate boundary (Sprint-1 1.4.2) accommodates a `PaymentReference` field that is null in V1
- Sample-Order workflow has explicit state-transitions where a payment-step can be inserted in V2 without restructuring
- Modular monolith (ADR-0006) reserves the option to introduce a `payment` module without affecting other modules

This means the V2 payment introduction is an additive change, not a restructure. The deferred decision does not become a structural lock-in.

### 2.4 Constitutional Rules That Survive Deferral

Independent of the deferral, the following rules apply when payment is eventually introduced:

- **Payment data NEVER persisted in NET7** (Sprint-1 ADR-0003 Rule 4 derivative). PSP-tokens, card data, and transaction details live exclusively in the new platform tier.
- **PCI-DSS scope minimization**: PSP-managed card collection (e.g., hosted fields, redirected checkout) — no raw card data touches AOT systems
- **DSGVO compliance for guest-payment data**: data minimization, retention policy, customer rights documented
- **Refund workflow defined**: not buried in PSP-dashboard-only

These rules are pre-committed; only the timing of implementation is deferred.

### 2.5 RFQ as the V1 Lead-Capture Path

The platform's V1 commercial-pipeline mechanism is the RFQ (Request-For-Quote, see Sprint-1 strategic-foundation §3 RFQ Aggregate). Anonymous + authenticated users alike can submit RFQs for commercial-volume products (e.g., the "A.Anfrage" tiers in Screenshot 6 pricing-staffel). This validates the broader hypothesis that anonymous-access drives B2B pipeline without requiring payment-enabled checkout.

// ASSUMPTION: RFQ submission is a Sprint-2 V1 deliverable, as listed in the ADR-0006 module inventory (`rfq` module). Detailed RFQ workflow design is out of scope of this ADR.

## 3. Alternatives Considered

### Alt 1 — Stripe (or alternative PSP) Integration in V1

**Approach:** Build payment integration as part of V1 sample-order workflow. Use Stripe Elements for PCI-compliant card collection. Wire webhooks to fulfillment trigger.

**Rejected because:**
- Business demand unvalidated — building payment for hypothetical volume is premature optimization
- PSP fees (~2-3% per transaction), monthly tooling cost, and DSGVO/PCI compliance work add real cost and scope
- V1 critical path (Strangler-Fig migration per ADR-0005) does not benefit from payment integration
- Removing scope from V1 accelerates time-to-value for the public-catalog and B2B-migration goals

### Alt 2 — Mollie / PayPal / Klarna

**Approach:** Same as Alt 1 but with a different PSP.

**Rejected because:**
- Same root rejection as Alt 1 (deferral not vendor selection)
- Vendor choice becomes meaningful only when payment is decided in; current ADR scope is not vendor selection

### Alt 3 — Subscription / Account-Credit Model

**Approach:** Customers prepay an account credit; sample-orders deduct from the credit. No per-transaction card collection.

**Rejected because:**
- Requires authenticated account; eliminates the anonymous-sample use case the mandate motivates
- Operational complexity (credit reconciliation, refund handling) without solving the validation question
- Discovery-blocked: no evidence customers want this

### Alt 4 — No Anonymous Sample-Order At All

**Approach:** Require authentication before any sample-request; fully bypass the anonymous-payment question.

**Rejected because:**
- Conflicts with the mandate's anonymous-discoverability goal
- Public catalog SEO-traffic that cannot convert to anything (not even sample-request) has reduced commercial value
- Anonymous sample-request without payment (Alt 0 — chosen) preserves the value

## 4. Consequences

### 4.1 Positive

- V1 scope reduced; engineering capacity refocused on Strangler-Fig migration and modular-monolith foundation
- Demand validated before infrastructure investment
- PSP-vendor decision deferred until evidence supports an informed selection (not gut-feel)
- PCI-DSS scope not expanded in V1
- DSGVO assessment for guest-payment-data deferred (still required at V2 entry, but not blocking V1)
- Sample Aggregate (Sprint-1 1.4.2) simpler in V1 — no PaymentBasedSample subtype, just one unified workflow
- AOT operational workflow stays continuous (current Bestellanfrage handling extends to anonymous samples)

### 4.2 Negative / Risks

- Lower friction-free checkout than competitors who offer payment-enabled sample-orders (specific commerce-UX disadvantage)
- Sample-handling operational overhead absorbed by AOT-Sales/Innendienst — must be monitored for capacity strain
- "Sample fraud" risk: repeated non-genuine requests cost shipping + product. Mitigated by: contact-detail collection, anti-spam, possible rate-limiting per IP/email (engineering measure, not payment)
- Validation metric ambiguity: "when is demand validated" needs concrete threshold setting in Sprint-2 Phase-0
- Re-decision overhead: when payment is eventually built, it's a meaningful V2 increment

### 4.3 Open Questions / Revisit Triggers

- Concrete validation-metric thresholds (volume, conversion, AOT-Sales feedback) — Sprint-2 Phase-0 task
- Anti-fraud measures for anonymous sample-requests in V1 (rate-limiting, captcha, contact-validation) — engineering scope
- AOT-Sales review cadence for sample-volume trends — quarterly review proposed
- Trigger for revisit: any of the validation metrics crossed, or strategic competitive pressure (e.g., competitor launches payment-enabled samples), or AOT-Sales escalation on operational burden
- DSGVO advance-prep: even with deferral, the V2 payment-integration DSGVO assessment is a known future task

## 5. Approval

Decided by: Human (System Architect, Carlos)
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Foundation ADR.
Status transitions: Accepted; revisit triggers in §4.3 produce a successor ADR when met. Until revisit, V1 ships without payment.

## 6. Related Documents

- /AGENTS.md (engineering charter, scope discipline)
- /docs/adr/0001-stack-and-deployment.md (style template)
- /docs/adr/0002-domain-patterns.md (Pattern 5 Self-Report Discipline — applied to deferral framing)
- /docs/adr/0003-erp-boundary-rules.md (Rule 4 Authentication-Credential-Containment — payment-data exclusion derivative; Rule 3 Idempotency Mandate for sample-order push)
- /docs/adr/0004-open-question-governance.md (governance for payment-readiness OQs)
- /docs/adr/0005-strangler-fig.md (migration umbrella — payment-deferral aligns with Phase-3 minimum-viable scope)
- /docs/adr/0006-modular-monolith.md (no `payment` module in V1 module-list; reservation for V2)
- /docs/adr/0007-content-strategy.md (parallel deferral pattern — both ADRs validate readiness before V2 commitment)
- /docs/domain/aggregates/sample.md (Sample Aggregate — V1 workflow refactored without payment subtype; see sample-sprint-2-extension.md)
- /docs/domain/open-questions-master.md (Discovery OQs for payment-readiness — see oq-master-sprint-2-extension.md OQ-payment cluster)
- /docs/architecture/strategic-foundation.md §1.2 (challenge analysis that motivated this deferral)
- Visual discovery: Screenshot 4 (current "Bestellanfrage abschicken" workflow — V1 baseline)
