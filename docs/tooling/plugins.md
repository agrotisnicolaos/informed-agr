# Plugins

Ten plugins are installed at user scope on this machine. Five are required by SETUP.md (`superpowers`, `frontend-design`, `skill-creator`, `context-mode`, `claude-mem`). Five are extras that came pre-installed (`code-review`, `github`, `feature-dev`, `claude-code-setup`, `mcp-server-dev`). The extras are useful but not template-critical.

User-scope = the plugin follows you across every repo on this machine. Verify install: `cat ~/.claude/plugins/installed_plugins.json`.

---

## REQUIRED PLUGINS (from SETUP.md)

### 1. superpowers

**Category:** Plugin (bundle of 14 process/discipline skills).

**What it does.** Bakes "good engineering process" into Claude's defaults. Each skill is a workflow Claude follows when its description matches the task. Most are auto-triggered by phrasing; you rarely invoke them by name.

**Activation:** **Automatic.** Each skill has a `description` frontmatter that tells Claude when to use it. Claude reads them all on session start and dispatches when the situation matches.

**Skills it adds:**

| Skill | Auto-trigger when… | What it does |
|---|---|---|
| `using-superpowers` | Start of every conversation | Establishes how to find and use skills (loaded for you in your first turn) |
| `brainstorming` | Any creative work — "build X", "add Y feature", "what should we do about Z" | Forces requirement exploration before implementation. Stops me from coding-first |
| `writing-plans` | You have a spec/requirements for a multi-step task, before code | Produces a written plan before touching code |
| `executing-plans` | You have a written plan to execute in a separate session with review checkpoints | Walks the plan with checkpoints |
| `subagent-driven-development` | Executing implementation plans with independent tasks in current session | Dispatches multiple subagents in parallel |
| `dispatching-parallel-agents` | 2+ independent tasks without shared state or sequential dependencies | Parallel agent fan-out |
| `test-driven-development` | Implementing any feature or bugfix, before writing implementation | Red-green-refactor enforcement |
| `systematic-debugging` | Encountering any bug, test failure, or unexpected behavior, before proposing fixes | Scientific-method bug investigation |
| `verification-before-completion` | About to claim work is done/passing, before commits or PRs | Forces "evidence before assertion" — run the verify command, paste the output |
| `requesting-code-review` | Completing tasks, before merging | Self-review checklist before asking another agent |
| `receiving-code-review` | Receiving feedback, before implementing suggestions | Push-back-with-rigor instead of performative agreement |
| `using-git-worktrees` | Starting feature work that needs isolation, or before executing implementation plans | Sets up an isolated workspace |
| `finishing-a-development-branch` | Implementation complete, tests pass, deciding how to integrate | Structured options for merge/PR/cleanup |
| `writing-skills` | Creating new skills, editing skills, or verifying they work | Meta-skill for authoring |

**How to nudge a specific skill:**
- *"Brainstorm this with me before we plan."* → triggers `brainstorming`
- *"Use TDD on this."* → triggers `test-driven-development`
- *"Debug this systematically — don't just guess."* → triggers `systematic-debugging`
- *"Verify before you tell me it works."* → triggers `verification-before-completion`

**When to use it:** Most non-trivial work. The skills bias toward discipline (plan-first, test-first, verify-first) which is the right default for solo-founder velocity over "feels productive but ships bugs".

**When NOT to use it:** Trivial one-line edits, throwaway exploration. Claude usually figures out when to skip a skill, but you can override with *"skip the planning step on this"*.

**Gotchas:**
- Some skills are **rigid** (TDD, debugging) — Claude shouldn't adapt them away. Others are **flexible** (patterns, naming).
- User CLAUDE.md instructions override skill behavior. If you write "don't use TDD on quick scripts," that wins.
- The skills genuinely do what their descriptions say. If you say "let's just code this fast" Claude may still do brainstorming first — push back if you want a true skip.

---

### 2. frontend-design

**Category:** Plugin (1 skill).

