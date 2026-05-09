# Customer Account Aggregate Sketch — AOT Procurement Platform

## 1. Purpose

This document is the DDD aggregate sketch for the **Customer Account** bounded context (`bounded-contexts.md` §4.g). It fixes the aggregate root, the entities and value objects inside the consistency boundary, two state machines (`ContractStatus` at customer level, `UserStatus` at user level), the security boundary around authentication, and the Web/ERP ownership at component level. Direct input for Sprint 2 ERP Boundary Design (especially the user-profile write API and the identity-bridging contract) and Sprint 3 code scaffold (entities, value objects, state machines, security-sensitive credential handling).

All structural choices in this document are **working hypotheses** until validated by the Sprint-0 ERP-discovery stream (`erp-discovery.md`) and the customer-discovery stream (`business-discovery.md`).

This is the third of five aggregate sketches for the P1 contexts. The 7-section structure follows the pattern template established in [`catalog.md`](./catalog.md) (Task 1.4.1); the *state-machine* invariant sub-class and the *write-path adapter* shape come from [`sample.md`](./sample.md) (Task 1.4.2). One pattern extension is introduced here and becomes reusable in subsequent sketches: a **security-boundary** sub-class of invariants for data that must never propagate across the ERP boundary (authentication credentials, MFA secrets, sessions, audit-trail content). Customer Account is also the first **hybrid** aggregate — ERP-defined customer/user master data sits alongside Web-only preferences and security-sensitive credentials.

## 2. Methodology

- **Aggregate** = a cluster of related domain objects treated as a single unit for data changes. One Aggregate Root acts as the only entry point; external code never reaches inside the aggregate by reference.
- **Entity vs Value Object**: an Entity has identity that persists across state changes (e.g., a `User` is the same user even after their role changes from *Tactical Buyer* to *Strategic Buyer*). A Value Object is defined entirely by its attributes (e.g., `UserStatus` is a stamped value within the lifecycle; two users in the same status are interchangeable on that attribute).
- **Web / ERP / Shared / Web-only ownership**: per `AGENTS.md` ERP Ownership Rules, NET7 owns master customer data, contracts, and pricing tiers. *ERP-defined* means the source of truth is NET7; *Shared* means web augments or holds local state alongside the ERP record; *Web-only* means the value lives entirely in the web database with no ERP counterpart. *Web-only security-sensitive* is a strict subset (see security-boundary invariants below).
- **Invariant typology** — five sub-classes: *structural* (cardinality, presence) — must hold at every read; *behavioural* (transition rules) — enforced at write; *state-machine* (legal status transitions) — enforced at every status change, status-history immutable; *cross-aggregate* (depends on another aggregate) — eventually consistent only; *security-boundary* (NEW) — hard rules that some data never crosses a boundary at all (e.g., never propagates to ERP, never logged in plaintext, never serialized over public APIs). Security-boundary invariants are enforced both at write (refuse) and at read (no boundary call exposes the value).
- **Hypothesis marking**: where ERP capability or domain semantics are unverified, the line carries `Hypothesis:` or `Open question:`. The §6 OQ-N register collects these for follow-up.
- **Cross-aggregate references**: aggregates reference each other **by ID only**. No object navigation crosses an aggregate boundary; reads of related data go through repositories or query services on the consumer side.

## 3. Aggregate Boundary

### Aggregate Root

**CustomerAccount** is the Aggregate Root (provisional). Justification: in B2B procurement the contractual entity is the customer organisation, not the individual user. Pricing tier, contract status, billing address, tax status, and the entire commercial relationship are properties of the *Customer*. Users exist only within a customer context — there is no meaningful "platform user" without a customer organisation behind them. Authentication, preferences, and per-user role assignments are therefore modelled as children of the Customer Account.

This is a non-trivial decision; the alternative (User as Root) is discussed below and tracked as `OQ-1`.

### Inside the Aggregate

Components inside the Customer Account aggregate (transactional consistency boundary). Three groups, by ownership:

Customer-level (ERP-defined, 6 components):

