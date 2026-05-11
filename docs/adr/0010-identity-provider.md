# ADR 0010 — Identity Provider (Proposed, Pending Discovery + Cost)

|Status    |**Proposed — Pending AOT-Legal (DSGVO), AOT-Mgmt (Cost), Team-Capability validation**|
|----------|-------------------------------------------------------------------------------------|
|Date      |2026-05-11                                                                           |
|Decided by|Human (System Architect) — pending stakeholder confirmation                          |
|Authors   |Human + Claude (chat) + Claude Code                                                  |

**Important:** This ADR is **NOT Accepted**. It proposes Auth0 as default V1 IdP to enable Phase-0 planning, but the decision is gated on §5 Gates G1-G4. Stack remains reversible until Phase-0 spike completion.

## 1. Context

ADR-0009 (Tech-Stack, Proposed) deferred the Identity Provider decision to a separate ADR. Strategic-foundation §6.2 Layer 5 proposed “managed IdP (Auth0 / AWS Cognito / Keycloak SaaS)” without selecting between them. Build-own IdP was rejected at proposal-level — AOT has no regulatory mandate to operate its own identity infrastructure, and authentication is not a core differentiator.

Three forces require a vendor proposal now:

1. **IdP is on critical path for V1 B2B-tier.** Per ADR-0005 §2.6 Phase-3 (B2B Migration Wave 1), the authenticated B2B portal requires functioning identity layer. No IdP, no authenticated catalog. No authenticated catalog, no migration from legacy.
1. **Customer-account-sprint-2-extension §2.1 defines IdentityBridgeMapping** as the sole platform-side mechanism connecting `auth.UserId` to NET7 `CustomerAccountId`. This requires the chosen IdP to support webhook/event emission for sync purposes.
1. **Constitutional constraints** are pre-committed:
- ADR-0003 Rule 4 — AuthenticationCredential NEVER propagated to NET7
- customer-account-sprint-2-extension §3 — GuestSession is a **platform-only** concept (NOT an IdP concept), handled by the `auth` module without IdP calls
- digital-revenue-platform.md Principle 4 — Authenticated layer handles procurement logic (the IdP serves it, doesn’t define it)

This ADR proposes Auth0 as the V1 default, with explicit gates before commitment.

## 2. Proposed Decision

### 2.1 Vendor — Auth0 for V1

**Choice:** Auth0 (Okta subsidiary) for V1 IdP.

**Rationale:**

- **Time-to-market:** strongest developer-experience for Next.js/Node.js stack (per ADR-0009 §2.1-2.2); fastest path to working B2B-tier
- **B2B-native features:** Auth0 Organizations capability directly maps to the multi-user account model required by digital-revenue-platform.md Principle 4 (procurement enablement) and customer-account-sprint-2-extension §4.1 UserRoleAssignment
- **Free tier covers V1 scale:** 7,000 MAU free; AOT’s V1 authenticated-user base estimate ~100-500 active users — well within free tier
- **V2-ready:** SAML/OIDC support for enterprise SSO (V2 roadmap per strategic-foundation §9.2)
- **Webhook + Action support:** enables IdentityBridgeMapping sync per customer-account-sprint-2-extension §2.1
- **JS/TS SDK quality:** aligns with team-skill profile per ADR-0009 Gate G2

### 2.2 Capability Requirements

The chosen IdP must support all of:

|Requirement                             |Why                                                                         |V1/V2                       |
|----------------------------------------|----------------------------------------------------------------------------|----------------------------|
|Email/password authentication           |Baseline B2B login                                                          |V1                          |
|Email OTP / passwordless option         |Friction reduction for B2B users                                            |V1                          |
|Custom claims in tokens                 |Role-based access (Buyer/Admin) per customer-account-sprint-2-extension §4.1|V1                          |
|Webhook/Action on user events           |IdentityBridgeMapping sync to NET7                                          |V1                          |
|Organizations / Multi-user-account model|Multiple users per CustomerAccount                                          |V1 (basic), V2 (hierarchies)|
|EU data residency option                |DSGVO compliance                                                            |V1                          |
|SAML 2.0 / OIDC enterprise SSO          |V2 enterprise customers                                                     |V2                          |
|API tokens / Machine-to-Machine auth    |V2 customer-ERP integration                                                 |V2                          |
|Rate limiting + bot protection          |Account-creation abuse prevention                                           |V1                          |

Auth0 supports all V1 requirements out-of-box. V2 requirements are available on higher plan tiers.

### 2.3 Integration Architecture

