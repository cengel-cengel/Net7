# ROADMAP.md
# AOT DIGITAL PROCUREMENT PLATFORM — MASTER DELIVERY ROADMAP

Project: AOT Digital Procurement Platform
Goal: Build a scalable enterprise B2B procurement platform connected to TopM NET7.

This is **not** a classic webshop. No payment, no consumer checkout.
Business model: sample requests, RFQs, procurement workflows, account-based customer portals.

For the engineering charter see `AGENTS.md`.
For Claude Code operational rules see `CLAUDE.md`.

---

## DELIVERY MODEL — ROLES

### Human (You)
**Role:** System Architect + Product Owner + Delivery Lead

Responsibilities:
- business decisions
- priorities
- stakeholder management
- scope control
- architecture approval
- ERP escalation
- acceptance decisions

### Claude Code (CLI)
**Role:** Senior implementation + architecture analysis engine

Responsibilities:
- repository management, multi-file refactors
- domain analysis, architecture proposals, interface design
- code generation, test generation
- local debugging, lint/type/test runs
- implementation plans

### Claude (chat — web / app)
**Role:** Program architect + QA + risk advisor

Responsibilities:
- challenge assumptions
- architecture review
- delivery planning, sprint structure
- risk management
- ERP strategy advisory
- stakeholder communication
- pre-commit review

---

# SPRINT 0 — DISCOVERY + PROJECT FOUNDATION

**Goal:** Remove uncertainty before coding.

| Role | Responsibilities |
|---|---|
| **Human** | Create repo, contact customer, contact TopM, collect discovery data, clarify scope |
| **Claude Code** | Initialize repository, create folder structure, scaffold docs, generate discovery checklists, identify risks, challenge missing requirements |
| **Claude (chat)** | Review discovery completeness, challenge business assumptions, identify project blockers, prepare interview guides |

**Deliverables:**
- `AGENTS.md`
- `CLAUDE.md`
- `ROADMAP.md`
- `business-discovery.md`
- `erp-discovery.md`

**Exit criteria:** ERP access and business ownership are clear.

---

# SPRINT 1 — DOMAIN MODELING

**Goal:** Understand the business before building software.

| Role | Responsibilities |
|---|---|
| **Human** | Validate product rules, pricing rules, customer types |
| **Claude Code** | Create domain folders, generate domain entities, value objects, bounded contexts, domain language |
| **Claude (chat)** | Challenge business conflicts, identify missing ownership, validate DDD boundaries |

**Deliverables:** Product domain, Customer domain, RFQ domain, Sample domain, Document domain.

**Exit criteria:** Business language is stable and documented.

---

# SPRINT 2 — ERP BOUNDARY DESIGN

**Goal:** Protect the system from ERP chaos.

| Role | Responsibilities |
|---|---|
| **Human** | Validate TopM assumptions, escalate missing interfaces |
| **Claude Code** | Create adapter folders, define ERP contracts, sync interfaces, error handling, retry logic |
| **Claude (chat)** | Review ERP isolation, identify data corruption risks, second-opinion gate on contract design |

**Deliverables:** ERP adapter interfaces, sync strategy, failure strategy, mapping rules.

**Exit criteria:** ERP boundaries isolated and reviewed.

---

# SPRINT 3 — CORE PLATFORM ARCHITECTURE

**Goal:** Build system skeleton.

| Role | Responsibilities |
|---|---|
| **Human** | Approve architecture |
| **Claude Code** | Generate file structure, application layer, infrastructure layer, domain layer, API contracts |
| **Claude (chat)** | Architecture review, scalability review |

**Deliverables:** Clean architecture skeleton, dependency map, folder conventions.

**Exit criteria:** Architecture approved by human.

---

# SPRINT 4 — AUTHENTICATION + CUSTOMER ACCESS

**Goal:** Secure customer access.

| Role | Responsibilities |
|---|---|
| **Human** | Define customer roles |
| **Claude Code** | Implement auth modules, session handling, role guards |
| **Claude (chat)** | Security review, access control review, threat modeling |

