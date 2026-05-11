# ADR 0011 — Read-Model Architecture (Constitutional Pattern)

| Status     | **Accepted**                          |
|------------|---------------------------------------|
| Date       | 2026-05-11                            |
| Decided by | Human (System Architect)              |
| Authors    | Human + Claude (chat) + Claude Code   |

## 1. Context

Sprint-2 Foundation produced 5 aggregate extensions (catalog, pricing, sample, customer-account, documents) that **implicitly reference a Read-Model Pattern** without ever formally codifying it. The pattern emerges in:

- **catalog-sprint-2-extension §2.1** — `LocaleVariant`, `SanitizedNarrativeHtml`, `ListPriceAnchor` are platform-side projections of NET7-sourced data
- **pricing-sprint-2-extension §2.4** — `PublicListPrice` is edge-cacheable projection; `CustomerPrice` is real-time with short-TTL cache (constitutional, INV-014)
- **documents-sprint-2-extension §2.2** — `PublicDocumentProjection` with `DocumentScope` derived classification + Default-Fail-Safe rule
- **customer-account-sprint-2-extension §2.1** — `IdentityBridgeMapping` is platform-side projection of NET7 customer-records
- **sample-sprint-2-extension §2.0** — outbound writes to NET7 via queued idempotent adapter (the inverse complement to read-models)

This pattern was originally proposed in `strategic-foundation.md §7` as a draft "ADR-0005 Read-Model Architecture" but was not adopted in the final Sprint-2 Foundation (Carlos selected 4 ADRs without this one). The pattern then **emerged anyway** through the aggregate extensions — making the constitutional codification overdue.

**Why now (Phase 3):** Phase-0 implementation will produce concrete projections, sync mechanisms, and cache-eviction strategies. Without this ADR, those decisions risk inconsistency across modules. Codifying the pattern explicitly ensures the `catalog` module's projection logic, the `pricing` module's caching strategy, and the `documents` module's scope-derivation all follow the same constitutional rules.

This ADR is **declarative**, not prescriptive — it documents what is already decided implicitly, makes it explicit, and binds future implementation to it.

## 2. Decision

The platform adopts the **Read-Model Architecture Pattern** as a constitutional rule with five sub-rules.

### 2.1 Read-Model Pattern (visual)

See /docs/architecture/read-model-flow.md for the full Mermaid diagram (deferred to separate file to avoid markdown-nesting fragility; can be added in follow-up commit).

Summary flow:
NET7 (System of Record) → NET7 Adapters → in-process Event Bus → module-local Read Models → Public/B2B APIs. Inverse direction: B2B writes → Outbound Queue → NET7 Adapters → NET7. Customer-specific pricing may bypass Read Model for short-TTL real-time calls.

### 2.2 Rule R1 — NET7 is Never Directly Read by Public API

Public-tier API (anonymous routes per ADR-0005 Phase 1) MUST NOT call NET7. All public reads go through module-local read models. This protects NET7 from:
- Crawler-burst traffic
- Cache-miss amplification under load
- Schema-coupling to public-tier response structures
- DSGVO/security-perimeter expansion of NET7 to public exposure

**Violation example:** a public catalog endpoint calling NET7 pricing API directly to render a product page. Forbidden.

### 2.3 Rule R2 — Customer-Specific Reads MAY Hit NET7 with Short-TTL Cache

Authenticated B2B reads that require **real-time per-customer data** (notably: customer-specific pricing per pricing-sprint-2-extension §2.4) MAY call NET7 in real-time, gated by:
- Short-TTL in-memory cache per (customer, product, quantity) tuple
- Fallback to PublicListPrice + banner if NET7 unavailable (per Sprint-1 ADR-0003 Rule 5)
- Never edge-cached (constitutional, pricing-extension INV-014)

The TTL window is module-dependent. Pricing module suggests 5-60 minutes. Other modules requiring real-time customer data must define their TTL with same discipline.

### 2.4 Rule R3 — Read Models are Eventually Consistent with NET7

Read models are **never** synchronously written by NET7. They are projected from events (preferred) or polled (fallback). This produces eventual consistency.

