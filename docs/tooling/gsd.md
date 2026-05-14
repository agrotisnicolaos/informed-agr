# GSD (Get-Shit-Done) bundle

The biggest single thing in the toolchain. ~65 skills prefixed `gsd-`, all installed at user scope under `~/.claude/skills/`. Distributed via npm (`npx get-shit-done-cc --claude --global`), not as a Claude Code plugin marketplace entry. Currently v1.40.0.

GSD is a **structured workflow** for shipping projects in phases. The default loop is `discuss → plan → execute → verify → ship`, repeated per phase, gated by quality reviews. It's opinionated — you adopt the discipline or you don't.

**All GSD skills are slash-command-activated** (you type `/gsd-<name>`). A few have skill-style auto-triggering on top, but the primary activation is manual.

Run `/gsd-help` for the live list. Start with [`/gsd-config`](#gsd-config) to set defaults, then [`/gsd-new-project`](#gsd-new-project) to begin.

---

## Project lifecycle (5 skills)

The outermost loop — projects contain milestones, milestones contain phases.

| Skill | What it does | When |
|---|---|---|
| `/gsd-new-project` | Initializes a new project with deep context-gathering. Produces `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`. | Day zero of any project. Run **once** per project. |
| `/gsd-new-milestone` | Starts a new milestone cycle — updates PROJECT.md and routes to requirements gathering. | When you've completed one milestone (e.g. v0.1) and want to scope v0.2. |
| `/gsd-complete-milestone` | Archives a completed milestone and prepares for the next version. | After all phases in a milestone are shipped + verified. |
| `/gsd-milestone-summary` | Generates a comprehensive summary of a milestone for team onboarding or review. | When sharing milestone progress with stakeholders. |
| `/gsd-cleanup` | Archives accumulated phase directories from completed milestones. | Periodically — when `.planning/` is getting cluttered. |

---

## Phase workflow — the canonical loop (6 skills)

The inner loop you'll run dozens of times per project. **This is the spine of GSD.**

| Order | Skill | What it does | Output |
|---|---|---|---|
| 1 | `/gsd-spec-phase <N>` | Clarifies WHAT a phase delivers. Scores ambiguity, asks targeted questions. | `SPEC.md` |
| 2 | `/gsd-discuss-phase <N>` | Adaptive questioning to gather phase context before planning. | Updated discussion log |
| 3 | `/gsd-plan-phase <N>` | Creates detailed phase plan with task breakdown, dependency analysis, goal-backward verification. Calls `mcp__context7__*` for current library docs. | `PLAN.md` |
| 4 | `/gsd-execute-phase <N>` | Executes all plans in the phase with wave-based parallelization. Auto-invokes superpowers TDD + verification. | Atomic commits per task |
| 5 | `/gsd-verify-work <N>` | Validates built features through conversational UAT (User Acceptance Testing). | UAT pass/fail |
| 6 | `/gsd-ship` | Creates PR, runs review, prepares for merge after verification passes. | PR opened |

**This is the single most important pattern in GSD.** Memorize this sequence.

Optional inserts mid-loop:
- After step 3 (plan): `/gsd-plan-review-convergence` — cross-AI plan review until no HIGH concerns remain.
- After step 3 (plan): `/gsd-review` — request external AI peer review of the plan.
- After step 4 (execute): `/gsd-add-tests` — generate tests for the completed phase based on UAT criteria.

---

## Quality gates (7 skills)

Run after `/gsd-execute-phase` (or retroactively on already-shipped phases).

| Skill | What it does | When |
|---|---|---|
| `/gsd-code-review` | Reviews source files changed during a phase for bugs, security issues, code quality. | After execute-phase, before ship. |
| `/gsd-secure-phase` | Retroactively verifies threat mitigations from PLAN.md threat model exist in code. | After execute-phase, especially for auth/data-handling work. |
| `/gsd-eval-review` | Audits an executed AI phase's evaluation coverage. Produces remediation plan. | For phases that built AI systems (LLM features, agents, classifiers). |
| `/gsd-ui-review` | Retroactive 6-pillar visual audit of implemented frontend code. | After any phase that shipped UI. |
| `/gsd-validate-phase` | Retroactively audits and fills Nyquist validation gaps for a completed phase. | When you suspect a phase shipped without enough tests. |
| `/gsd-audit-uat` | Cross-phase audit of all outstanding UAT and verification items. | Periodically — to catch UAT debt across multiple phases. |
| `/gsd-audit-milestone` | Audits milestone completion against original intent before archiving. | Before `/gsd-complete-milestone`. |

---

## Diagnostics + recovery (5 skills)

For when things go wrong or you need to inspect state.

| Skill | What it does | When |
|---|---|---|
| `/gsd-debug` | Systematic debugging with persistent state across context resets. Spawns `gsd-debugger` agent. | Any non-trivial bug. Replaces ad-hoc "let me look around". |
| `/gsd-health` | Diagnoses planning directory health and optionally repairs issues. | When `.planning/` looks broken or stale. |
| `/gsd-stats` | Displays project statistics — phases, plans, requirements, git metrics, timeline. | Status check; weekly review. |
| `/gsd-forensics` | Post-mortem investigation for failed GSD workflows — diagnoses what went wrong. | When a phase failed mysteriously and you want to know why. |
| `/gsd-undo` | Safe git revert. Rolls back phase or plan commits using the phase manifest with dependency checks. | When you need to back out a phase's commits without breaking dependencies. |

---

## Speed/control utilities (8 skills)

Different speeds and shapes for the workflow.

| Skill | What it does | When |
|---|---|---|
| `/gsd-fast` | Executes a trivial task inline — no subagents, no planning overhead. | Tiny one-line edits, typo fixes. The "I just need this done now" mode. |
| `/gsd-quick` | Executes a quick task with GSD guarantees (atomic commits, state tracking) but skips optional agents. | Small task that's not trivial enough for `/gsd-fast` but not big enough for full phase workflow. |
| `/gsd-progress` | Checks progress, advances workflow, or dispatches freeform intent — the unified GSD situational command. | "What should I do next?" or "where am I in this phase?" |
| `/gsd-resume-work` | Resumes work from previous session with full context restoration. | Start of a new session in an existing project. |
| `/gsd-pause-work` | Creates context handoff when pausing work mid-phase. | End of a session, mid-phase, when you want a clean resume next time. |
| `/gsd-autonomous` | Runs all remaining phases autonomously — discuss → plan → execute per phase. | When you trust the roadmap fully and want hands-off execution. **High risk; review what it does after.** |
| `/gsd-audit-fix` | Autonomous audit-to-fix pipeline — finds issues, classifies, fixes, tests, commits. | Cleanup pass on tech debt. |
| `/gsd-manager` | Interactive command center for managing multiple phases from one terminal. | Multi-phase or multi-workstream juggling. |

---

## Context + intelligence (7 skills)

Build, query, and refresh project knowledge.

| Skill | What it does | When |
|---|---|---|
| `/gsd-map-codebase` | Analyzes codebase with parallel mapper agents to produce `.planning/codebase/` documents. | After importing a large pre-existing codebase. **Different from `/graphify`** — this writes structured analysis docs, not a graph. |
| `/gsd-graphify` | Builds, queries, and inspects the project knowledge graph in `.planning/graphs/`. | When you want a queryable graph that's GSD-aware (knows about phases, decisions, etc.). **Different from third-party `/graphify`** — see [`README.md`](README.md#three-knowledge-of-codebase-systems-coexist). |
| `/gsd-docs-update` | Generates or updates project documentation verified against the codebase. | After a phase that changed user-facing behavior. |
| `/gsd-ingest-docs` | Bootstraps or merges a `.planning/` setup from existing ADRs, PRDs, SPECs, docs in a repo. | Onboarding GSD into a project that already has scattered planning docs. |
| `/gsd-extract-learnings` | Extracts decisions, lessons, patterns, surprises from completed phase artifacts. | After shipping a phase — especially one that hit unexpected obstacles. |
| `/gsd-profile-user` | Generates developer behavioral profile and creates Claude-discoverable artifacts. | Once per developer onboarding to GSD. |
| `/gsd-import` | Ingests external plans with conflict detection against project decisions before writing anything. | When someone hands you a spec/plan from outside the project. |

---

## Capture + thread management (4 skills)

Things you collect, things that span phases.

| Skill | What it does | When |
|---|---|---|
| `/gsd-capture` | Captures ideas, tasks, notes, seeds to their destination. | Background idea capture — like writing on a sticky note. |
| `/gsd-thread` | Manages persistent context threads for cross-session work. | Multi-session investigations that don't fit a single phase. |
| `/gsd-workstreams` | Manages parallel workstreams — list, create, switch, status, progress, complete, resume. | Working on 2+ phases in parallel (rare for solo dev). |
| `/gsd-workspace` | Manages GSD workspaces — create, list, remove isolated workspace environments. | Separating multiple GSD projects on one machine. |

---

## Specialized phases (3 skills)

For phases that produce specific kinds of artifacts.

| Skill | What it does | When |
|---|---|---|
| `/gsd-ui-phase` | Generates UI design contract (`UI-SPEC.md`) for frontend phases. | Before any phase that builds UI. **Hands off to `frontend-design`** for implementation. |
| `/gsd-ai-integration-phase` | Generates an `AI-SPEC.md` design contract for phases that involve building AI systems. | Before any phase that adds an LLM call, agent, or classifier. **Hands off to `gsd-eval-review`** post-build. |
| `/gsd-spec-phase` | (Already in canonical loop, but listed here too — generates phase SPEC.md.) | First step of any new phase. |

---

## Exploration + ideation (5 skills)

Before you commit to plans.

| Skill | What it does | When |
|---|---|---|
| `/gsd-explore` | Socratic ideation and idea routing — think through ideas before committing to plans. | Earliest stage; before SPEC.md. |
| `/gsd-sketch` | Sketches UI/design ideas with throwaway HTML mockups, or proposes what to sketch next. | Visual brainstorming before `/gsd-ui-phase`. |
| `/gsd-spike` | Spikes an idea through experiential exploration, or proposes what to spike next. | Time-boxed feasibility check before committing to a phase. |
| `/gsd-review-backlog` | Reviews and promotes backlog items to active milestone. | Periodically — to keep the active scope current. |
| `/gsd-inbox` | Triages and reviews open GitHub issues and PRs against project templates and contribution guidelines. | Cleaning up PR/issue backlog. |

---

## Phase admin (3 skills)

CRUD for phases themselves.

| Skill | What it does | When |
|---|---|---|
| `/gsd-phase` | CRUD for phases in ROADMAP.md — add, insert, remove, edit phases. | When the roadmap shape changes. |
| `/gsd-pr-branch` | Creates a clean PR branch by filtering out `.planning/` commits — ready for code review. | When you need a PR without the planning-doc noise in the diff. |
| `/gsd-ultraplan-phase` | **[BETA]** Offload plan-phase to Claude Code's ultraplan cloud; review in browser; import back. | Heavy planning when you want offline cloud compute, not local context. |

---

## Settings + meta (5 skills)

| Skill | What it does | When |
|---|---|---|
| `/gsd-help` | Shows available GSD commands and usage guide. | When you forget what's available. |
| `/gsd-config` | Configures GSD settings — workflow toggles, advanced knobs, integrations, model profile. | Once per project (or once globally). Set: `mode=yolo`, `granularity=standard`, `research=yes`, `plan-check=yes`, `verifier=yes`. |
| `/gsd-settings` | Configures GSD workflow toggles and model profile. | Same family as `/gsd-config` — slightly different surface. |
| `/gsd-update` | Updates GSD to latest version with changelog display. | Periodically (npx idempotent install). |
| `/gsd-plan-review-convergence` | Cross-AI plan convergence loop — replan with review feedback until no HIGH concerns remain. | After `/gsd-plan-phase` if you want extra rigor. |

---

## Namespace dispatchers (5 skills)

Each `gsd-ns-*` is a **convenience router** — you don't memorize the underlying skills, you ask for the namespace and it dispatches. Useful when you forgot the exact skill name.

| Skill | Routes to (one of) |
|---|---|
| `/gsd-ns-workflow` | `discuss`, `plan`, `execute`, `verify`, `phase`, `progress` |
| `/gsd-ns-context` | `map`, `graphify`, `docs`, `learnings` |
| `/gsd-ns-review` | `code-review`, `debug`, `audit`, `security`, `eval`, `ui` |
| `/gsd-ns-manage` | `workstreams`, `thread`, `update`, `ship`, `inbox` |
| `/gsd-ns-project` | `milestones`, `audits`, `summary` |
| `/gsd-ns-ideate` | `explore`, `sketch`, `spike`, `spec`, `capture` |

Use the namespace when you remember the *area* but not the *exact name*. Example: *"`/gsd-ns-review` — I want to audit security on this phase"* dispatches to `gsd-secure-phase`.

---

## What you actually need to know

Most projects, you'll use ~12 skills consistently:

- **Setup once:** `/gsd-config`, `/gsd-new-project`
- **Each milestone:** `/gsd-new-milestone`, `/gsd-complete-milestone`
- **Each phase (the spine):** `/gsd-spec-phase`, `/gsd-discuss-phase`, `/gsd-plan-phase`, `/gsd-execute-phase`, `/gsd-verify-work`, `/gsd-ship`
- **As needed:** `/gsd-debug`, `/gsd-progress`, `/gsd-resume-work`, `/gsd-pause-work`

The other ~50 are situational. Don't try to memorize all of them — `/gsd-help` is a `?` away, and the namespace dispatchers (`gsd-ns-*`) catch you when you forget.
