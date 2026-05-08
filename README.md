# AOT — Digital Procurement Platform (NET7)

B2B digital procurement platform for **AOT**, a supplier of organic raw materials for the cosmetics and food industries. The system integrates with the ERP **TopM NET7** to support sample requests, RFQ workflows, procurement workflows, and authenticated customer dashboards. It is **not** a classic webshop — no payment processing, no public checkout, no consumer workflow.

## Status

Sprint 0 — Discovery & Foundation: deliverables merged (tag `sprint-0`). Exit criterion "ERP access and business ownership are clear" pending interview execution. Next: Sprint 1 — Domain Modeling.

## Repo Layout

```
/
├─ AGENTS.md
├─ CLAUDE.md
├─ ROADMAP.md
├─ README.md
└─ docs/
   ├─ workflow.md
   ├─ business-discovery.md
   ├─ erp-discovery.md
   ├─ procurement-hypotheses.md
   └─ adr/
      └─ 0001-stack-and-deployment.md
```

## Start here

- [`AGENTS.md`](./AGENTS.md) — engineering charter, hard rules, ERP ownership.
- [`ROADMAP.md`](./ROADMAP.md) — sprint plan.
- [`docs/workflow.md`](./docs/workflow.md) — collaboration protocol (bridge pattern).
- [`docs/adr/`](./docs/adr/) — Architectural Decision Records (ADR 0001 = stack & deployment).

## Discovery & Strategy Documents

- [`docs/business-discovery.md`](./docs/business-discovery.md) (DE) — customer interview guide for AOT buyers, product developers, and quality managers.
- [`docs/erp-discovery.md`](./docs/erp-discovery.md) (DE) — TopM/NET7 interview guide for technical and account/commercial discovery.
- [`docs/procurement-hypotheses.md`](./docs/procurement-hypotheses.md) (EN) — working assumptions for Version 1, to be validated against discovery findings and analytics.

## Roadmap

See [`ROADMAP.md`](./ROADMAP.md) for sprint structure and deliverables.