**Sync targets:**
- Critical fields (price, availability, document scope): <60 seconds lag (p95)
- Less-critical fields (descriptions, narrative content): <5 minutes lag (p95)
- Bulk-updates (rare schema changes): <1 hour lag acceptable

Lag must be **monitored and alerted**. Sync lag >2x target = incident.

### 2.5 Rule R4 — Writes to NET7 are Queued, Idempotent, and Outbound-Only

Per Sprint-1 ADR-0003 Rule 3 (Idempotency Mandate) — all NET7 writes from the platform (sample orders, customer profile changes, RFQ-submissions) go through:
- Outbound queue (decouples write from request-cycle)
- Idempotency-token discipline (retry-safe)
- Backpressure-aware (NET7 capacity protected)

The inverse of read-models: the platform receives **events** from NET7 (or polls), and **emits commands** to NET7 (queued).

### 2.6 Rule R5 — Each Module Owns Its Read Models

Per ADR-0006 Modular Monolith §2.2 (module boundary discipline) — each module owns the read models it consumes. No cross-module read-model access except via module's public interface.

**Cross-module data flow:**
- Catalog module owns its catalog read model
- Pricing module owns PublicListPrice projection
- Catalog module product-detail endpoint asks Pricing module for the price (via Pricing module public API), not by reading Pricing data directly

This rule enables V2 decomposition: if Pricing module becomes a service, the interface remains stable.

### 2.7 Read Model Categories

| Category | Purpose | Edge-cacheable? | TTL/Refresh |
|----------|---------|------------------|-------------|
| Anonymous-public projection (e.g., catalog, public docs, list price) | Serve high-volume anonymous traffic | Yes | Refresh on event; serve from cache |
| B2B-authenticated projection (e.g., per-customer document scope) | Serve authenticated reads | Conditional (per-user invalidation) | Refresh on event |
| Real-time customer-specific (e.g., per-customer pricing) | Reflect current NET7 state | NEVER edge-cached | Short-TTL in-memory only (5-60 min) |
| Internal-only projection (e.g., audit trails, analytics) | Platform operations | N/A | Eventually consistent, no SLA |
| Downstream projections (e.g., search index) | Optimized for query pattern | Per downstream system | Event-subscribed from upstream RM |

### 2.8 Sync Mechanisms (in priority order)

1. **Event-driven** (preferred): NET7 emits change events via webhook/queue → adapter → in-process event bus → projection updates
2. **Polling with timestamps** (fallback): adapter polls NET7 at interval, fetches changes since last-poll-timestamp
3. **Full diff** (worst-case): nightly comparison + sync. Acceptable only for low-velocity entities.

The chosen mechanism per data-domain depends on NET7 capabilities (OQ-047 / OQ-048 from strategic-foundation §12 historical deliberation) — to be confirmed in TopM Discovery.

## 3. Pattern Application — Cross-Reference Table

| Module | Read Model | Sync | Cacheability | Reference |
|--------|-----------|------|--------------|-----------|
| catalog | LocaleVariant + SanitizedNarrativeHtml + ListPriceAnchor + visibility-flag | Event-driven from NET7 product changes | Public-cacheable | catalog-sprint-2-extension §2.1 |
| catalog | Hero-Portfolio-Membership | Platform-side decision (not NET7-sourced) | Public-cacheable | Hero-portfolio-selection-framework |
| pricing | PublicListPrice | Event-driven from NET7 ListPriceUpdated | Edge-cacheable, daily | pricing-sprint-2-extension §2.1, INV-017 |
| pricing | CustomerPrice (per customer, product, qty) | Real-time NET7 call | NEVER edge-cached, short-TTL only | pricing-sprint-2-extension §2.2-2.4, INV-014 |
| documents | PublicDocumentProjection | Event-driven from NET7 DMS-changes | Public-cacheable | documents-sprint-2-extension §2.1-2.2 |
| documents | DocumentScope derivation | Computed in adapter with Default-Fail-Safe | Per-version | documents-sprint-2-extension §2.3 |
| customer-account | IdentityBridgeMapping | Event-driven from NET7 customer changes + Auth0 webhook | Internal-only | customer-account-sprint-2-extension §2.1; ADR-0010 §2.3 |
| customer-account | GuestSession | Platform-only, NOT from NET7 | Internal-only | customer-account-sprint-2-extension §3 |
| sample | SampleOrder state | Platform-owned (writes to NET7 outbound) | Internal | sample-sprint-2-extension §4.1 |
| search (downstream) | Product search index | Subscribed to catalog RM updates | Edge-cacheable | strategic-foundation §6.2 Layer 3; ADR-0009 §2.6 defers vendor |