```
┌──────────────────────────────────────────────────┐
│ User (B2B authenticated)                          │
└────────────────────┬──────────────────────────────┘
                     │ login flow
                     ▼
┌──────────────────────────────────────────────────┐
│ Auth0 (IdP)                                       │
│ - Email/password verification                     │
│ - Issues JWT with custom claims                   │
│ - Webhook on user.created → IdentityBridge        │
└────────────────────┬──────────────────────────────┘
                     │ JWT + webhook
                     ▼
┌──────────────────────────────────────────────────┐
│ `auth` module (in Modular Monolith per ADR-0006)  │
│ - Validates JWT                                   │
│ - Maintains IdentityBridgeMapping (UserId ↔       │
│   NET7 CustomerAccountId)                         │
│ - Handles GuestSession entirely platform-side     │
│   (NO Auth0 involvement for anonymous)            │
└────────────────────┬──────────────────────────────┘
                     │ resolved CustomerAccountId
                     ▼
┌──────────────────────────────────────────────────┐
│ Other modules (catalog, pricing, sample, rfq, etc) │
│ - Receive authenticated CustomerAccountId via      │
│   `auth` module API (per ADR-0006 module boundary) │
└──────────────────────────────────────────────────┘
```

**Critical separation:**

- **Authenticated users** → Auth0 handles credentials, JWT signing, MFA, password reset
- **GuestSessions** → handled entirely by `auth` module platform-side (per customer-account-sprint-2-extension §3, no Auth0 involvement). This preserves the Web-only nature of GuestSession.
- **NET7** → receives no Auth0 data; only the resolved `CustomerAccountId` from IdentityBridge (per Sprint-1 ADR-0003 Rule 4 constitutional)

### 2.4 Cost Model

**V1 estimate:** $0/month for first 7,000 MAU (Free tier “B2C Essentials”).

**Likely V1 actual:** $0-25/month (Free tier covers; small risk of overage on MFA volume).

**V2 estimate:** $240-1,500/month depending on user count + features (e.g., Enterprise plan needed for SAML, custom domains).

// ASSUMPTION: V1 active-user estimate of 100-500 B2B users is conservative; if AOT customer-base scale is materially different, cost projection needs revision.

### 2.5 Status & Gates

This proposal becomes **Accepted** only after ALL gates pass:

- **Gate G1 — Cost projection accepted:** AOT-Mgmt confirms V1 + projected V2 Auth0 costs fit budget (OQ-062 follow-on; AOT-Mgmt-owned)
- **Gate G2 — DSGVO confirmed:** AOT-Legal validates Auth0’s EU data-residency + Data Processing Agreement; confirms GuestSession-separation strategy is DSGVO-compliant
- **Gate G3 — Team-Capability:** V1 team has Auth0/Next.js-SDK experience OR commitment to ramp-up
- **Gate G4 — Phase-0 spike confirms:** IdentityBridge webhook → NET7-adapter flow works on this stack

Until ALL pass, the ADR remains **Proposed**. Reversibility is high — Auth0 OIDC-compliance means migration to Cognito or Keycloak SaaS is feasible (data export + token-format compatibility).

## 3. Alternatives Considered

### Alt 1 — AWS Cognito

**Approach:** AWS-native managed identity. Native integration if backend on AWS.

**Trade-offs:**

- Pro: Cheapest at scale ($0.0055/MAU above 50,000 free tier), predictable pricing, strong AWS-ecosystem integration
- Con: Weaker DX than Auth0, less polished SDKs, B2B-Organizations model requires more custom code, AWS-tilted vendor relationship

**Status:** **Primary fallback if Gate G1 (Cost) fails.** If V2 user count pushes Auth0 cost above acceptable threshold, Cognito migration is the planned exit. Cognito covers all V1 requirements; gap is DX speed.

### Alt 2 — Keycloak SaaS (Phasetwo, Cloud-IAM, etc.)

**Approach:** Managed hosting of open-source Keycloak (Red Hat-originated, broad enterprise IAM features).

**Trade-offs:**

- Pro: Open-source foundation — easier long-term vendor exit (self-host fallback); strong SAML/OIDC; full feature parity with Auth0 + Cognito
- Con: SaaS-provider quality variable; smaller ecosystem than Auth0/Cognito; ops complexity higher even managed; less polished JS-SDK

**Status:** **Secondary fallback if Gate G2 (DSGVO/EU-data-residency) blocks Auth0.** Some Keycloak-SaaS providers are EU-headquartered with strong DSGVO posture.

### Alt 3 — Build-Own IdP

**Approach:** Custom-built authentication service.

**Rejected because:**

- Not a core differentiator for AOT
- Re-implements security primitives (password hashing, token signing, MFA, OAuth flows) — significant attack-surface investment
- No regulatory mandate; B2B raw-materials sector has no special identity-handling requirements
- 6-12 months engineering investment for parity with managed offerings

