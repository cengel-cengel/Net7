# AOT Hero Portfolio — Selection Framework

**Status:** Pre-Discovery Artifact — operationalizes digital-revenue-platform.md §5 (Curated Hero Portfolio V1)
**Purpose:** Convert the strategic "30-80 Hero products" mandate into a concrete decision process that produces a specific V1 launch list
**Audience:** AOT-Sales + AOT-Marketing (curators), AOT-Mgmt (approvers), Engineering (downstream consumers)
**Used in:** Discovery Session 1 (Strategy + Operations, 90 Min) as structured agenda
**Date:** 2026-05-11

---

## §1 Purpose

The digital-revenue-platform.md Principle 5 commits V1 to a **curated Hero portfolio** of ~30-80 products rather than full ~800-product public catalog. This framework gives AOT-Sales + Marketing an **operational decision tool** to:

1. Identify Hero-candidate products from AOT's ~800-product catalog
2. Score them on objective dimensions (commercial / strategic / content-readiness / documentation)
3. Produce a ranked list from which V1 launch set is drawn
4. Establish add/remove governance for post-V1 evolution

The framework is **not** a static list — it is a **repeatable decision process**. V1 produces an initial list; quarterly review can adjust.

**What this framework is NOT:**
- Not a customer-selection tool (which customers see what is handled separately per documents-sprint-2-extension scope)
- Not a content-creation tool (it identifies *what* gets content investment, not *how* content is produced)
- Not an engineering decision (visibility-flag implementation per catalog-sprint-2-extension §2.1)

---

## §2 Selection Dimensions

Four dimensions are scored per product. Each on a 1-5 scale (where 5 = strongest fit, 1 = weakest fit).

### Dimension D1 — Commercial Significance (weight: 30%)

How material is this product to AOT revenue?

| Score | Threshold |
|-------|-----------|
| 5 | Top-10 by revenue OR top-20 by margin contribution |
| 4 | Top-25 by revenue OR top-50 by margin |
| 3 | Top-50 by revenue |
| 2 | Top-100 by revenue |
| 1 | Long-tail (below top-100 in revenue and margin) |

// ASSUMPTION: AOT-Sales has revenue + margin data per product from NET7 — pulled from Statistik-Tab (Screenshot 6 visible tabs list). If granular margin data not available, revenue-only scoring is acceptable for V1.

### Dimension D2 — Strategic Positioning (weight: 30%)

Does this product anchor an AOT-key positioning that drives SEO-targeting + brand-narrative?

| Score | Criterion |
|-------|-----------|
| 5 | Anchor of AOT's flagship narrative (e.g., Upcycling-Beauty hero, signature COSMOS-certified Bio cosmetic oil, food-grade specialty oil that defines AOT's positioning) |
| 4 | Strong fit with one of AOT's primary positioning pillars (Bio cosmetics, food-grade speciality, certified raw materials) |
| 3 | Generic positioning fit (broad B2B raw material, no specific narrative) |
| 2 | Out-of-scope of primary positioning but high commercial value |
| 1 | Commodity item with no story differentiation |

// ASSUMPTION: AOT's primary positioning pillars are: Bio cosmetic oils, food-grade specialty ingredients, certified raw materials (BIO/COSMOS/NATRUE/KOSHER), Upcycling-Beauty narrative (Visual Discovery Screenshot 5). Validate during Session 1.

### Dimension D3 — Content Readiness (weight: 20%)

Is the product's master data sufficient for a high-quality public page today, or does content investment need to precede launch?

| Score | State |
|-------|-------|
| 5 | Complete: Webshop-Beschreibung HTML present + clean, application/use-case text present, high-quality product image, sub-title disambiguator if applicable, certification claims current |
| 4 | Mostly complete — minor gaps (e.g., 1 missing application example, sub-title needs writing) |
| 3 | Partially complete — significant content gaps but recoverable in 1-2 days of Marketing work |
| 2 | Major gaps — 1+ week of Marketing investment needed before launch-ready |
| 1 | Effectively no content — would need full content-creation cycle |

### Dimension D4 — Documentation Coverage (weight: 20%)

Are the regulatory documents (SDS/COA/TDS/Allergen/Flowchart) current and presentable for public access?

| Score | State |
|-------|-------|
| 5 | All applicable docs current (within 12 months), correctly scoped (Public-OK for SDS, etc.) per documents-sprint-2-extension §2.2 |
| 4 | 1 minor doc-gap (e.g., Flowchart outdated but SDS/COA current) |
| 3 | Documentation present but several docs aging (12-24 months old) |
| 2 | Critical docs (SDS) outdated or missing |
| 1 | Severe documentation gaps — would require regulatory review before public-exposure |

