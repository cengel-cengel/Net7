# ADR 0005 — Strangler Fig Migration Strategy

| Status     | **Accepted**                          |
|------------|---------------------------------------|
| Date       | 2026-05-11                            |
| Decided by | Human (System Architect)              |
| Authors    | Human + Claude (chat) + Claude Code   |

## 1. Context

The current AOT B2B portal is served from a legacy server-rendered application at URLs like `inhouse.top?funktion=produkte&ssid=...` (visual discovery, Screenshots 8/9). Session-IDs in URLs, no SEO-friendly routing, no edge-cache compatibility, no public anonymous access path.

The platform mandate (Sprint-1 Closeout + Discovery NEW FACTS) requires:
- Public SEO discoverability for anonymous users
- Authenticated B2B users see customer-specific prices (NET7-driven)
- One-stop digital experience (website + catalog + portal indistinguishable to user)
- NET7 remains backend-of-record (Sprint-1 ADR-0003 Rule 1)

Two strategic options were considered:
- Retrofit the legacy portal to add public/SEO capabilities
- Build a new experience layer in parallel, migrate traffic incrementally

The legacy portal's session-bound URL pattern, server-rendered HTML coupling to NET7, and lack of edge-cache primitives make retrofit architecturally infeasible at acceptable cost. The Sprint-1 capability-map (Section §3) already identifies the experience layer as a distinct capability separate from ERP.

This ADR codifies the migration strategy. It is the architectural foundation that all subsequent Sprint-2 work assumes.

## 2. Decision

Adopt the **Strangler Fig Pattern** for migrating from the legacy portal to the new unified platform.

### 2.1 NET7 as Backend-of-Record (unchanged)

NET7 remains the System of Record for product master, pricing, inventory, DMS, certifications, customer accounts. No changes to NET7 schema or workflows are part of this migration.

This is the constitutional alignment with Sprint-1 ADR-0003 Rule 1 (ERP as Single Source of Truth). The migration does not invalidate Sprint-1 — it builds on top.

### 2.2 New Experience Layer Built Externally

A new application (modular monolith per ADR-0006) is built **alongside** the legacy portal. It does not modify the legacy code path. It consumes from NET7 through the adapter layer (Sprint-1 Adapter-Topology) and serves public + authenticated traffic with its own rendering, caching, and routing.

### 2.3 Parallel Operation Window

Both systems run in production simultaneously during the migration window. Traffic is split by:
- Public/anonymous traffic: served exclusively by the new platform (legacy cannot serve this anyway)
- Authenticated B2B traffic: incrementally migrated by customer segment

// ASSUMPTION: customer segment migration in tranches (top-N customers first, long tail later) — exact sequence determined by AOT-Sales in Discovery.

### 2.4 Feature-Parity Gate Before Legacy Sunset

The legacy portal is decommissioned only when:
- The new platform reaches functional parity for authenticated B2B workflows
- All customers have been migrated (or have opted to stay until forced sunset)
- A 30-day overlap-monitoring period shows no traffic on the legacy portal

// ASSUMPTION: 30-day overlap is conservative; can be shortened with customer-PM confirmation.

### 2.5 No Big-Bang Replacement

The migration strategy explicitly rejects a single cut-over event. Traffic shifts gradually as new capabilities ship. The legacy portal serves as fallback throughout the window.

### 2.6 Domain Migration Order

Migration order follows public-traffic-priority + risk-minimization:
1. Public catalog (anonymous reads; legacy has no equivalent — net-new capability)
2. Document access (public-scoped documents)
3. Sample-request workflow (per ADR-0008 deferred payment; aligns with current Bestellanfrage logic)
4. Authenticated B2B catalog with customer-specific pricing
5. Authenticated B2B document portal (full scope)
6. Quote/RFQ workflows
7. Customer self-service (account, addresses, order history)

// ASSUMPTION: this order assumes public-tier value-capture is V1 priority. Validated against Sprint-1 Capability-Map §3.a-d.

## 3. Alternatives Considered

### Alt 1 — Retrofit Legacy Portal for Public/SEO

**Approach:** Extend the existing inhouse.top portal with anonymous routes, SEO meta-tags, and a public catalog mode.

**Rejected because:**
- Session-bound URL pattern (`?ssid=...`) is incompatible with SEO canonical URLs
- Server-rendered HTML couples brand templates to ERP logic
- Adding edge-cache primitives requires re-architecting request lifecycle
- Estimated retrofit effort exceeds parallel-build effort
- Risk of destabilizing operational B2B portal during retrofit

### Alt 2 — Big-Bang Replacement

**Approach:** Build the new platform to full feature parity, then cut over all customers in a single event.

**Rejected because:**
- No fallback if new platform has unforeseen issues
- Customer disruption from forced migration without opt-in window
- Pre-migration testing cannot cover all customer-specific workflows
- Conway's Law: small team cannot deliver full B2B parity before public-tier launches

### Alt 3 — Greenfield NET7 Replacement

**Approach:** Build a new platform and migrate away from NET7 entirely.

**Rejected because:**
- Out of scope of the mandate (NET7 stays as system of record)
- NET7 contains years of operational data, integrations, workflows
- Replacing NET7 is an order-of-magnitude larger initiative than the experience-layer mandate

## 4. Consequences

### 4.1 Positive

- New platform can be built without disturbing operational B2B portal
- Public-tier ships value (SEO, discovery) early, before B2B parity is required
- Risk localized: new-platform failures do not affect existing customers using legacy
- Customer migration can be paced and supported individually
- Sprint-1 NET7 adapter foundation directly reusable (no rework)
- Legacy portal remains as fallback during entire migration window

### 4.2 Negative / Risks

- Maintenance overhead of two production systems during the migration window (V1 duration, estimated 6-12 months)
- Risk of "permanent parallel" if migration loses momentum
- Customer confusion if both portals are publicly accessible without clear messaging
- Data drift risk if any state is duplicated between systems (mitigated by NET7-as-SoT discipline)

### 4.3 Open Questions / Revisit Triggers

- Customer migration timeline acceptable to AOT-Sales (Discovery, see OQ-master)
- Legacy portal sunset criteria — strict (zero traffic) vs pragmatic (90% migrated)
- Fallback policy if new platform has incident during migration (route 100% to legacy?)
- Trigger for revisit: if migration window extends >18 months, revisit strategy

## 5. Approval

Decided by: Human (System Architect, Carlos)
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Foundation ADR.
Status transitions: Accepted; Superseded only via successor ADR naming this one.

## 6. Related Documents

- /AGENTS.md (engineering charter, constitutional)
- /docs/adr/0001-stack-and-deployment.md (style template)
- /docs/adr/0002-domain-patterns.md (Pattern Foundation — Pattern 4 Adapter Profile Set)
- /docs/adr/0003-erp-boundary-rules.md (Rule 1 NET7-as-SoT — direct constitutional alignment)
- /docs/adr/0004-open-question-governance.md (OQ-Lifecycle for migration-related questions)
- /docs/adr/0006-modular-monolith.md (next ADR — internal structure of the new experience layer)
- /docs/adr/0007-content-strategy.md (content ownership split during migration)
- /docs/adr/0008-payment-strategy.md (payment-deferred shapes Phase-3 migration scope)
- /docs/domain/capability-map.md §3 (capability scope for the new experience layer)
- /docs/domain/adapter-topology.md (Sprint-1 P4, visual reference for the integration boundary)
- /docs/domain/open-questions-master.md (Discovery OQs for migration tranches)
- /docs/architecture/strategic-foundation.md §6 ADR-0005 source-discussion (rationale provenance)
