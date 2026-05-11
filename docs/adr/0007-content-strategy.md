# ADR 0007 — Hybrid Content Strategy

| Status     | **Accepted**                          |
|------------|---------------------------------------|
| Date       | 2026-05-11                            |
| Decided by | Human (System Architect)              |
| Authors    | Human + Claude (chat) + Claude Code   |

## 1. Context

Visual discovery (Screenshot 3) revealed that NET7 stores marketing HTML content with inline-styled brand colors (`#D20072`) directly in the article master record under the `Webshop Beschreibung` tab. This includes:
- Product narrative text ("Das gelbe Aprikosenkernöl bio ist kaltgepresst...")
- Marketing positioning ("Verwendung Kosmetik", "Ausgewählte Bio Projekte")
- Brand-style HTML (`<h3 style="color: #D20072; ...">`)

Visual discovery (Screenshot 5) also showed marketing-style content surfaced on the public product page (e.g., "Upcycling Beauty" article framing, sustainability storytelling) that is editorial in nature, not master-data.

Sprint-1 modeled the Catalog Aggregate (1.4.1) and Documents Aggregate (1.4.4) without separately accounting for marketing/content concerns. Sprint-1 ADR-0003 Rule 1 (NET7 as Single Source of Truth) was scoped to product master, prices, inventory, certifications, documents — not explicitly content.

The platform mandate requires public SEO-discoverable product pages. SEO performance depends on content quality, freshness, and structure beyond what a product master record contains. Multi-language content (DE + EN at minimum) compounds this.

Two structural problems with the current arrangement:
1. **Brand-design coupled to ERP edits**: a brand redesign requires editing every product's HTML in NET7 manually.
2. **Content lifecycle ≠ product lifecycle**: marketing copy changes monthly; INCI/CAS data changes rarely. Same storage layer for both creates conflict.

This ADR codifies the content ownership strategy and the migration trajectory.

## 2. Decision

Adopt a **Hybrid Content Model** with explicit ownership boundaries between NET7 and a dedicated content tier (frontend-managed in V1, CMS-managed in V2).

### 2.1 NET7 owns: Product Master Data

NET7 remains the System of Record for:
- Product identifiers (Teilenummer, EAN-13, INCI, CAS)
- Technical specifications (Sachmerkmale, applications, certifications)
- Pricing (all staffeln and price-classes per Sprint-1 Pricing Aggregate)
- Inventory state
- Regulatory documents (SDS, COA, TDS, Allergen Statement, Flowchart)
- Certifications status (BIO, COSMOS, NATRUE, KOSHER)

This is the existing constitutional alignment with Sprint-1 ADR-0003 Rule 1.

### 2.2 Frontend / CMS owns: SEO + Marketing Content

The new experience layer owns:
- SEO meta-tags, Schema.org structured data, sitemap.xml, robots.txt
- Marketing copy (product positioning, application narratives, brand stories)
- Content articles (sustainability, regulatory updates, formulation guides — V2)
- Brand pages, landing pages, hero imagery
- Multi-language editorial content
- URL structure, internal linking strategy

### 2.3 V1: Hybrid with Adapter Sanitization

V1 keeps the existing NET7 `Webshop Beschreibung` HTML as the **source for product narrative**, but:
- The Catalog adapter **strips inline styles** (`style="..."` attributes) when projecting to the read-model
- The frontend renders the text content with its own design-system styles applied
- Brand color `#D20072` and other style decisions live in the frontend design tokens, not the content
- This decouples brand-design from per-product ERP edits without requiring content migration

// ASSUMPTION: HTML sanitization can be implemented as a deterministic transform (allow-list of tags, strip-attribute on style). Risk if NET7 HTML contains structurally-important inline styles. Validated during V1 implementation.

### 2.4 V2: Introduce CMS

In V2, a headless CMS is introduced (vendor TBD — Strapi, Sanity, Contentful, or alternative). The CMS becomes the owner of:
- Editorial content (articles, brand pages)
- Optionally: gradual migration of NET7 `Webshop Beschreibung` text into CMS-managed product narrative

This migration is **opt-in per product**. Products without CMS-narrative continue to use NET7-sourced narrative. The decision is one-way per product (once migrated to CMS, NET7 narrative is no longer surfaced).

// ASSUMPTION: V2 timing depends on Sprint-1 phase outcomes and content-team availability. Not a Sprint-2 commitment.

### 2.5 Content Authorship Ownership

V1:
- Product narrative (Webshop Beschreibung): edited by AOT product owner in NET7 (existing workflow, unchanged)
- SEO content (meta-tags, structured data, landing pages): owned by engineering, configured per-route in code
- Marketing pages (about, contact, etc.): static-site or simple-CMS, owned by marketing function

V2:
- Editorial content (articles, brand stories): owned by content team (role to be staffed)
- Translation workflow: requires editorial role with translation discipline; auto-translation tools (DeepL, etc.) acceptable for non-regulatory content

// ASSUMPTION: AOT does not currently have a dedicated content/marketing role. Establishing this role is a V2 prerequisite. Discovery question (see OQ-master).

### 2.6 Regulatory Documents Remain in NET7 DMS

