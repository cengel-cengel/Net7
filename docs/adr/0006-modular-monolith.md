# ADR 0006 — Modular Monolith for V1

| Status     | **Accepted**                          |
|------------|---------------------------------------|
| Date       | 2026-05-11                            |
| Decided by | Human (System Architect)              |
| Authors    | Human + Claude (chat) + Claude Code   |

## 1. Context

The Sprint-1 domain modelling produced 5 aggregate sketches (Catalog, Sample, Customer-Account, Documents-Compliance, Pricing-Contracts) and identified 8 bounded contexts. The strategic-foundation work (Sprint-2 prep) extended this to 10 bounded contexts including Search, Payment, Content/CMS, and Integration.

Sprint-1 ADR-0002 (Domain Patterns) and ADR-0003 (ERP-Boundary-Rules) defined the structural patterns and rules for adapter-driven integration. They did not commit to a runtime deployment architecture (services vs monolith).

Conway's Law: the organization shapes the system. AOT engineering capacity for Sprint-2 is small (estimated 3-5 engineers including external partners). Building 8-10 separately-deployed services with that team size produces a distributed monolith (anti-pattern): all the operational complexity of microservices, none of the team-scaling benefits.

The Strangler Fig migration (ADR-0005) ships the new experience layer alongside the legacy portal. To minimize V1 operational risk and accelerate time-to-value, the new layer must be deployable as a single unit while preserving the option to decompose to services in V2.

This ADR codifies the V1 runtime architecture.

## 2. Decision

V1 of the new experience layer is built as a **Modular Monolith** with strictly-enforced domain module boundaries.

### 2.1 Module Inventory

The V1 modular monolith contains these domain modules:

- **catalog** — Public + B2B product catalog, faceting, listing, detail
- **pricing** — List-price exposure, customer-pricing computation, RFQ-tier routing
- **customer** — Customer accounts, identity bridging to NET7 (per Sprint-1 ADR-0003 Rule 4)
- **documents** — Document scope, version surfacing, public + B2B-scoped delivery
- **sample** — Sample-request workflow (per ADR-0008, no payment integration V1)
- **rfq** — Quote/RFQ submission and tracking (lightweight V1)
- **auth** — Authentication, session management, role checks
- **search** — Search index management, query routing, faceted search

// ASSUMPTION: 8 modules align with Carlos's mandate explicit module list. May refine during Sprint-2.

### 2.2 Module Boundary Discipline

- No cross-module imports of internal types. Each module exposes a **public interface** (TypeScript types, function signatures) that other modules consume.
- Internal types are not exported beyond the module.
- Cross-module communication via:
  - Direct synchronous calls through the public interface (function-call cost only)
  - In-process event bus for asynchronous fan-out (e.g., `ProductPublished` event consumed by `search` module)
- Database schemas are logically separated per module (one schema per module within a single database instance, or one database per module if simple to operate).

### 2.3 Single Deployment Unit

V1 ships as one deployable artifact. One process, one container, one CI/CD pipeline, one set of secrets. Operational simplicity is the explicit goal.

### 2.4 Internal Event Bus

Asynchronous events are dispatched through an in-process event bus (function-call backed, no external broker). This means:
- Zero infrastructure overhead for V1
- Event handlers run in-process, same transaction context if synchronous, separate context if asynchronous
- Failure mode: a panic in a handler does not crash the whole monolith (handlers isolated)

If event volume or complexity grows beyond V1, the in-process bus can be swapped for an external broker (Kafka/NATS/RabbitMQ) without changing module-level code. This is the V2 decomposition path.

### 2.5 Module-to-Service Decomposition Path (V2)

V1 must preserve the **option** to decompose any module into a separately-deployed service in V2. This is achieved by:
- Strict module boundaries (as above)
- All cross-module communication through the public interface (no shortcut access)
- Each module owns its data (no shared tables across modules)
- Event-bus is the only async coupling (replaceable with external broker)

When V2 demands warrant a decomposition (high load on one module, independent scaling, team-ownership separation), a module can be extracted to a service with minimal refactoring of the rest of the system.

### 2.6 Shared Infrastructure (single-process services)

These cross-cutting concerns are shared across modules in V1:
- Logging, metrics, tracing
- Authentication middleware (`auth` module exposes the check, others consume)
- HTTP framework
- Database connection pool
- Configuration