- `CustomerAccount` — the root entity; identity stable across customer-level state changes.
- `CustomerId` — value object wrapping the stable identifier from NET7. *Hypothesis: identifier survives mergers and renames (`erp-discovery.md` §3.c).*
- `CompanyName` — value object; the buyer-facing display name.
- `TaxStatus` — value object pairing VAT-ID and country.
- `ContractStatus` — value object stamped onto the customer (state-machine domain: *Pending → Active → Suspended → Active / Closed*).
- `DefaultShippingAddress`, `DefaultBillingAddress` — value objects. Defaults consumed by Sample Lifecycle and Order Management; per-request override permitted at consumer side.

User-level (Shared, 4 components):

- `User` — entity; identity stable across role and status changes. *Hypothesis: child entity inside CustomerAccount (OQ-1); cardinality 1..N.*
- `UserId` — value object. *Hypothesis: NET7-issued vs web-issued open (OQ-4); affects identity-bridging.*
- `UserRole` — value object: *Strategic Buyer / Tactical Buyer / Mixed / Product Developer / Quality Manager*, sourced from `AGENTS.md` Personas.
- `UserStatus` — value object stamped onto the user (state-machine domain: *Active → Inactive → Locked → Active*).

Web-only (3 components, including security-sensitive):

- `UserPreferences` — entity. UX preferences: language, default workspace, notification settings. Web-only persistence; never propagated to NET7.
- `AuthenticationCredential` — entity. **Security-sensitive**: password hash, MFA secrets, recovery tokens, session lifecycle metadata. *Hypothesis: inside Customer Account (OQ-2).* Subject to `INV-5` (security-boundary): never propagates to NET7.

The transactional invariants in §4 (INV-1 … INV-8) all hold inside this boundary, including the two state-machine invariants and the security-boundary invariant.

### Outside the Aggregate

Explicitly **not** inside the Customer Account aggregate:

- `Workspace`, `ProductList` — owned by the **Workspaces / Saved Lists** aggregate; referenced by `WorkspaceReference`. Workspaces are scoped to a customer (and possibly to a user), but the workspace lifecycle and contents have their own consistency rules.
- `SampleRequest`, `SampleStatus`, `DecisionFeedback` — owned by the **Sample Lifecycle** aggregate; referenced by `SampleReference`.
- `RFQRequest`, `Quote`, `QuoteValidity` — owned by the **RFQ Lifecycle** aggregate; referenced by `RFQReference`.
- `Order`, `OrderLine`, `DeliveryTracking` — owned by the **Order Management** aggregate; referenced by `OrderReference`.
- `Document` (account agreements, contract docs, KYC docs) — owned by the **Documents & Compliance** aggregate; referenced by `DocumentReference` per status milestone (e.g., contract signature on *Active*).
- `Price`, `Contract`, `PaymentTerms` — owned by the **Pricing & Contracts** aggregate; the customer-level relationship is declared here (`ContractStatus`), but the contract terms themselves live in their own aggregate.

### Discussion

**Aggregate Root — CustomerAccount (Org-centric) or User (Person-centric)?**
- *Pro CustomerAccount (default)*: contractual entity in B2B procurement is the customer organisation; pricing tier, contract, billing, tax, default addresses are customer-level; multi-user collaboration on samples / RFQs / orders within a customer org is the V1 expectation; authentication is meaningful only relative to a customer.
- *Contra CustomerAccount*: in a multi-tenant future, a single physical person may belong to several customer orgs (e.g., consultant working with multiple AOT clients) — modelling User as Root would simplify that. Lifecycle of a User (deactivate, password reset) does not need the entire customer aggregate locked.
- *Pro User (alternative)*: matches typical SaaS identity-platform patterns; cleaner separation between identity and commercial relationship; SSO / OIDC integration is naturally identity-first.
- **Provisional decision**: CustomerAccount as Root, User as child entity. V1 is single-tenant per `INV-2` (one User belongs to exactly one Customer). Revisit per `OQ-1` if `business-discovery.md` §3.b reveals significant multi-customer-user scenarios; multi-tenant scope is tracked as `OQ-3`.

