# workflow.md
# Collaboration Protocol — AOT Procurement Platform

This document defines the three-way collaboration protocol between the **Human**, **Claude (chat)**, and **Claude Code**.

It is read at the start of every Task by Claude Code and by Claude (chat).

For the engineering charter see `AGENTS.md`.
For Claude Code operational rules see `CLAUDE.md`.
For the delivery plan see `ROADMAP.md`.

---

## 1. Roles

### Human (Carlos)
- System Architect, Product Owner, Delivery Lead.
- Decides business rules, architecture, priorities, acceptance.
- Works primarily mobile (iPhone), occasionally desktop browser.
- Writes German. Expects concise, mobile-readable replies.

### Claude (chat — web / app)
- Program architect, planner, reviewer, risk advisor.
- Reads public GitHub files via `web_fetch`.
- Writes Task-Prompts as code blocks (delegated to Claude Code).
- Reviews PRs by fetching the PR URL.
- **Cannot** push code, create PRs, or modify GitHub state directly.

### Claude Code (web browser / mobile app)
- Implementation agent.
- GitHub-linked via OAuth — has read/write access to the repo.
- Inspects, plans, implements, opens PRs.
- Reads `AGENTS.md`, `CLAUDE.md`, `ROADMAP.md`, this `workflow.md`, and relevant ADRs at the start of every Task.

---

## 2. Infrastructure constraint — cloud-only

- No local development environment, no local clones (Sprints 0–2; revisit at Sprint 3 if needed).
- Repository operations via GitHub Web UI or Claude Code's GitHub integration.
- Implementation runs in Claude Code's cloud sandbox.
- Frontend preview: Vercel preview deployments per PR.
- Backend logs / runtime: Railway dashboard.

Trade-off accepted: no local debugging. If a bug requires deep inspection, escalate to a Codespaces session as an out-of-band exception.

---

## 3. Terminology

- **Sprint** = Roadmap phase. Days to weeks. Multiple Tasks. Multiple deliverables. Defined in `ROADMAP.md`.
- **Task** = micro-iteration. Minutes to hours. One PR.
- **`1 Task = 1 PR = 1 merge`**.
- Never bundle backend + frontend in one Task → split into phased Tasks.

---

## 4. Task lifecycle (cloud-only bridge pattern)

1. Human states wish (1–2 sentences).
2. Claude (chat) asks multi-choice clarifying question if needed (max 3 questions, 2–4 options each, with recommendation).
3. Human answers (1–2 words).
4. Claude (chat) inspects relevant files via `web_fetch` (max 5–10 findings).
5. Claude (chat) presents PLAN as bullet list + **STOP for GO PLAN**.
6. Human: GO or corrections.
7. Claude (chat) writes Task-Prompt for Claude Code as a single code block.
8. Human pastes into Claude Code (browser or mobile).
9. Claude Code: inspects → plans → STOPs → implements → tests → DIFF → STOPs → opens PR. Returns PR URL.
10. Human posts PR URL (or full Claude Code output) back to Claude (chat).
11. Claude (chat) reviews via `web_fetch` on PR URL. Returns **GO MERGE** as code block, or corrections.
12. Human merges in browser. Tests on Vercel preview / Railway dashboard.
13. Human reports status: OK or Bug.
14. Bug → back to step 1. OK → next Task.

**STOP points (never skip):**
- After step 5 (before Claude Code writes anything).
- After step 11 (before merge).

---

## 5. Principles

- **KOMPAKT** — all replies short, mobile-readable. Bullets over prose.
- **Inspektion first** — read before writing, always.
- **Multi-choice over open questions** at architecture decisions.
- **Code blocks for actionable items** (Task-Prompts, GO commands).
- **Migrations idempotent** (`IF NOT EXISTS`, `DO`-blocks).
- **Bug: diagnosis before fix.**
- **No speculation** — never invent ERP capabilities, API behavior, or library features. Mark assumptions explicitly with `// ASSUMPTION:`.
- **No tables on mobile** (too narrow). Bulleted/numbered lists instead.
- **No long explanations when "GO" suffices.**

---

## 6. Challenge Principle

If the Human says something that contradicts `AGENTS.md`, an existing ADR, or established architecture: Claude (chat) **flags the contradiction immediately** rather than asking for confirmation or complying silently.

Better to surface conflicts than ship incoherent code.

This applies to Claude Code too: if a Task-Prompt asks for something outside the engineering charter, Claude Code stops and asks before implementing.

---

## 7. Task-Prompt format (mandatory structure for prompts to Claude Code)

Every Task-Prompt issued by Claude (chat) must follow this template, delivered as one fenced code block ready to copy-paste:

````
═══════════════════════════════════════════════════════
TASK [number]: [short title]
═══════════════════════════════════════════════════════

ZIEL
[1-2 Sätze: was Erfolg bedeutet]

KONTEXT
- Sprint: [aktueller Sprint aus ROADMAP.md]
- Files in scope: [Liste]
- Vorherige Tasks/PRs: [falls relevant]

SUB-TASKS
A) [erster konkreter Sub-Task]
B) [zweiter]
C) [dritter, falls nötig]

WORKFLOW
1. Inspektion: lies [Files] vollständig bevor du schreibst
2. Plan: zeig geplante Änderungen als Bullets
3. STOP — warte auf "GO PLAN" bevor du implementierst
4. Implementation: kleine isolierte Commits
5. Build/Test: lint, typecheck, tests
6. DIFF: zeig Zusammenfassung
7. STOP — warte auf "GO PUSH" bevor du PR erstellst
8. PR-Titel: <type>(<scope>): <kurzbeschreibung>
   PR-Body: WHAT / WHY / TEST STEPS / RISKS

KOMPAKT
Antworten kurz, Bullets statt Prosa, keine Tabellen.

CONSTRAINTS
- Respektiere AGENTS.md Hard Rules
- Respektiere ADR 0001 (kein apps/, packages/, turbo.json bis Sprint 3)
- Markiere Annahmen explizit: // ASSUMPTION:
- Bei unklarer Anforderung: STOP und fragen
````

---

## 8. GO commands from Claude (chat)

After review steps, Claude (chat) returns a copy-paste-ready code block:

````
GO PLAN
[optional kurze Notiz]
````

oder

````
GO PUSH
[optional kurze Notiz]
````

oder

````
GO MERGE
Test-Schritte:
1. [...]
2. [...]
````

oder bei Korrekturen: konkrete Änderungswünsche als bullet list, kein GO.

---

## 9. PR conventions

- **Title**: Conventional Commits — `<type>(<scope>): <description>` (e.g., `feat(rfq): add sample request validation`).
- **Body**: WHAT, WHY, TEST STEPS, RISKS.
- **Branch**: `<type>/<short-slug>` (e.g., `feat/rfq-validation`, `fix/erp-retry-timeout`).
- **One Task = one PR**. No bundling.

---

## 10. Pause and handover

If the Human pauses mid-Task: explicit signal **"NICHTS BRENNT — ALLES STEHT"**.

Claude (chat) responds by snapshotting current state:
- Active Task ID and title
- Last PR opened (URL + status)
- Next planned step
- Outstanding STOP-point (waiting for GO PLAN / GO MERGE)

Resumption is then clean from a single re-entry message.

---

## 11. References

- [`AGENTS.md`](../AGENTS.md) — engineering charter (binding for all tools)
- [`CLAUDE.md`](../CLAUDE.md) — Claude Code operating rules
- [`ROADMAP.md`](../ROADMAP.md) — Sprint structure and deliverables
- [`docs/adr/`](./adr/) — Architecture Decision Records
