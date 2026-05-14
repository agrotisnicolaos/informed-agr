# Plugin & Skill Setup — Next Project

Hand this file to Claude on day one of a new repo. It tells Claude which Claude Code plugins, marketplaces, and user-skill bundles to install, and in what order.

> **Audience:** Claude in a fresh repo.
> **Goal:** End state = same skills + plugins available as in the previous Demetra iOS project.

---

## 0. Prereqs (verify, do not assume)

Before installing anything, check what is already on the machine:

```bash
ls ~/.claude/plugins/installed_plugins.json 2>/dev/null && cat ~/.claude/plugins/installed_plugins.json | python3 -c "import sys,json; [print(k) for k in json.load(sys.stdin)['plugins'].keys()]"
ls ~/.claude/skills/ 2>/dev/null | head
ls ~/.claude/get-shit-done/VERSION 2>/dev/null
```

Anything already present at the **user scope** (not just project scope) is reusable — do not reinstall.

---

## 1. Install marketplaces

In Claude Code, run:

```
/plugin marketplace add context-mode/context-mode
/plugin marketplace add thedotmack/claude-mem
```

The first marketplace (`claude-plugins-official`) is pre-registered — no action needed for plugins that live in it.

---

## 2. Install plugins (user scope)

Install at user scope so the plugin follows you across all repos.

```
/plugin install superpowers@claude-plugins-official
/plugin install frontend-design@claude-plugins-official
/plugin install skill-creator@claude-plugins-official
/plugin install context-mode@context-mode
```

After each install, restart Claude Code if prompted, then verify the plugin's skills show up in the available-skills list.

### 2a. Install claude-mem (special — do NOT use `npm install -g`)

claude-mem has two install paths. **Pick one.** Do not mix.

**Option 1 (recommended) — single command outside Claude Code:**

```bash
npx claude-mem install
```

This registers plugin hooks, sets up the worker service, and writes config to `~/.claude/`. After running, restart Claude Code. Context from previous sessions will automatically appear in new sessions.

**Option 2 — Claude Code plugin marketplace flow (matches what other plugins do):**

```
/plugin marketplace add thedotmack/claude-mem
/plugin install claude-mem
```

Restart Claude Code afterwards.

**Do NOT do this:**

```
npm install -g claude-mem      # ⚠️ installs SDK/library only — no hooks, no worker
```

`npm install -g` installs only the JavaScript library. It does not register the plugin or start the memory worker. Symptoms if you do this by mistake: claude-mem appears in `node_modules` but no cross-session memory shows up in new Claude sessions, and `~/.claude/plugins/installed_plugins.json` has no entry for it. Fix by running `npx claude-mem install` (which is idempotent and will register correctly on top of the npm install).

**Verify install:**

```bash
# Plugin install (Option 2) shows up here:
cat ~/.claude/plugins/installed_plugins.json | grep -i claude-mem

# Either install registers hooks under ~/.claude/:
ls ~/.claude/ | grep -i claude-mem
# Or check that the marketplace dir exists:
ls ~/.claude/plugins/marketplaces/thedotmack/

# Worker / web viewer (Option 1 spins up a local viewer):
curl -s http://localhost:37777/ -o /dev/null && echo "viewer up" || echo "viewer not running"
```

Node ≥18 required. As of this writing, claude-mem version is 6.5.0.

### What each plugin gives you

| Plugin | Purpose | Key skills it adds |
|---|---|---|
| `superpowers` | Discipline skills: TDD, brainstorming, plan-writing, code review, debugging, parallel-agent dispatch | `superpowers:brainstorming`, `superpowers:writing-plans`, `superpowers:executing-plans`, `superpowers:test-driven-development`, `superpowers:systematic-debugging`, `superpowers:verification-before-completion`, `superpowers:requesting-code-review`, `superpowers:receiving-code-review`, `superpowers:using-git-worktrees`, `superpowers:dispatching-parallel-agents`, `superpowers:subagent-driven-development`, `superpowers:writing-skills`, `superpowers:finishing-a-development-branch`, `superpowers:using-superpowers` |
| `frontend-design` | Production-grade frontend code that doesn't look "AI-generic" | `frontend-design:frontend-design` |
| `skill-creator` | Create, edit, eval, and optimize custom skills | `skill-creator:skill-creator` |
| `context-mode` | Keep large tool outputs out of Claude's context window. Adds an MCP server (`mcp__plugin_context-mode_context-mode__*`) for batched shell/queries that get auto-indexed and FTS5-searched. Adds `/ctx-stats`, `/ctx-purge`, `/ctx-doctor`, `/ctx-upgrade`, `/ctx-insight` | `context-mode:context-mode`, `context-mode:diagnose`, `context-mode:tdd`, `context-mode:grill-me`, `context-mode:grill-with-docs`, `context-mode:improve-codebase-architecture`, `context-mode:context-mode-ops` |
| `claude-mem` | Cross-session memory (recall what was discussed in previous sessions) | n/a — runs in the background via hooks |