**AuthenticationCredential — inside Customer Account, or own Security aggregate?**
- *Pro inside (default)*: every authentication is bound to a User inside a CustomerAccount; security-boundary `INV-5` enforces that credentials never reach NET7, which is enough isolation for V1; lifecycle is tightly coupled to User (`INV-8` couples `UserStatus` to authentication availability); single transaction can lock both a user and revoke their session.
- *Contra inside*: authentication has independent lifecycles in mature systems (token rotation, MFA enrolment, password expiry, session management) that may pollute customer-aggregate transactions; a dedicated Security aggregate is the standard pattern for SSO / SAML / OIDC integration; testing and audit are easier when credentials live in their own boundary.
- **Provisional decision**: inside Customer Account for V1. The `AuthenticationCredential` entity is governed by the security-boundary invariant `INV-5` — credential data never propagates to NET7 under any code path. Revisit per `OQ-2` if SSO / SAML integration is brought into V1 (`OQ-7`); if so, the Security aggregate becomes a separate Sprint-1.4 sketch.

**Multi-Customer-User — V1 single-tenant or multi-tenant?**
- *Pro single-tenant (default)*: simpler invariants (`INV-2`), cleaner auth-scoping (one session = one customer context); matches AGENTS.md persona model (Strategic Buyer is a person *at a customer*, not an independent identity); minimises identity-bridging complexity at adapter level; if multi-tenant emerges, can be refactored later.
- *Contra single-tenant*: real B2B reality includes consultants, agencies, parent-/sub-company structures, and procurement teams that span multiple legal entities; modelling forced single-tenant up front may invalidate parts of the data model when multi-tenant arrives.
- **Provisional decision**: single-tenant for V1 (`INV-2`). Multi-tenant scope is `OQ-3`; if `business-discovery.md` interviews reveal substantial multi-customer scenarios, this becomes a v1.5 / v2 architectural shift, not a V1 retrofit. Identity-bridging in `Net7CustomerAdapter` is designed assuming `(WebIdentity → CustomerId)` is 1:1.

## 4. Internal Structure

### Schema

| Name                       | Type          | Owner                  | Notes                                                                                                |
|----------------------------|---------------|------------------------|------------------------------------------------------------------------------------------------------|
| `CustomerAccount`          | Entity (Root) | ERP-defined            | Org-level account; identity is `CustomerId`; root of the aggregate.                                  |
| `CustomerId`               | Value Object  | ERP-defined            | NET7-issued. *Hypothesis: stable across mergers and renames (`erp-discovery.md` §3.c).*              |
| `CompanyName`              | Value Object  | ERP-defined            | Buyer-facing display name.                                                                           |
| `TaxStatus`                | Value Object  | ERP-defined            | VAT-ID + country.                                                                                    |
| `ContractStatus`           | Value Object  | ERP-defined            | State machine: *Pending → Active → Suspended → Active / Closed*.                                     |
| `DefaultShippingAddress`   | Value Object  | ERP-defined            | Default for samples / orders. Per-request override at consumer side (e.g., Sample `DeliveryAddress`). |
| `DefaultBillingAddress`    | Value Object  | ERP-defined            | Default for orders.                                                                                  |
| `User`                     | Entity        | Shared                 | Individual login. *Hypothesis: child entity inside CustomerAccount (OQ-1).* Cardinality 1..N.        |
| `UserId`                   | Value Object  | Shared                 | *Hypothesis: NET7-issued vs web-issued open (OQ-4).* Drives identity-bridging strategy.              |
| `UserRole`                 | Value Object  | Shared                 | *Strategic Buyer / Tactical Buyer / Mixed / Product Developer / Quality Manager*.                    |
| `UserStatus`               | Value Object  | Shared                 | State machine: *Active → Inactive → Locked → Active*.                                                |
| `UserPreferences`          | Entity        | Web                    | Language, default workspace, notification settings. Web-only persistence.                            |
| `AuthenticationCredential` | Entity        | Web (security-sensitive) | Password hash, MFA secrets, recovery tokens, session metadata. Subject to `INV-5` security-boundary. |

### Invariants

Transactional invariants — must hold at every commit of the aggregate:

- **INV-1** (structural). `CustomerAccount` has at least one `User`. A customer without users cannot authenticate or transact.
- **INV-2** (structural). `User` belongs to exactly one `CustomerAccount`. *Hypothesis: V1 single-tenant; multi-tenant scope is `OQ-3`.* Cross-customer user-sharing is not permitted in V1.
- **INV-3** (state-machine). `ContractStatus` transitions follow the customer-level lifecycle: *Pending → Active*, *Active → Suspended*, *Suspended → Active*, *{Active, Suspended} → Closed*. Closed is terminal. Backward transitions other than *Suspended → Active* are prohibited; status-history is append-only.
- **INV-4** (state-machine). `UserStatus` transitions follow the user-level lifecycle: *Active → Inactive*, *Active → Locked*, *Inactive → Active*, *Locked → Active*. Both *Inactive* and *Locked* are reversible, but a *Locked → Active* transition requires an explicit unlock action (out-of-band, captured in the audit log).
- **INV-5** (security-boundary). `AuthenticationCredential` content (password hashes, MFA secrets, recovery tokens) never propagates to NET7. The `Net7CustomerAdapter` write surface explicitly excludes these fields; adapter contract enforces this at compile time. Logs never serialize credential content; serialization rejects credential fields by default.
- **INV-6** (behavioural). ERP-sourced fields (`CustomerId`, `CompanyName`, `TaxStatus`, `ContractStatus`, default addresses, optionally `UserId` per `OQ-4`) are read-only on the web side. Local mutation is a violation; the only legal write paths are *create / update* via the adapter or *status-sync* from NET7.
- **INV-7** (structural). `UserPreferences` belong to exactly one `User`. There is no shared-preferences entity at customer level; preferences are intrinsically per-user.
- **INV-8** (state-machine). A `User` whose `UserStatus` is *Inactive* or *Locked* cannot authenticate. The `AuthenticationCredential` check verifies `UserStatus` before issuing a session; otherwise authentication is refused with no information leaked about the credential check itself.

### Consistency Rules

- **Transactional (inside aggregate)**: customer status transitions, user creation / deactivation / locking, role changes, preference updates, and credential changes commit atomically. INV-1 through INV-8 hold at commit time. Status-history append (for both `ContractStatus` and `UserStatus`) and the corresponding state-machine invariant evaluations occur in the same transaction.
- **Eventual (cross-aggregate)**: `ContractStatus` updates from NET7 are eventually consistent — there will be a delay between contract events on the ERP side and dashboard / authentication-gate reflection on the web. *Hypothesis: lag of < 1 minute acceptable for V1 contract-status sync; lag of < 5 minutes acceptable for user-status sync. Validate via `erp-discovery.md` §3.f.* The `WorkspaceReference` / `SampleReference` / `RFQReference` / `OrderReference` linkages to other aggregates are eventually consistent — a customer can be marked *Closed* before all in-flight references are reconciled.

## 5. Web/ERP Ownership and Adapter Boundary

Per-component ownership statement and adapter mediation. Customer Account is the first **hybrid** aggregate on this platform: ERP-defined customer master sits next to Web-only preferences and security-sensitive credentials. The adapter contract reflects the split.

- `CustomerAccount` — **ERP-defined**. NET7 is source of truth. Web reads through adapter. Mediated by `Net7CustomerAdapter`.
- `CustomerId` — **ERP-defined**. NET7 issues; web treats as opaque stable handle. *Hypothesis: stable across renames / mergers; validate per `erp-discovery.md` §3.c.*
- `CompanyName`, `TaxStatus`, `DefaultShippingAddress`, `DefaultBillingAddress` — **ERP-defined**. Read-through with short-TTL cache. `Net7CustomerAdapter`.
- `ContractStatus` — **ERP-defined**. State-machine authority lives in NET7 contract management. *Hypothesis: web models the same state names but does not transition them locally (OQ-6 sync mechanism).*
- `User` — **Shared**. *Hypothesis: User master may live in NET7 (mirrored, then `UserId` is ERP-defined) or be web-native with NET7 only carrying CustomerId (then `UserId` is web-issued and the adapter performs identity-bridging at session start). Decision in OQ-4.*
- `UserId` — **Shared**. Owner depends on `OQ-4` resolution.
- `UserRole` — **Shared**. *Hypothesis: web maintains the role mapping; ERP may publish a role hint that web translates onto the AGENTS persona vocabulary (OQ-8).*
- `UserStatus` — **Shared**. *Hypothesis: status authority is shared — ERP can suspend (e.g., contract-driven), web can lock (e.g., failed-login). Both writes converge on the same VO.*
- `UserPreferences` — **Web**. Web-only persistence in V1. Never propagates to NET7.
- `AuthenticationCredential` — **Web (security-sensitive)**. Web-only persistence. Subject to `INV-5`: never propagates to NET7 under any code path. Logs never serialize credential content.