---

## §3 Scoring Aggregation

```
Aggregate Score = (D1 × 0.30) + (D2 × 0.30) + (D3 × 0.20) + (D4 × 0.20)

Range: 1.0 - 5.0
```

**Hero-inclusion threshold: Aggregate Score ≥ 3.5**

Rationale: 3.5 represents an upper-third performer (mid-point between "average" and "strong"). Products scoring below typically have a deficiency in at least one dimension that materially weakens public-launch readiness.

// ASSUMPTION: 3.5 threshold may need calibration after first scoring pass. If <30 products qualify, threshold lowers to 3.0. If >80 qualify, threshold raises to 3.7 or commercial-D1 weight increases.

---

## §4 Decision Workflow

### Role Assignments

| Role | Responsibility | Who |
|------|----------------|-----|
| Curator | Initial candidate list + D1/D2 scoring | AOT-Sales (Sales-Lead + Account-Managers) |
| Content Validator | D3/D4 scoring | AOT-Marketing + QM (joint) |
| Final Approver | Approves Hero list per quarter | AOT-Sales-Lead (consultation: AOT-Mgmt for strategic Anchors) |
| Engineering Consumer | Implements visibility-flag per catalog-sprint-2-extension §2.1 | Platform Engineering |

### V1 Initial Selection Process

1. **Candidate identification** (AOT-Sales, 2-4 hours preparation):
   - Pull all products with `aktiv` status from NET7
   - Filter: products with ABC=A or ABC=B (commercial materiality pre-filter)
   - Optional further filter: products with at least one COSMOS/BIO/NATRUE/KOSHER certification (positioning pre-filter)
   - // ASSUMPTION: ABC + certification pre-filter reduces ~800 to ~150-300 candidates. If reduction is too aggressive, relax filters.

2. **D1/D2 scoring** (AOT-Sales, structured spreadsheet):
   - Score each candidate D1 (Commercial) using NET7 revenue + margin data
   - Score each candidate D2 (Strategic) using AOT-positioning judgment
   - Filter: keep candidates with (D1 + D2) ≥ 6 (effectively top-quartile commercial-strategic)
   - Output: shortlist of ~80-150 candidates

3. **D3/D4 scoring** (AOT-Marketing + QM, per-candidate):
   - Audit content state per candidate (Webshop-Beschreibung quality, image quality, etc.)
   - Audit documentation state per candidate (SDS/COA/TDS dates, scope-fitness)
   - Score D3 + D4

4. **Aggregate scoring + threshold** (joint review, 2-hour workshop):
   - Compute aggregate scores
   - Apply 3.5 threshold
   - Adjust threshold if list-size is outside 30-80 range
   - Identify products with high D1/D2 but low D3/D4 — these are "Hero-eligible with content-investment" — schedule content work before launch or hold for second wave

5. **Final approval** (AOT-Sales-Lead, with strategic-spot-check by AOT-Mgmt):
   - Approve list
   - Decide on launch sequencing (see §5)
   - Sign off

### Post-V1 Add/Remove Workflow

- **Quarterly review** (90 Min, AOT-Sales + Marketing + Engineering):
  - Performance review: which Hero products converted (sample-requests, RFQs, account-creations)?
  - Add candidates: which non-Hero products grew commercially or strategically?
  - Remove candidates: which Hero products underperformed for 2 consecutive quarters?
- **Ad-hoc additions** (between quarterly reviews):
  - New product launches with strong commercial expectations
  - Marketing campaign anchor products
  - Approval: Sales-Lead solo

---

## §5 V1 Launch Sequencing

Even within the Hero list, products launch in waves to manage content-investment + engineering load:

| Wave | Products | Timing | Goal |
|------|----------|--------|------|
| Wave 1 (Launch) | ~20-25 top-scoring Heroes | V1 launch date | Establish positioning, anchor SEO targeting |
| Wave 2 | ~15-20 next Heroes | Launch + 4 weeks | Expand catalog depth, broaden long-tail-keyword coverage |
| Wave 3 | Remaining Heroes (15-35) | Launch + 8-12 weeks | Complete V1 Hero portfolio |

// ASSUMPTION: 3-wave structure assumes content-pipeline capacity of ~5-10 high-quality product pages per week (mixed Marketing + Engineering effort). Adjust waves to actual capacity.

---

## §6 Content Investment per Hero Product

Each Hero product receives **minimum content set** before its launch wave:

### Required Content (V1 launch minimum)