**What it does.** When you ask Claude to build UI, this skill kicks in to push for distinctive, production-grade design instead of generic AI-generated aesthetics. It biases the output toward variety in layout, opinionated typography choices, real spacing/density decisions — not just "card grid with rounded corners and a hero".

**Activation:** **Automatic** when you ask for a web component, page, or app. Triggered by phrasing like "build a landing page", "create a dashboard", "design a form".

**How to nudge it:** Already triggers on UI requests. Force it: *"Use the frontend-design skill — I want this to feel hand-crafted, not AI-generic."*

**When to use it:** Any UI you're shipping (or sketching). Especially marketing pages, dashboards, forms — anywhere "looks good" matters.

**When NOT to use it:** Internal admin tools, throwaway prototypes, server-side code, mobile UI (iOS/Android — different skill territory; this is web-focused).

**Gotchas:**
- Iterates on aesthetic decisions. Don't expect first-shot perfection — give it feedback.
- Plays well with `gsd-ui-phase` and `gsd-sketch` (those handle the spec; frontend-design handles the build).

---

### 3. skill-creator

**Category:** Plugin (1 skill).

**What it does.** Meta-tool for authoring, editing, evaluating, and optimizing your own custom skills. When you want to add a new skill (e.g. a project-specific workflow), this skill walks you through the structure (frontmatter, description, body), runs evals to test trigger accuracy, and benchmarks performance with variance analysis.

**Activation:** **Manual.** Invoke when:
- *"Create a new skill that does X."*
- *"Optimize the description on my existing skill — it's not triggering when it should."*
- *"Test my skill against these scenarios."*

**How to use it:** Just say what you want. Examples:
- *"Make a skill that captures all my Linear ticket numbers from a conversation and posts a summary to a Slack channel."*
- *"Improve the trigger description on `~/.claude/skills/my-deploy/SKILL.md` — it's only firing when I say 'deploy' but should also fire on 'ship', 'release', 'push to prod'."*
- *"Eval my skill across 10 sample prompts and tell me false-positive rate."*

**When to use it:** Any time you find yourself doing the same multi-step thing across sessions/projects. If you've explained the same workflow 3+ times, build a skill.

**When NOT to use it:** One-off tasks. Don't pre-build skills speculatively — wait until the third repetition.

**Gotchas:**
- Skills live at `~/.claude/skills/<name>/SKILL.md` (user scope) or `.claude/skills/<name>/SKILL.md` (project scope).
- The `description` field is load-bearing — it's how Claude decides when to dispatch your skill.
- Evals are useful for non-obvious trigger phrasing — don't skip them on skills you'll rely on.

---

### 4. context-mode