### Net7CustomerAdapter

The single mediation layer for Customer Account ↔ NET7 traffic. Hybrid read/write/identity-bridge adapter. Builds on the write-path pattern from `sample.md` (Task 1.4.2).

- **Read**: customer master, `ContractStatus`, user list, default addresses, `TaxStatus`, optional `UserRole` hint.
- **Write-back (hybrid)**: `User` profile updates (`User`, `UserRole`, contact details where relevant). *Hypothesis: NET7 supports user-profile-write API; if not (portal-only per `erp-discovery.md` §3.e), CAP-023 becomes a read-only capability and the adapter degrades to identity-bridging only (OQ-5).* Profile-write is **idempotent** against a client-generated correlation token, mirroring the `sample.md` write pattern. `UserPreferences` and `AuthenticationCredential` are explicitly excluded from any write-back; the adapter contract refuses these fields at compile time per `INV-5` and the `UserPreferences` web-only convention.
- **Status-sync**: `ContractStatus` and `UserStatus` updates from NET7 (push from event/webhook vs scheduled pull, `OQ-6`). Status-sync drives both authentication gating (via `INV-8`) and dashboard reflection.
- **Identity-bridging**: maps web-side authenticated identity to `CustomerId` at session start. Critical for every other authenticated capability that needs a customer scope (CAP-006 pricing, CAP-011 sample request, CAP-018 order placement, etc.). *Hypothesis: bridge is a 1:1 lookup against NET7; multi-tenant scenarios (`OQ-3`) would require multi-step resolution.*
- **Failure mode**: write failures retry with the same correlation token (idempotency); status-sync failures show a *stale-banner* with a `last-synced-at` timestamp; `AuthenticationCredential` continues to operate purely on the web side regardless of ERP availability — a NET7 outage does not lock users out of authenticated read paths that depend only on cached customer master data, though writes that need fresh ERP data will degrade gracefully.

### Sprint-2 Hook

Input for the Sprint 2 ERP Boundary Design adapter contract. Open ERP capabilities to validate before contract sign-off:

- **User-profile write API** (CAP-023) — exists, or is profile management portal-only? Drives `OQ-5` and the `Net7CustomerAdapter.write` surface.
- **Customer-/User-status event stream** — does NET7 emit transition events, or must web poll? (`OQ-6`.)
- **User-Role source** — ERP-defined or web-mapped? Affects role-translation layer in the adapter (`OQ-8`).
- **User-ID strategy** — NET7-issued (User mirrored to ERP) or web-issued (User is web-native with web → ERP identity bridge)? (`OQ-4`.)
- **Stable Customer-ID** — survives mergers, renames, successor accounts? (`erp-discovery.md` §3.c.) Critical for identity-bridging because every authenticated capability resolves through `CustomerId`.

## 6. Cross-Aggregate Relationships and Open Architecture Questions

### Cross-Aggregate ID-References

Customer Account references other aggregates **by identifier only**. No object navigation crosses these boundaries; consumers fetch the related data via the target aggregate's repository.

- **`WorkspaceReference`** → Workspaces / Saved Lists aggregate. A user can own multiple workspaces; cardinality and visibility (per-user vs per-customer-org) are open per `OQ-1` of `Workspaces` aggregate (Task 1.4.x).
- **`SampleReference`** → Sample Lifecycle aggregate. Aggregated views on the customer dashboard (samples by users in this customer) are rendered by querying Sample with `(CustomerId, optional UserId)`.
- **`RFQReference`** → RFQ Lifecycle aggregate. Same pattern as samples for dashboard rendering.
- **`OrderReference`** → Order Management aggregate. Same pattern.
- **`DocumentReference`** → Documents & Compliance aggregate. Per-status documents (e.g., contract signature on `ContractStatus = Active`, KYC documents on `Pending → Active`) are referenced from the customer's status history.

