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
