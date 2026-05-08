# Procurement Hypotheses — AOT Digital Procurement Platform

- **Status:** Working assumptions for Version 1
- **Version:** 1.0
- **Authoring:** AOT Program / Architecture

> **Important:** These are NOT customer-validated insights.
>
> These are procurement-driven design hypotheses based on:
> - B2B strategic procurement best practices
> - ingredient sourcing workflows
> - compliance-heavy raw material industries
> - cosmetics + food procurement behavior
>
> These assumptions MUST be validated later using:
> - usage analytics
> - sample conversion rates
> - RFQ conversion rates
> - document download behavior
> - customer success feedback

---

## 1. Business Assumption

AOT is NOT building a classic webshop.

AOT is building:

"Digital Procurement Platform"

The platform exists to:

- reduce buyer friction
- accelerate sample conversion
- reduce RFQ turnaround time
- reduce document-related sales effort
- improve customer retention
- differentiate against competitors through procurement UX

---

## 2. Primary Buyer Assumption

Primary user:

Strategic Procurement Buyer

Typical goals:

- find approved suppliers faster
- reduce internal approval effort
- reduce procurement risk
- secure compliant documentation
- compare sourcing options faster

Typical frustrations:

- missing documents
- slow supplier responses
- unclear lead times
- no visibility into alternatives
- repeated manual requests

---

## 3. Secondary Personas

### Product Developer

Goals:

- find technically suitable ingredients
- validate functionality quickly
- obtain samples quickly
- access technical data

Needs:

- TDS
- formulation guidance
- sample workflow
- application data

---

### Quality Manager

Goals:

- ensure compliance
- ensure audit readiness
- verify documentation

Needs:

- SDS
- COA
- certificates
- batch traceability
- version history

---

## 4. Procurement Decision Criteria

Hypothesis:

Buyers prioritize:

1. Quality consistency
2. Documentation completeness
3. Lead time reliability
4. Supplier responsiveness
5. Price competitiveness
6. Sustainability / certification

Price alone is NOT the primary decision driver.

---

## 5. Buyer Journey Hypothesis

### Stage 1 — Search

Buyer wants to search by:

- product name
- INCI
- CAS
- botanical name
- application
- certification
- origin
- packaging unit

Failure condition:

If product cannot be found in < 30 seconds,
buyer leaves.

---

### Stage 2 — Qualification

Buyer compares:

- MOQ
- origin
- lead time
- certifications
- application fit
- packaging
- availability

Failure condition:

If comparison is manual,
friction increases.

---

### Stage 3 — Documentation

Buyer wants:

- SDS
- COA
- TDS
- organic certificates
- allergen statements
- kosher / halal
- batch-specific documents

Failure condition:

If documents require email requests,
conversion drops.

---

### Stage 4 — Sample Request

Buyer wants:

one-click sample request.

Required fields:

- sample quantity
- application
- project name
- requested delivery date
- optional comment

Failure condition:

If form is long or unclear,
sample requests drop.

---

### Stage 5 — RFQ

Buyer expects:

Convert sample to RFQ.

Expected behavior:

Sample data should auto-populate RFQ workflow.

Failure condition:

If data must be entered twice,
friction increases.

---

### Stage 6 — Repeat Procurement

Buyer wants:

dashboard with:

- previous samples
- previous RFQs
- order history
- invoices
- certificates
- delivery tracking

Failure condition:

If repeat orders require sales contact,
self-service value drops.

---

## 6. Procurement UX Hypotheses

Must-have features:

### Smart Search

Support:

- INCI
- CAS
- product name
- certification
- application

---

### Product Comparison

Compare:

- origin
- MOQ
- lead time
- certifications
- packaging
- availability

---

### Document Completeness

Show:

"Required documents available"

Example:

8/8 documents available

---

### Availability Confidence

Do NOT show exact stock.

Show:

- Available
- Low availability
- Made to order

---

### Alternatives

If unavailable:

Show:

Comparable alternatives

Never show:

Out of stock only

---

### Saved Lists

Allow:

Project-based product lists

Examples:

- Anti Aging Serum
- Functional Beverage
- Organic Skincare

---

## 7. ERP Assumptions

ERP:

TopM NET7

ERP remains source of truth for:

- products
- prices
- stock
- customer contracts
- batch traceability
- documents
- order history

Web platform owns:

- UX
- SEO
- search
- sessions
- workflows
- dashboards

Never duplicate ERP business logic.

---

## 8. Success Metrics

Version 1 success is measured by:

### Sample Conversion

Target:

increase sample requests

---

### RFQ Speed

Target:

reduce RFQ turnaround time

---

### Sales Efficiency

Target:

reduce manual document requests

---

### Customer Retention

Target:

increase repeat procurement behavior

---

## 9. Version 1 Feature Priorities

Priority 1:

- product search
- product details
- documents
- sample requests

Priority 2:

- RFQ workflow
- dashboard
- reorder

Priority 3:

- personalization
- recommendations
- analytics

---

## 10. Validation Rule

These hypotheses are assumptions.

Do NOT treat as facts.

Validate later using:

- analytics
- heatmaps
- sample conversions
- document downloads
- buyer interviews

---

## References

- [`AGENTS.md`](../AGENTS.md) — Personas, Project Context, ERP Ownership Rules
- [`business-discovery.md`](./business-discovery.md) — discovery interview guide that validates these hypotheses
- [`ROADMAP.md`](../ROADMAP.md) — Sprint structure and feature priorities