### Open Architecture Questions

These questions are the primary input to Sprint 2 ERP Boundary Design and may also be answered by `business-discovery.md` / `erp-discovery.md` interviews.

- **OQ-1** *(critical, Aggregate-Root)*. `User` as child entity inside `CustomerAccount` (current default) or own aggregate? Triggers for re-evaluation: substantial multi-customer-user scenarios; SSO / OIDC integration becomes V1; user-lifecycle complexity grows beyond the current state machine. Validate via `business-discovery.md` §3.b on multi-contact handling.
- **OQ-2**. `AuthenticationCredential` — inside Customer Account (current default) or own Security aggregate? Trigger: SSO / SAML / OIDC integration; MFA enrolment becomes a multi-step flow with its own state machine; credential rotation policies require independent transactions. Couples to `OQ-7`.
- **OQ-3**. Multi-tenant scope — V1 single-tenant (`INV-2`) or multi-tenant from V1? Trigger: discovery reveals substantial multi-customer-user scenarios (consultants, agencies, parent / sub-company structures). Affects `INV-2`, `Net7CustomerAdapter` identity-bridging, and session-scoping across the entire platform.
- **OQ-4**. `UserId` source — NET7-issued (User mirrored to ERP) or web-issued (User is web-native with web → ERP identity bridge at session start)? Drives `Net7CustomerAdapter.identity-bridging` strategy and the User write-path. Validate via `erp-discovery.md` §3.b, §3.c.
- **OQ-5**. User-profile updates — read-only ERP-mirror (web cannot write profile) or write-back via adapter (CAP-023)? If portal-only, CAP-023 degrades to a passive read with an external link to the TopM portal. Validate via `erp-discovery.md` §3.e.
- **OQ-6**. `ContractStatus` and `UserStatus` synchronisation — push from NET7 (event / webhook) vs scheduled pull on the web side? Drives both freshness UX and adapter complexity. Affects authentication gating via `INV-8`. Validate via `erp-discovery.md` §3.f.
- **OQ-7**. SSO / SAML / OIDC integration — V1 native or post-V1? V1-native pulls `AuthenticationCredential` toward its own Security aggregate (`OQ-2`); post-V1 keeps the credential entity inside Customer Account. Validate via `business-discovery.md` §3.i (Tech-Readiness) on customer expectations.
- **OQ-8**. `UserRole` mapping — per-User intrinsic role (the same person carries the same role everywhere) or per-CustomerAccount-User-mapping (the same person can be Strategic Buyer at Customer A and Tactical Buyer at Customer B)? Coupled to multi-tenant scope (`OQ-3`). Validate via `business-discovery.md` §3.b.

## 7. References

- [`catalog.md`](./catalog.md) — pattern source (7-section structure, schema-table style).
- [`sample.md`](./sample.md) — pattern extensions (state-machine sub-class, write-path adapter shape, idempotency tokens).
- [`bounded-contexts.md` §4.g, §5](../bounded-contexts.md) — Customer Account bounded context; identity-scope edges to other contexts.
- [`capability-map.md` §3.g](../capability-map.md) — Customer capabilities CAP-022 *Authenticate*, CAP-023 *Manage Profile*, CAP-024 *Manage Preferences*; `Net7CustomerAdapter` naming.
- [`glossary.md`](../glossary.md) — `Strategic Buyer`, `Tactical Buyer`, `Mixed Buyer`, `Customer-ID`, `Pricing Tier`, `Workspace`, `ApplicationProject` definitions.
- [`AGENTS.md`](../../../AGENTS.md) — engineering charter; User Personas; ERP Ownership Rules.
- [`business-discovery.md`](../../business-discovery.md) — §3.b Beschaffungsprozess (multi-contact reality), §3.h Lieferanten-Beziehungen (account-level relationship), §3.i Tech-Readiness (auth expectations).
- [`erp-discovery.md`](../../erp-discovery.md) — §3.b Auth/Authz, §3.c Datenmodell (Customer-ID stability), §3.e Schreib-Operationen (profile writes), §3.f Sync-Strategie.