---

## 3. Install the GSD (Get-Shit-Done) workflow

GSD is not a Claude Code marketplace plugin — it is a separate skill bundle distributed via npm that installs under `~/.claude/get-shit-done/` and `~/.claude/skills/gsd-*`.

**Install (user scope, follows you across all repos):**

```bash
npx get-shit-done-cc --claude --global
```

Flags:
- `--claude` — installs for Claude Code (other IDEs supported via different flags)
- `--global` — user-scope install so it follows across all repos. Drop this flag for project-scope install if you ever want a per-repo version.

**Verify install:**

```bash
cat ~/.claude/get-shit-done/VERSION 2>/dev/null   # should print a version like 1.40.0
ls ~/.claude/skills/ | grep '^gsd-' | wc -l       # should print ~70
```

If the VERSION file already exists, the bundle is installed. Re-running `npx get-shit-done-cc --claude --global` is the upgrade path; it is idempotent.

### What GSD gives you

A full workflow for phased projects: new project → milestones → phases → discuss → plan → execute → verify → ship. Key skills it adds (~70 total, prefixed `gsd-`):

- **Project lifecycle:** `gsd-new-project`, `gsd-new-milestone`, `gsd-complete-milestone`, `gsd-milestone-summary`
- **Phase workflow:** `gsd-spec-phase`, `gsd-discuss-phase`, `gsd-plan-phase`, `gsd-execute-phase`, `gsd-verify-work`, `gsd-ship`
- **Quality gates:** `gsd-code-review`, `gsd-secure-phase`, `gsd-eval-review`, `gsd-ui-review`, `gsd-validate-phase`, `gsd-audit-uat`, `gsd-audit-milestone`
- **Diagnostics:** `gsd-health`, `gsd-stats`, `gsd-forensics`, `gsd-debug`
- **Utility:** `gsd-fast`, `gsd-quick`, `gsd-progress`, `gsd-resume-work`, `gsd-pause-work`, `gsd-capture`, `gsd-thread`, `gsd-workstreams`
- **Docs/context:** `gsd-docs-update`, `gsd-map-codebase`, `gsd-graphify`, `gsd-ingest-docs`, `gsd-extract-learnings`

Run `/gsd-help` for the current full list.

### Recommended GSD config for a new solo-founder project

```
mode = yolo
granularity = standard
research = yes
plan-check = yes
verifier = yes
```

Set via `/gsd-config` after install.

---

## 4. Project-level setup (after plugins are installed)

Once the plugins are in, do these in the new repo's `.claude/` directory:

### 4a. Drop in the 6 domain agents

Copy the 6 files from `learnings-for-next-project/agents/` into `.claude/agents/`. Then update each agent's "First instruction" doc list to point at the actual docs in the new repo (they are placeholders like `docs/product-context.md` — keep this naming convention so the agents work without further edits).

### 4b. Create the canonical doc set the agents read

Every domain agent reads these five docs on every invocation. Create them in `docs/` on day one, even if mostly empty:

1. `docs/product-context.md` — what the product is, who the user is, current stage
2. `docs/constraints.md` — hard constraints, architectural rules
3. `docs/phasing.md` — phase framing (1 / 1.5 / 2 / 3) with reversibility tags
4. `docs/budget.md` — $0/mo ceiling, paid-recommendation approval flow
5. `docs/decisions.md` — locked decisions log (D-001+), append-only

A `CLAUDE.md` at repo root that points agents at these five docs and at the agent-routing table is the single most load-bearing setup file. Copy the routing table from the previous project's `CLAUDE.md` and adjust.

### 4c. Initialize GSD project

```
/gsd-new-project
```

Answer the deep-context questions honestly — it produces `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md` which the GSD workflow keys off of.

### 4d. (Optional) Map the codebase

If there's pre-existing code in the new repo:

```
/gsd-map-codebase
```

Writes parallel analyses to `.planning/codebase/`.

---

## 5. Verify

After all of the above:

```bash
ls -la .claude/agents/         # should show 6 .md files
ls -la docs/                    # should show the 5 canonical docs
cat CLAUDE.md                   # should reference the 5 docs + routing table
```

In Claude Code, type `/` and confirm you see `gsd-*`, `superpowers:*`, `context-mode:*`, `frontend-design:*`, and `skill-creator:*` skills listed.

---

## 6. Day-one workflow

1. `/gsd-new-project` — establish PROJECT.md
2. Fill in `docs/product-context.md`, `docs/constraints.md`, `docs/phasing.md`, `docs/budget.md` (start `docs/decisions.md` empty)
3. `/gsd-new-milestone` — establish the first milestone
4. `/gsd-spec-phase 1` → `/gsd-discuss-phase 1` → `/gsd-plan-phase 1` → `/gsd-execute-phase 1` → `/gsd-verify-work 1` → `/gsd-ship`

That sequence is the durable pattern. Everything else is variation.
