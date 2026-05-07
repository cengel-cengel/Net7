# AGENTS.md
# NET7 PROCUREMENT PLATFORM — AI ENGINEERING OPERATING SYSTEM

This file is the engineering charter. It is tool-agnostic and applies to every AI tool used in this project.
Tool-specific operational rules live in `CLAUDE.md`.
Delivery plan and sprint structure live in `ROADMAP.md`.

---

## PROJECT CONTEXT

Company: **AOT**

Business: B2B supplier for organic raw materials
- cosmetic ingredients
- food ingredients

ERP: **TopM NET7**

**IMPORTANT — this is NOT a classic ecommerce shop.**

There is:
- NO payment processing
- NO public checkout
- NO consumer workflow

The platform supports:
- sample requests
- RFQ workflows
- procurement workflows
- account-based ordering
- authenticated customer dashboards

---

## PRIMARY GOAL

Build a scalable enterprise-grade digital procurement platform.

Optimize for:
- reduced procurement friction
- faster RFQ workflows
- faster sample conversion
- SEO visibility
- maintainability
- ERP safety
- handover readiness

---

## USER PERSONAS

Primary:
- Strategic procurement buyers

Secondary:
- Product developers
- Quality managers

---

## ERP OWNERSHIP RULES

TopM NET7 is the **single source of truth** for:
- products
- prices
- customer contracts
- stock
- logistics
- batch traceability
- documents
- order history

The web platform owns:
- UX
- SEO
- content
- sessions
- dashboards
- workflows

**Never** duplicate ERP business logic.
**Never** move ERP pricing logic into frontend code.
**Never** bypass ERP adapters.

---

## ENGINEERING PRINCIPLES

Always optimize for:
1. maintainability
2. explicit architecture
3. documentation
4. modularity
5. testability
6. handover readiness
7. ERP isolation

Never optimize for:
- short-term hacks
- speed over quality
- tightly coupled code
- implicit logic
- duplicated business rules

---

## ARCHITECTURE RULES

Mandatory:
- Clean Architecture
- Domain Driven Design
- API-first
- Adapter Pattern
- strongly typed interfaces
- modular services
- separation of concerns

Forbidden:
- business logic in UI
- direct ERP access from frontend
- direct DB access from components
- shared mutable global state
- monolithic service layers

---

## ERP INTEGRATION RULES

All ERP communication MUST go through isolated adapter interfaces.

ERP integrations MUST:
- support retries
- support failure recovery
- be idempotent
- be logged
- support async workflows

Never assume ERP availability.
All ERP failures must degrade gracefully.

---

## DEVELOPMENT WORKFLOW

### Step 1 — Understand
Before coding:
- identify assumptions
- identify missing information
- identify risks
- identify ERP boundaries
- identify ownership

Challenge unclear requirements **before** implementation.

### Step 2 — Plan
Before implementation, generate:
- architecture proposal
- data flow
- service boundaries
- folder structure
- interface definitions
- dependency map

**Do not start coding immediately.**

### Step 3 — Implement
- small isolated modules
- small commits
- explicit interfaces
- explicit typing
- isolated responsibilities

Never generate huge monolithic implementations.
Never generate entire systems in one step.

### Step 4 — Verify
- run tests
- validate interfaces
- validate edge cases
- validate error handling
- validate ERP safety

### Step 5 — Document
- technical explanation
- tradeoff explanation
- setup instructions
- documentation comments
- API documentation

---

## REVIEW CHECKLIST (before every commit)

### Architecture
- Is ERP isolated?
- Are responsibilities separated?
- Is logic modular?

### Security
- Any exposed secrets?
- Unsafe input handling?
- Auth issues?

### Maintainability
- Clear naming?
- Understandable logic?
- Explicit interfaces?

### Scalability
- Hidden coupling?
- State issues?
- Future bottlenecks?

### ERP Safety
- Could ERP data be corrupted?
- Are retries safe?
- Is sync idempotent?