**Deliverables:** Login, roles, permissions.

**Exit criteria:** Secure customer access verified.

---

# SPRINT 5 — PRODUCT CATALOG + SEARCH

**Goal:** Build procurement search experience.

| Role | Responsibilities |
|---|---|
| **Human** | Validate filters and search semantics |
| **Claude Code** | Implement product APIs, search engine, filters, caching, UI modules |
| **Claude (chat)** | UX review, procurement flow review |

**Deliverables:** Product search, product detail pages, filters.

**Exit criteria:** Buyers can find products efficiently.

---

# SPRINT 6 — DOCUMENT INTELLIGENCE

**Goal:** Expose technical product documents.

| Role | Responsibilities |
|---|---|
| **Human** | Define required document types and access rules |
| **Claude Code** | Build document APIs, version handling, download services, UI |
| **Claude (chat)** | QA + compliance review |

**Deliverables:** SDS, COA, TDS, certificates.

**Exit criteria:** Documents work reliably with versioning.

---

# SPRINT 7 — SAMPLE REQUEST WORKFLOW

**Goal:** Generate qualified leads.

| Role | Responsibilities |
|---|---|
| **Human** | Define approval rules |
| **Claude Code** | Build forms, validation logic, RFQ objects, notifications |
| **Claude (chat)** | Conversion review, business process review |

**Deliverables:** Sample request engine.

**Exit criteria:** Sample workflow works end-to-end.

---

# SPRINT 8 — RFQ + PROCUREMENT WORKFLOW

**Goal:** Enable real procurement.

| Role | Responsibilities |
|---|---|
| **Human** | Define escalation logic |
| **Claude Code** | Build RFQ APIs, workflow engine, approval states, buyer dashboard |
| **Claude (chat)** | Process optimization review |

**Deliverables:** RFQ management.

**Exit criteria:** Procurement workflow stable.

---

# SPRINT 9 — CUSTOMER DASHBOARD

**Goal:** Self-service platform.

| Role | Responsibilities |
|---|---|
| **Human** | Define dashboard priorities |
| **Claude Code** | Build dashboard UI, orders, invoices, certificates, reorder logic |
| **Claude (chat)** | Buyer UX review |

**Deliverables:** Customer portal.

**Exit criteria:** Buyers self-serve successfully.

---

# SPRINT 10 — SEO + CONTENT ENGINE

**Goal:** Organic growth.

| Role | Responsibilities |
|---|---|
| **Human** | Define SEO priorities |
| **Claude Code** | Build content components, landing pages, schema markup, content architecture |
| **Claude (chat)** | Conversion + SEO review |

**Deliverables:** SEO platform.

**Exit criteria:** Pages indexable and ranking-ready.

---

# SPRINT 11 — TESTING + HARDENING

**Goal:** Remove operational risk.

| Role | Responsibilities |
|---|---|
| **Human** | Coordinate UAT |
| **Claude Code** | Generate unit, integration, edge-case tests; fix bugs |
| **Claude (chat)** | Risk review |

**Deliverables:** Stable release candidate.

**Exit criteria:** Release approved.

---

# SPRINT 12 — GO LIVE + HANDOVER

**Goal:** Transfer ownership safely.

| Role | Responsibilities |
|---|---|
| **Human** | Stakeholder signoff |
| **Claude Code** | Deployment support, generate setup / recovery / onboarding docs |
| **Claude (chat)** | Final architecture review |

**Deliverables:** Production system, handover package.

**Exit criteria:** System operates without you.

---

# FINAL SUCCESS CRITERIA

Success is **not** "the website is online".

Success is:
- ERP stable
- buyers adopt the system
- sales uses workflows
- customer service trusts the platform
- system survives handover

## Sprint 2 — Strategic Foundation + Architecture (2026-05-09 through 2026-05-11)

Sprint 2 established the constitutional architecture for the unified Digital Revenue Platform per the AOT mandate "one platform, demand generation + procurement enablement".