## 3. Alternatives Considered

### Alt 1 — Microservices from V1

**Approach:** Build each module as a separately-deployed service from day 1.

**Rejected because:**
- Team size insufficient (3-5 engineers cannot operate 8 services)
- Operational complexity (service discovery, distributed tracing, deployment orchestration) without commensurate scaling benefit
- Bounded contexts not yet validated by production traffic — premature decomposition is irreversible
- Each new module's interface evolves; refactoring across service boundaries is painful

### Alt 2 — Single Monolith Without Module Boundaries

**Approach:** Build a single application without strict module discipline. Let domain boundaries emerge organically.

**Rejected because:**
- Sprint-1 already identified clear domain boundaries — discarding them is regression
- Code-bases without enforced boundaries devolve into "big ball of mud"
- V2 decomposition becomes prohibitively expensive
- Cross-cutting changes (e.g., changing pricing logic) leak into unrelated areas

### Alt 3 — Distributed Monolith

**Approach:** Multiple services, but tightly coupled (shared databases, synchronous cross-service calls, shared types).

**Rejected because:**
- Worst of both worlds: operational complexity of services + coupling of monolith
- Recognized industry anti-pattern
- Implicit risk if Alt-1 (microservices) is rushed without proper boundary design

## 4. Consequences

### 4.1 Positive

- Single deployment artifact: simple CI/CD, simple operations
- Same-process function-calls between modules (low latency, easy debugging)
- Module boundaries enforced by code review and conventions (not infrastructure)
- V2 decomposition path preserved; no architectural lock-in
- Sprint-1 domain patterns (ADR-0002) map directly to module boundaries
- Aligns with team-size reality without compromising future scalability

### 4.2 Negative / Risks

- Module boundaries are convention-enforced, not infrastructure-enforced. Discipline degrades over time without active code review.
- A bug in one module can affect the whole process (no fault isolation between modules)
- Scaling is whole-monolith-only in V1 (can't scale search-module independently)
- Shared database schema requires coordinated migrations
- // ASSUMPTION: tooling and linting can enforce module boundaries in practice (TypeScript `package.json` scopes, import-restriction-lints, or equivalent). Validated during V1 implementation.

### 4.3 Open Questions / Revisit Triggers

- Database isolation: one DB with schemas per module, or one DB per module? // ASSUMPTION: start single-DB with per-module schemas; revisit if write-load justifies separation.
- Module boundary violations: how detected, how blocked? Static analysis tooling required.
- Trigger for decomposition: when does V2 microservice extraction make sense per module?
- Test strategy: integration tests across modules vs module-isolated tests.
- ADR-0006 revisit after V1 first stable release (post-Phase-1 in ADR-0005 phasing).

## 5. Approval

Decided by: Human (System Architect, Carlos)
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Foundation ADR.
Status transitions: Accepted; module-list amendments allowed via successor ADRs; full Superseded only via successor naming this one.

## 6. Related Documents

- /AGENTS.md (engineering charter, modular architecture mandate)
- /docs/adr/0001-stack-and-deployment.md (style template)
- /docs/adr/0002-domain-patterns.md (Pattern 1 7-H2 template, Pattern 4 Adapter Profile Set — module boundaries align with these patterns)
- /docs/adr/0003-erp-boundary-rules.md (Rule 1 NET7-as-SoT; Rule 3 Idempotency Mandate applies to all modules)
- /docs/adr/0004-open-question-governance.md (governance for module-evolution OQs)
- /docs/adr/0005-strangler-fig.md (predecessor in this Sprint-2 ADR-sequence; defines what is being built)
- /docs/adr/0007-content-strategy.md (content module boundary — implications for catalog and CMS V2)
- /docs/adr/0008-payment-strategy.md (payment-module absent V1; module-list V2 will add it if validated)
- /docs/domain/bounded-contexts.md (Sprint-1 BC inventory — modules align with BC boundaries)
- /docs/domain/capability-map.md (capability-to-module mapping)
- /docs/domain/adapter-topology.md (NET7-adapter is cross-module infrastructure)
- /docs/architecture/strategic-foundation.md §11 (Conway's-Law analysis that motivated this ADR)