Covered in [`mcps.md`](mcps.md#1-context-mode) (the plugin bundles the MCP server + skills + slash commands). Highlights:

- 7 skills auto-triggered by their descriptions: `context-mode`, `diagnose`, `tdd`, `grill-me`, `grill-with-docs`, `improve-codebase-architecture`, `context-mode-ops`
- 5 slash commands: `/ctx-stats`, `/ctx-doctor`, `/ctx-purge`, `/ctx-upgrade`, `/ctx-insight`
- The MCP server fires automatically via PreToolUse hook — keeps tool output out of the chat

---

### 5. claude-mem

**Category:** Plugin (no skills — runs entirely via background hooks + an MCP search server).

**What it does.** Cross-session memory. As you work, hooks index conversation summaries to a local database (`~/.claude-mem`). On a new session start in the same project, prior context is recalled — so you don't have to re-explain what you were doing last week.

**Status on this machine:** Installed and registered, but **`auto-memory disabled`** via `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` env var. The plugin is present, the MCP search server connects (`plugin:claude-mem:mcp-search ✓ Connected`), but no automatic recall is happening on session starts.

**Activation:** **Automatic when enabled.** Hooks fire on session lifecycle events. The MCP server (`mcp__claude_mem_*`) is callable any time for explicit search.

**How to use it explicitly:**
- *"Search claude-mem for what we decided about Supabase last week."*
- Open the local viewer: `npx claude-mem start` (worker) → http://localhost:37701
- *"Look up prior conversations about [topic] in claude-mem."*

**When to use it:**
- Cross-session continuity — picking up where you left off in any project on this machine
- Recalling decisions made in a different repo on the same topic
- Onboarding to your own work after time away

**When NOT to use it:**
- Sensitive content you don't want indexed locally
- Sessions you genuinely want to be standalone

**Gotchas:**
- **Currently disabled.** To enable: `unset CLAUDE_CODE_DISABLE_AUTO_MEMORY` in your shell rc, restart Claude Code.
- The worker (web viewer at http://localhost:37701) is **not auto-started** on this machine. Run `npx claude-mem start` in a terminal to bring it up.
- Distinct from the per-project memory in `/Users/nicolasagrotis/.claude/projects/<slug>/memory/` — that's the [auto-memory system](#) maintained by Claude Code itself, separate from claude-mem.
- Don't confuse with `superpowers:writing-plans` or `gsd-thread` — those are *intentional* persistence; claude-mem is *automatic* recall.

---

## EXTRA PLUGINS (pre-installed, not in SETUP.md)

These came installed before this template existed. They're useful but not required by the kit's day-one workflow. I won't go as deep — just enough to know when to reach for them.

### 6. code-review

**Adds:** `/review` slash command, `/security-review` slash command. Reviews a pull request or pending changes for bugs, code quality, security issues. Distinct from `gsd-code-review` which targets a specific phase's source files.

**When to use:** Before opening a PR, or to do a quick second-pass on uncommitted changes. `gsd-code-review` is more thorough for phase work; `/review` is faster for ad-hoc.

### 7. github

**Adds:** GitHub-flavored helpers. Some overlap with the `gh` CLI — convenience layer.

**When to use:** When you want Claude to handle a GitHub task (issue triage, PR comment, label) and the `gh` CLI command isn't obvious.

### 8. feature-dev

**Adds:** Feature-development scaffolding helpers — quick starters for common patterns.

**When to use:** Less relevant when you have GSD's full phase workflow. Probably skip in favor of `gsd-spec-phase` → `gsd-plan-phase` → `gsd-execute-phase`.

### 9. claude-code-setup

**Adds:** `/init` slash command — initializes a new CLAUDE.md for a project by analyzing the codebase. Useful for greenfield repos that don't have a CLAUDE.md yet.

**When to use:** A new project where you want a baseline CLAUDE.md auto-generated from the code. **Skip in this template** — we already have a hand-tuned CLAUDE.md.

### 10. mcp-server-dev

**Adds:** Helpers for building your own MCP servers (testing harness, scaffolding).

**When to use:** Only when you're authoring an MCP server. Edge case for most projects.

---

## Cross-plugin interactions

- **superpowers + GSD:** Heavy overlap. Both push discipline (TDD, plan-first). GSD wraps superpowers — when you run `/gsd-execute-phase`, it internally invokes `superpowers:test-driven-development` and `superpowers:verification-before-completion`. You don't need to invoke superpowers directly during a GSD phase.
- **frontend-design + GSD:** `/gsd-ui-phase` produces a `UI-SPEC.md` design contract. `frontend-design` then implements against that spec. Use `/gsd-ui-review` to retroactively audit.
- **claude-mem + auto-memory:** Both record things across sessions but at different granularities. claude-mem is conversation summaries; auto-memory is structured facts (`feedback_*.md`, `project_*.md`). They complement each other.
- **skill-creator + writing-skills (superpowers):** Same target (authoring skills); skill-creator is the heavy meta-tool, `superpowers:writing-skills` is a leaner discipline checklist. Use the latter for quick edits, the former for new-from-scratch.
- **context-mode + everything:** Background — every batched shell call goes through context-mode's MCP. You don't think about it; it's just on.