## 4. Alternatives Considered

### Alt 1 — Direct NET7 Reads from Public Tier

**Approach:** Public catalog calls NET7 directly per request.

**Rejected because:**
- NET7 not architected for public-traffic load patterns
- Cannot serve sub-200ms p95 for SEO-grade Core Web Vitals
- Schema-coupling brittle
- DSGVO/security perimeter expansion
- Catastrophic failure mode if NET7 maintenance window or outage

### Alt 2 — Full Data Duplication

**Approach:** Platform holds canonical product/price/customer data; NET7 is one of multiple sinks.

**Rejected because:**
- Violates Sprint-1 ADR-0003 Rule 1 (NET7 = Single Source of Truth)
- Out of scope of mandate
- Operational complexity: two systems-of-truth always drift

### Alt 3 — Synchronous Write-Through Caching

**Rejected because:**
- Cache-miss amplification under load (cold cache = NET7 stampede)
- No fallback if NET7 unavailable mid-request
- Couples cache-coherence problem to request lifecycle

### Alt 4 — CQRS + Event Sourcing (full)

**Rejected for V1:**
- Significant operational complexity for small team
- Event-sourcing benefits not yet justified by use cases
- ADR-0006 Modular Monolith deliberately defers this to V2 if useful

The read-model pattern here is **CQRS-lite**: read models exist, write side via outbound queue to NET7, not event-sourced commands.

## 5. Consequences

### 5.1 Positive

- Public-tier traffic isolated from NET7
- NET7 maintenance windows tolerated (read models serve stale-but-served per ADR-0003 Rule 5)
- Module independence: each module can iterate its read-model schema independently
- V2 decomposition path preserved
- Performance budget achievable

### 5.2 Risks

- **Eventual consistency surfaces to users.** Mitigation: communicate via "Letzte Aktualisierung" timestamps where commercially meaningful.
- **Sync lag monitoring required.** Alert if lag >2x target.
- **Backfill complexity.** On first deploy + on schema changes, need full backfill from NET7. Mitigation: explicit backfill mechanism per module; documented runbook.
- **Event-bus reliability.** In-process bus failure cascades. Mitigation: persistent event log + replay capability; V2 external broker.

### 5.3 Required Phase-0 Engineering

- Concrete event-bus implementation
- Concrete projection-update mechanism (atomic per event)
- Sync-lag instrumentation + dashboards
- Backfill mechanism (per-module)
- Cache-invalidation strategy per cacheable read-model
- Fallback-behavior implementation per Sprint-1 ADR-0003 Rule 5

## 6. Approval

Decided by: Human (System Architect, Carlos)
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Phase-3 deliverable.
Status: **Accepted** — codifies existing emergent pattern; binding on all current and future modules.

## 7. Related Documents

- /AGENTS.md (engineering charter)
- /docs/strategy/digital-revenue-platform.md (Principle 2 NET7-as-SoR-only — served by R1)
- /docs/adr/0003-erp-boundary-rules.md (Rule 1, Rule 3, Rule 5 — implemented by R1/R4/R2)
- /docs/adr/0005-strangler-fig.md (external experience layer mechanism)
- /docs/adr/0006-modular-monolith.md (R5 module boundary; §2.4 event bus)
- /docs/adr/0009-tech-stack.md (Postgres as read-model store)
- /docs/adr/0010-identity-provider.md (Auth0 webhook = R3 sync example)
- /docs/domain/aggregates/catalog-sprint-2-extension.md
- /docs/domain/aggregates/pricing-contracts-sprint-2-extension.md
- /docs/domain/aggregates/documents-compliance-sprint-2-extension.md
- /docs/domain/aggregates/customer-account-sprint-2-extension.md
- /docs/domain/aggregates/sample-sprint-2-extension.md
- /docs/architecture/strategic-foundation.md §6.2, §7