| Element | Source | Owner |
|---------|--------|-------|
| Product narrative (DE) | NET7 Webshop-Beschreibung (sanitized per ADR-0007) OR Marketing-written replacement | Marketing |
| Product narrative (EN) | Auto-translation (DeepL) acceptable V1, human-reviewed | Marketing |
| 2-3 specific use-case examples | NET7 Anwendungsgebiete OR Marketing | Marketing |
| Product image (high-quality, white-background per Visual Discovery Screenshot 9 style) | AOT image library OR commissioned photography | Marketing |
| Sub-title disambiguator (e.g., "für kosmetische Zwecke", "GLA 20%" per Screenshot 9) | NET7 Sachmerkmal OR Marketing | Marketing |
| Certification badges (BIO/COSMOS/NATRUE/KOSHER as applicable) | NET7 Zertifizierungs-Master | Auto-generated |
| Documentation links (SDS/COA/TDS Public-scoped) | NET7 DMS | Auto-generated per documents-sprint-2-extension §2.2 |
| Public list-price anchor | Pricing module per OQ-053 resolution | Auto-computed |
| SEO meta-title (60-70 chars) | Marketing | Marketing |
| SEO meta-description (140-160 chars) | Marketing | Marketing |
| Schema.org Product structured data | Auto-generated from above | Engineering |

### Enhanced Content (V1.5 / V2)

- Application guides (article-form deep-dives)
- Customer case studies (with consent)
- Formulation suggestions
- Sustainability stories (Upcycling-Beauty narrative deep-dives)
- Video content

These are V1.5+ scope per ADR-0007 (V2 CMS introduction).

---

## §7 Session 1 Agenda (90 Min)

This framework converts Session 1 into a structured working session:

### Pre-Session (preparation by AOT-Sales)

- AOT-Sales prepares D1/D2 scoring spreadsheet ahead of session
- Marketing prepares D3/D4 audit-template
- Both materials shared with attendees 48h before session

### Session Agenda

| Time | Topic | Output |
|------|-------|--------|
| 0:00-0:15 | Strategy refresher: digital-revenue-platform.md Principle 5 + this framework | Shared understanding |
| 0:15-0:30 | Calibrate dimensions: are D1-D4 the right axes? Adjust scoring scales for AOT-specifics. | Locked scoring methodology |
| 0:30-0:45 | OQ-053 (Public List-Price Anchor): walk through options, pick strategy | Decision-Record OQ-053 |
| 0:45-1:15 | Walk through D1/D2 pre-scored candidates: agree on top-50 shortlist | Shortlist (~50-80 candidates) |
| 1:15-1:30 | Wave-sequencing decision: which 20-25 in Wave 1? | Wave-1 list named |

### Session Deliverable

- **Decision-Record** signed: List of Wave-1 Hero products, OQ-053 List-Price decision, dimension-weight adjustments if any
- Action items: Marketing to score D3/D4 on shortlist within 1 week; finalized Hero list approved by Sales-Lead within 2 weeks
- Triggers OQ-049-Closure (per digital-revenue-platform.md §7 — OQ-049 reframed as Hero-Curation)

---

## §8 Discovery Anchors

This framework operationalizes:
- digital-revenue-platform.md §5 — Curated Hero Portfolio V1 scope
- digital-revenue-platform.md §7 — OQ-049 reframe
- digital-revenue-platform.md §8 — Hero-curation ownership question (this framework provides the workflow)
- digital-revenue-platform.md §9 — V1 success criteria (Hero-portfolio conversion metrics)
- aot-discovery-phase-0-brief.md §5 OQ-049 → updated to use this framework as discussion-structure
- catalog-sprint-2-extension §2.1 → PublicVisibilityFlag semantics now means Hero-membership (per this framework's output)

---

## §9 References

- /docs/strategy/digital-revenue-platform.md (constitutional Principle 5 + §5 Hero-Portfolio + §8 Ownership-Territory)
- /docs/adr/0005-strangler-fig.md (Phase 1 public catalog = Hero portfolio first)
- /docs/adr/0007-content-strategy.md (V1 sanitized HTML + V2 CMS — Hero products are first CMS-migration candidates)
- /docs/domain/aggregates/catalog-sprint-2-extension.md §2.1 (PublicVisibilityFlag = Hero-Membership semantic)
- /docs/domain/open-questions-master-sprint-2-extension.md (OQ-049 closure via this framework)
- /docs/discovery/aot-discovery-phase-0-brief.md (Session 1 outreach + agenda template — updated to point here)
- Visual Discovery: Screenshot 6 (ABC-classification, Statistik-tab as data source), Screenshot 9 (existing product-card style for content-consistency), Screenshot 5 (Upcycling-Beauty narrative anchor)
- AGENTS.md (engineering charter — Hero-portfolio-curation as new ownership territory)