### Sprint-2 Foundation
- `docs/architecture/strategic-foundation.md` — 1255-line strategic deliberation document (14 sections)
- 4 ADRs Accepted: 0005 Strangler-Fig, 0006 Modular Monolith, 0007 Hybrid Content Strategy, 0008 Payment Strategy (Deferred)
- 5 Aggregate Sprint-2 Extensions: Catalog, Pricing, Sample, Customer-Account, Documents
- OQ-Master Sprint-2 Extension — 15 new OQs (OQ-047 to OQ-061) across 6 clusters

### Phase-1 Consolidation
- `docs/strategy/digital-revenue-platform.md` — Constitutional Strategy with 5 Architecture Principles + Curated Hero Portfolio V1 mandate
- `docs/discovery/aot-discovery-phase-0-brief.md` — 3 focused stakeholder sessions for the 4 H-Risk OQs
- Pricing Extension: DISTRIBUTOR / KEY_ACCOUNT / DIGITAL_CUSTOMER category mapping (derived from NET7 PriceClass per INV-018)

### Phase-2 Tech-Stack
- ADR-0009 Tech-Stack (Proposed): Vercel + Railway + Next.js + Postgres, gated on G1-G4 (Hosting, Team-Capability, Cost, Phase-0-Spike)

### Housekeeping
- OQ-namespace conflict resolved: strategic-foundation §12 OQ-numbering relabeled as historical deliberation, hosting-OQ migrated to extension as OQ-062

### Phase-3 Pre-Discovery
- `docs/discovery/hero-portfolio-selection-framework.md` — Operational tool for Discovery Session 1 (Mgmt + Sales); 4 scoring dimensions + workflow + content-investment spec
- ADR-0010 Identity Provider (Proposed): Auth0 V1 with fallbacks (Cognito, Keycloak SaaS), gated on G1-G4
- ADR-0011 Read-Model Architecture (Accepted): Constitutional 5-rule pattern codifying emergent read-model approach across all 5 aggregate extensions
- This ROADMAP update

### Sprint-2 Status
**Discovery-Ready.** Constitutional layer complete; implementation gated on Discovery outcomes.

### Open H-Risk OQs (Discovery-blocking)
- OQ-049 — Hero-Portfolio Curation (reframed from generic Public-Visibility-Filter)
- OQ-051 — DocumentScope Mapping (NET7 B2B-Dokumentenanzeige → 4-tier classification with Default-Fail-Safe)
- OQ-053 — Public List-Price Anchor Strategy
- OQ-062 — Hosting Strategy (ADR-0009 Gate G1)

### Next Phase Triggers
- Discovery Sessions execute (3 stakeholder groups: Mgmt+Sales / Compliance / IT)
- 4 H-Risk OQs closed with Triplet-Closer (Decision + Artifact-Update + Engineering-Story) per ADR-0004 Rule 6
- ADR-0009 + ADR-0010 transition from Proposed to Accepted after Gates G1-G4 pass
- Phase-0 NET7 Adapter Spike begins

### ADR Inventory After Sprint-2 Phase-3

| ADR | Status | Topic |
|-----|--------|-------|
| 0001 | Accepted | Stack & Deployment (Sprint-1) |
| 0002 | Accepted | Domain Patterns (Sprint-1) |
| 0003 | Accepted | ERP Boundary Rules (Sprint-1) |
| 0004 | Accepted | Open-Question Governance (Sprint-1) |
| 0005 | Accepted | Strangler-Fig Migration |
| 0006 | Accepted | Modular Monolith V1 |
| 0007 | Accepted | Hybrid Content Strategy |
| 0008 | Accepted | Payment Strategy (Deferred) |
| 0009 | Proposed | Technology Stack (gated G1-G4) |
| 0010 | Proposed | Identity Provider Auth0 (gated G1-G4) |
| 0011 | Accepted | Read-Model Architecture (constitutional) |