### Handover
- Can another team understand this?
- Is setup reproducible?
- Is documentation sufficient?

---

## CODING STYLE

Requirements:
- TypeScript strict mode
- strongly typed contracts
- reusable components
- explicit error handling
- no magic strings
- no hidden side effects

Prefer:
- composition over inheritance
- pure functions
- dependency injection
- feature isolation

---

## FOLDER STRUCTURE PRINCIPLES

Separate:
- domain
- infrastructure
- application
- UI
- ERP adapters
- API contracts

Never mix:
- ERP logic with UI
- infrastructure with domain logic

(Concrete folder layout: see `CLAUDE.md` §4.)

---

## OUTPUT FORMAT RULES

Always answer in:
- markdown
- structured sections
- code blocks
- tables when useful

Always include:
- reasoning
- tradeoffs
- assumptions
- risks

---

## TASK EXECUTION RULES

For non-trivial tasks:
1. analyze
2. plan
3. propose architecture
4. wait for confirmation if major architecture changes exist
5. implement incrementally

If uncertain: **ask questions first.**

Never hallucinate ERP capabilities.
Never invent APIs without explicitly marking assumptions.

---

## GIT & COMMIT RULES

Before commit:
- run lint
- run tests
- validate types
- review architecture (use checklist above)

Commit style: `<type>(<scope>): <short description>`

Examples:
- `feat(product-search): add CAS search adapter`
- `feat(rfq): implement sample request validation`
- `fix(erp-sync): handle retry timeout`

---

## DOCUMENTATION RULES

Every major module MUST contain:
- purpose
- ownership
- dependencies
- risks
- extension points

Every adapter MUST document:
- ERP assumptions
- retry behavior
- failure behavior
- sync strategy

---

## PERFORMANCE RULES

Optimize for:
1. reliability first
2. maintainability second
3. performance third

Avoid premature optimization.
Use caching only with explicit ownership and invalidation strategy.

---

## SECURITY RULES

Never expose:
- ERP credentials
- internal endpoints
- sensitive customer data

Always validate:
- input
- authentication
- authorization

Never trust frontend input.

---

## TOOLING & ROLES

This project is delivered with the following tools and role boundaries.

### Tools

| Tool | Role | Operating doc |
|---|---|---|
| **Human** | System Architect, Product Owner, Delivery Lead | this file (decision authority) |
| **Claude Code** (CLI) | Implementation engine: code, refactoring, tests, repo ops, architecture proposals | `CLAUDE.md` |
| **Claude** (web/app chat) | Program architect, QA, risk advisor, sprint planning, pre-commit review | this file |

### Role boundaries
- All AI tools follow the rules in this file.
- Claude Code has filesystem and repo access. It executes.
- Claude (chat) has no filesystem access. It plans, reviews, challenges.
- The human is the only decision authority for business rules, architecture sign-off, ERP escalation, and acceptance.

### Single-vendor review risk
Both Claude Code and Claude chat run on the same model family. This removes the vendor-diversity that a ChatGPT + Codex split would provide.

Mitigations:
- The **human** is the independent review gate. AI agreement does not equal correctness.
- Claude (chat) is explicitly instructed to **challenge** Claude Code output, not confirm it.
- For high-stakes architectural decisions (ERP boundary design, auth model, sync strategy, pricing flows, security-sensitive code), an external second opinion (different vendor or human expert) is required before merge.

### AI collaboration rules

AI tools are **not** the system architect.
AI tools are the implementation and analysis engine.

The human decides:
- business rules
- architecture boundaries
- ownership
- priorities

The AI's role:
- challenge assumptions
- identify risks
- implement safely
- document clearly
- improve maintainability

---

## MOST IMPORTANT RULE

**Do not optimize for fast demo generation.**

Optimize for:
- enterprise stability
- maintainability
- ERP safety
- future handover
- long-term scalability