### Alt 4 — Identity-via-NET7 (Existing Customer Database)

**Approach:** Use NET7’s existing customer records as the identity source.

**Rejected because:**

- Violates Sprint-1 ADR-0003 Rule 4 (AuthenticationCredential never in NET7)
- NET7 not designed as IdP — no OAuth/OIDC support, no MFA, no session management
- Would require building IdP-equivalent layer on top of NET7

## 4. Consequences

### 4.1 If Adopted (post-gates)

- Fast time-to-market: Auth0 Quickstart for Next.js means functional auth in days, not weeks
- Code-sharing: Auth0 SDK provides both Next.js (frontend) + Node.js (backend) bindings → consistent token handling
- Operational burden minimized: Auth0 handles password hashing, MFA, breach detection, bot protection
- DSGVO posture: Auth0 has established EU-data-residency option and standard DPA
- V2-readiness: SAML/OIDC support already in place; no re-platforming when first enterprise customer requests SSO
- Vendor risk localized: Auth0 OIDC-compliance enables migration to alternatives if cost/strategy changes

### 4.2 Risks

- **Pricing escalation V2:** Auth0 cost grows with user count + features. Multi-thousand-user B2B base at V2 could push to $1000+/mo. Mitigation: monitor Auth0 cost trajectory; Cognito fallback path established
- **Okta-acquisition uncertainty:** Auth0 acquired by Okta; pricing/strategy may evolve. Mitigation: OIDC-compliance keeps migration feasible
- **Lock-in via custom Actions:** Auth0’s custom-rule mechanism creates platform-specific code. Mitigation: keep custom logic minimal; mirror in `auth` module where possible
- **DSGVO scrutiny on GuestSession:** GuestSession data lives outside Auth0 (in `auth` module’s own storage). AOT-Legal must validate DSGVO-treatment of GuestSession data (retention, deletion, user-rights). Not an Auth0 risk but a `auth` module design requirement
- **Account-creation abuse:** anonymous users could try mass-account-creation. Mitigation: Auth0 bot protection + rate limiting + email verification mandatory

### 4.3 Reversibility

Reversibility stages similar to ADR-0009:

- Before Phase-0 spike: trivially reversible (no production identity records yet)
- After Phase-0 spike but before V1 launch: moderate (some user data created; IdP-export-then-reimport possible)
- After V1 launch: significant (production user accounts; migration requires user-consent + re-authentication cycle)

OIDC-compliance preserves migration option indefinitely.

## 5. Approval

**Status: Proposed.**

Decided by: Human (System Architect, Carlos) — proposal only, not Accepted.
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Phase-3 deliverable.

**Status transitions:**

- Currently: Proposed
- Upgrade to Accepted: requires ALL 4 gates (G1–G4 per §2.5) to pass
- Downgrade to Superseded: requires successor ADR if Discovery mandates different IdP
- This ADR does NOT block Phase-0 implementation planning; it enables it.

## 6. Related Documents

- /AGENTS.md (engineering charter — security mandate)
- /docs/strategy/digital-revenue-platform.md (Principle 4 — authenticated layer procurement logic served by IdP)
- /docs/adr/0001-stack-and-deployment.md (style template)
- /docs/adr/0002-domain-patterns.md (Pattern 5 Self-Report Discipline — applied to vendor evaluation)
- /docs/adr/0003-erp-boundary-rules.md (Rule 4 Authentication-Credential-Containment — constitutional; NET7 isolation preserved by §2.3 integration architecture)
- /docs/adr/0005-strangler-fig.md (Phase-3 B2B-Migration requires functioning IdP — direct dependency)
- /docs/adr/0006-modular-monolith.md (`auth` module hosts IdentityBridgeMapping + GuestSession; IdP plugs into this module)
- /docs/adr/0007-content-strategy.md (no direct interaction; content tier is anonymous-facing)
- /docs/adr/0008-payment-strategy.md (payment deferred; when introduced V2, PSP-auth coordinates with this IdP)
- /docs/adr/0009-tech-stack.md (parallel proposal-pending pattern; §2.5 explicitly deferred IdP to this ADR)
- /docs/domain/aggregates/customer-account.md + customer-account-sprint-2-extension §2.1 §3 (IdentityBridgeMapping + GuestSession parallel concept)
- /docs/domain/open-questions-master-sprint-2-extension.md (OQ-062 hosting — adjacent decision; no direct OQ for IdP yet, may add OQ-063 if Gates produce questions)
- /docs/discovery/aot-discovery-phase-0-brief.md (Session-3 IT-discussion may surface IdP topics in addition to hosting)
- /docs/architecture/strategic-foundation.md §6.2 Layer 5 (predecessor proposal that this ADR refines into concrete vendor recommendation)
