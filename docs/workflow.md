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

## 11. Sandbox Constraints

The Anthropic Claude Code sandbox enforces specific limitations on git operations against the GitHub remote. These were discovered during Sprint 0 closeout (when an annotated `sprint-0` tag could not be pushed) and reinforced during the Sprint-1 glossary-fix workaround (PR #9 had to land as a fresh branch rather than as an amend of PR #8).

- **HTTP 403 from the sandbox proxy** for:
  - Tag pushes (e.g., `sprint-X` annotated tags via `git push origin <tag>` or `git push --tags`).
  - Branch deletions (`git push origin --delete <branch>`).
  - Updates to existing remote branches (any push that modifies an existing ref — including `--amend` + force-push, retry of a partially-failed initial push, second commit on an already-pushed branch).
- **What works**:
  - First-time push of a new branch with new commits (`git push -u origin <new-branch>`).
  - All read-only `mcp__github__*` calls (status, check-runs, reviews, comments, file contents).
  - `mcp__github__create_pull_request` and `mcp__github__merge_pull_request`.
- **Implication**: every reference-mutation operation against the remote (tag, branch delete, branch update) must be performed by the Human via the GitHub Web UI. Claude Code cannot recover from a partial push by amending or force-pushing — it must abandon the branch and create a fresh one.

This constraint is the structural reason for §12 Branch Protection Rules below.

---

## 12. Branch Protection Rules

A direct consequence of §11 Sandbox Constraints. Branch discipline is enforced by the sandbox: there is no syntactic way to do a second push to an existing branch.

- **`1 Branch = 1 Commit = 1 PR`** — strictly enforced. Every cosmetic fix or content adjustment after the initial push requires a new branch and a new PR. Recovery-DIFF (§13) is the discipline that catches problems *before* the initial push, where they are still cheap.
- **No `git commit --amend` after the first push** — would require force-push, which the sandbox rejects.
- **Branch-naming conventions**:
  - `docs/<slug>` — general documentation PRs.
  - `domain/<slug>` — domain-modeling PRs (Sprint 1 aggregate sketches, capability map, glossary, etc.).
  - `sprint-X-closeout/<slug>` — closeout-rework PRs at the end of Sprint N (used in Sprint-1 closeout).
  - Sprint-2+ conventions to be defined as the implementation phase begins.
- **Branch cleanup after merge**: via the GitHub Web UI ("Delete branch" button on the merged PR or the branches page). Claude Code cannot delete branches.
- **See also**: §9 PR conventions for PR title/body format and the `<type>/<short-slug>` branch pattern. §12 covers protection rules (what cannot happen and why); §9 covers conventions (what should happen).

---

## 13. Recovery DIFF

A methodical content-verification step inserted between the Stats-Push (§15 step 5) and PR creation (§15 step 9). Originated as a workaround during Sprint-1 — stats-only push output (a single line "1 file changed, N insertions") repeatedly hid content-quality problems that surfaced only after merge (e.g., the glossary count mismatch in PR #8 that required cosmetic fix-PR #9). It is now an explicit step in the 9-step workflow.

- **Why**: a `git push` confirmation does not verify content. Sprint-1 evidence shows that pattern-drift, count mismatches, and cosmetic bugs slip through stats-only review and require costly fix-PRs after merge.
- **When**: after step 5 (Stats-Push-STOP) and before step 9 (PR creation) — inserted as steps 6 + 7 in the 9-step workflow (§15).
- **What Claude (chat) requests** in the Recovery-DIFF prompt:
  - Complete outline with line numbers (H1 / H2 / H3 hierarchy).
  - Sample rows (first / last in tables; critical or hand-named rows from the task-prompt).
  - Counts (invariants, open questions, components, capabilities, risks — depending on doc type).
  - Pattern consistency vs predecessor docs (for documents in a series, e.g., aggregate sketches).
  - Bytes-per-line ratio with long-liner samples — a content-density sanity check (cf. §15 sub-patterns).
  - Verification of Carlos-mandatory items where the task-prompt named explicit must-haves (e.g., "must include all 5 example risks").
- **What Claude Code returns**: a structured Recovery-DIFF response in a single code block, point-by-point against the request.
- **Status**: from Sprint-1 forward, Recovery-DIFF is a default step in any non-trivial PR cycle. Skipping it is a regression to Sprint-0-style stats-only push.

---

## 14. Mobile Paste Echo Detection

A pathology-handling pattern. Mobile clipboard or app behaviour occasionally produces accidental echoes — the same task-prompt or message pasted twice in close succession, or empty messages where a task-prompt was expected.

- **Phenomenon**:
  - Identical block within recent turns (literal duplicate).
  - Truncated or empty body where a task-prompt or `GO` command was expected.
  - Specifically observed during Sprint 1: Task 1.4.4 `GO MERGE` posted twice; Sprint-1 closeout P3 task-prompt repeated.
- **Detection**: literal-equality check on the most recent inbound message against the previous Human turn (or two).
- **Response**: do **not** process the duplicate as a new request. Instead, prompt actively:
  > "Looks like a duplicate paste — was that intentional, or did you mean to send something else?"
  Empty messages: same gentle clarification rather than silent discard.
- **Why this matters**: silently re-running a `GO MERGE` is harmless (idempotent), but silently re-running a `Plan-STOP` or `GO PUSH` could cause double-execution of side-effecting steps. The cost of asking is low; the cost of acting on a phantom message can be a duplicate PR.

---

## 15. PR Review Sequence (9-Step Workflow)

Codifies the Claude-Code-internal flow that drills down into step 9 of §4 Task Lifecycle (*"Claude Code: inspects → plans → STOPs → implements → tests → DIFF → STOPs → opens PR"*). Established and refined across Sprint 1 (Tasks 1.4.1 – 1.4.5 aggregate sketches, Sprint-1 closeout P1 / P2).

The 9 steps:

1. **Inspektion** — Claude Code reads source files (predecessor docs, AGENTS.md, capability-map, etc.).
2. **Plan-STOP** — Claude Code returns plan: outline, components, naming, themes, assumptions. No implementation yet.
3. **GO PLAN** — Claude (chat) reviews the plan and releases (or returns corrections).
4. **Implementation** — Claude Code writes the file(s), isolated commits.
5. **Stats-Push-STOP** — Claude Code commits, pushes the branch, reports stats (lines, bytes, heading counts). Stops here.
6. **Recovery-DIFF-Anstoß** — Claude (chat) requests the content-depth DIFF (per §13).
7. **Recovery-DIFF** — Claude Code returns outline + samples + counts + B/L sanity-check, in a single structured response.
8. **GO PUSH** — Claude (chat) reviews the DIFF and releases (or requests content enrichment).
9. **PR creation** — Claude Code calls `mcp__github__create_pull_request` with `WHAT / WHY / TEST STEPS / RISKS` body (per §7 task-prompt template).

After step 9 the bridge pattern resumes (§4 steps 10–14): Claude (chat) reviews the PR via `web_fetch`, returns `GO MERGE` (or corrections), Carlos merges via UI, tests on Vercel preview / Railway dashboard, and reports status.

**Sub-patterns observed across Sprint-1**:

- **Bytes-per-line profile** as a content-density sanity-check. Empirical Sprint-1 ranges: <60 for list-dominated docs (procurement-hypotheses 13 B/line, glossary 55 B/line, capability-map 60 B/line), 88–150 for prose-heavy DDD docs (bounded-contexts 88, aggregate sketches 100–150). Values substantially outside the expected range for the doc type are a yellow flag for content quality.
- **Pattern-application self-report** inside the doc — the document itself names which patterns it reuses from predecessors and which extensions are *not relevant*. Established in Sprint-1 aggregate sketches §2 Methodology bullets.
- **Carlos-mandatory verification step** during Recovery-DIFF — when the task-prompt names explicit must-haves (e.g., "must include 5 example risks", "must use 7-column schema"), the Recovery-DIFF response includes an explicit pass/fail check on each named item.

---

## 16. References

- [`AGENTS.md`](../AGENTS.md) — engineering charter (binding for all tools)
- [`CLAUDE.md`](../CLAUDE.md) — Claude Code operating rules
- [`ROADMAP.md`](../ROADMAP.md) — Sprint structure and deliverables
- [`docs/adr/`](./adr/) — Architecture Decision Records