The NET7 DMS (Screenshot 1) with versioning, prüfintervall, and workflow remains the authoritative store for:
- SDS (Safety Data Sheets / MSDS)
- COA (Certificates of Analysis)
- TDS (Technical Data Sheets)
- Flowcharts
- Allergen statements
- Customer-specific compliance documents

CMS is **not** in scope for regulatory documents. The DMS workflow is fit-for-purpose and audit-grade.

## 3. Alternatives Considered

### Alt 1 — All Content Stays in NET7

**Approach:** Keep status quo. All product narrative, marketing copy, and editorial content lives in NET7.

**Rejected because:**
- Brand redesign requires per-product ERP edit (operational nightmare)
- Content velocity (marketing copy changes monthly) creates ERP-workflow churn
- Multi-language scaling difficult (NET7's language model is field-level, not document-level)
- Marketing-team workflow not aligned with ERP-team workflow
- SEO performance demands content depth (articles, guides) that doesn't fit in product-master records

### Alt 2 — All Content Migrated to CMS Immediately

**Approach:** Introduce a CMS in V1, migrate all product narrative + marketing content out of NET7 immediately.

**Rejected because:**
- Migration scope too large for V1 timeline
- CMS vendor selection requires Discovery and team-capability assessment (not Sprint-2-ready)
- Existing NET7 workflow (Screenshot 3 shows AOT-team active editor of Webshop Beschreibung) would need replacement before V1 ship
- Risk of content drift during migration (two systems with overlapping responsibility)

### Alt 3 — No CMS Ever; Structured Fields Only

**Approach:** Frontend renders content from structured product fields only (no HTML, no marketing narrative).

**Rejected because:**
- Eliminates marketing flexibility (no application narratives, no brand storytelling)
- Conflicts with SEO requirement (rich content drives ranking)
- Discards existing AOT content investment (Webshop Beschreibung text is real value)
- Reduces public catalog to "data sheet" experience, missing the storytelling that competitors invest in

## 4. Consequences

### 4.1 Positive

- Brand design decoupled from per-product ERP edits (V1 win via sanitization)
- Clear ownership boundaries: NET7 = master data, frontend = experience
- V2 CMS path preserved without forcing premature commitment
- AOT-team workflow for product narrative unchanged in V1 (low disruption)
- Frontend design system can evolve independently of content
- SEO content strategy can grow over time (start with structured + sanitized HTML, add editorial layer in V2)

### 4.2 Negative / Risks

- HTML sanitization is a brittle transform — risk of breaking display if NET7 content has structurally-important styles. Mitigated by: comprehensive sanitization test-cases against current NET7 content (Sprint-2 Phase-0 validation).
- Content-ownership ambiguity for borderline cases (e.g., is "applications" a master-data field or marketing copy?) — requires governance, not just architecture
- V2 CMS migration is non-trivial; deferring creates accumulated debt
- AOT may need to hire/contract a content/marketing role for V2 — organizational decision, not architectural
- Multi-language content strategy unaddressed in V1 (deferred to V2; auto-translation acceptable for non-regulatory in interim)

### 4.3 Open Questions / Revisit Triggers

- Sanitization scope: what HTML tags/attributes are allowed in the V1 transform? (Sprint-2 Phase-0 task)
- Long-term ownership of product narrative: stays in NET7 or migrates to CMS in V2? (Discovery-blocked)
- CMS vendor selection: Strapi, Sanity, Contentful, or other? (V2 decision)
- Content authoring role: who edits CMS content? Existing AOT-team vs new hire? (Discovery)
- Translation workflow: human-translation, auto-translation (DeepL), hybrid? (V2 decision)
- Revisit trigger: brand redesign initiative would accelerate CMS introduction.

## 5. Approval

Decided by: Human (System Architect, Carlos)
Date: 2026-05-11
Recorded by: Claude (chat) + Claude Code as Sprint-2 Foundation ADR.
Status transitions: Accepted; V2 CMS-vendor and migration-trigger decisions remain open and will be successor ADRs.

## 6. Related Documents

- /AGENTS.md (engineering charter, documentation discipline)
- /docs/adr/0001-stack-and-deployment.md (style template)
- /docs/adr/0002-domain-patterns.md (Pattern 1 7-H2 template; Pattern 5 Self-Report Discipline applied to content evolution)
- /docs/adr/0003-erp-boundary-rules.md (Rule 1 NET7-as-SoT — this ADR refines the scope to exclude marketing content from V2 onward)
- /docs/adr/0004-open-question-governance.md (governance for content-strategy OQs)
- /docs/adr/0005-strangler-fig.md (the migration umbrella under which content-strategy evolves)
- /docs/adr/0006-modular-monolith.md (catalog module hosts sanitization in V1)
- /docs/adr/0008-payment-strategy.md (parallel deferral pattern — both ADRs validate readiness before V2 commitment)
- /docs/domain/aggregates/catalog.md (Catalog aggregate; Webshop Beschreibung surfaces here)
- /docs/domain/aggregates/documents-compliance.md (DMS — regulatory document ownership unchanged)
- /docs/domain/glossary.md (terminology baseline — adds content-tier vocabulary)
- /docs/architecture/strategic-foundation.md §1.3 (challenge analysis that motivated this ADR)
- Visual discovery: Screenshot 3 (Webshop Beschreibung HTML in NET7), Screenshot 5 (public product page content)
