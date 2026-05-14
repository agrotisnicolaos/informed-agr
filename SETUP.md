# Plugin & Skill Setup — informed-agr template

Hand this file to Claude on day one of a new repo spawned from this template. It tells Claude which Claude Code plugins, marketplaces, MCP servers, and user-skill bundles to install, and in what order.

> **Audience:** Claude in a fresh repo that was created via "Use this template" from `agrotisnicolaos/informed-agr`.
> **Goal:** End state = same skills, plugins, and MCP servers available as in the reference setup (Demetra iOS distillation).

---

## 0. Prereqs (verify, do not assume)

Before installing anything, check what is already on the machine:

```bash
ls ~/.claude/plugins/installed_plugins.json 2>/dev/null && cat ~/.claude/plugins/installed_plugins.json | python3 -c "import sys,json; [print(k) for k in json.load(sys.stdin)['plugins'].keys()]"
ls ~/.claude/skills/ 2>/dev/null | head
ls ~/.claude/get-shit-done/VERSION 2>/dev/null
claude mcp list 2>/dev/null
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
cat ~/.claude/plugins/installed_plugins.json | grep -i claude-mem
ls ~/.claude/ | grep -i claude-mem
ls ~/.claude/plugins/marketplaces/thedotmack/
curl -s http://localhost:37777/ -o /dev/null && echo "viewer up" || echo "viewer not running"
```

Node ≥18 required.

### 2b. What each plugin gives you

| Plugin | Purpose |
|---|---|
| `superpowers` | Discipline skills: TDD, brainstorming, plan-writing, code review, debugging, parallel-agent dispatch |
| `frontend-design` | Production-grade frontend code that doesn't look "AI-generic" |
| `skill-creator` | Create, edit, eval, and optimize custom skills |
| `context-mode` | Keep large tool outputs out of Claude's context window. Adds an MCP server (`mcp__plugin_context-mode_context-mode__*`) for batched shell/queries that get auto-indexed and FTS5-searched. Adds `/ctx-stats`, `/ctx-purge`, `/ctx-doctor`, `/ctx-upgrade`, `/ctx-insight` |
| `claude-mem` | Cross-session memory (recall what was discussed in previous sessions) |

### 2c. Install Context7 (required MCP)

Context7 is an MCP server that provides up-to-date library/framework docs to Claude. **GSD's planner, executor, and researcher agents already call `mcp__context7__*` tools** (`resolve-library-id`, `get-library-docs`) in 9+ workflow files. Without Context7 installed, those calls silently fail and GSD degrades.

**Install:** follow the upstream README at https://github.com/upstash/context7 for the current command. Install paths shift; do not hardcode. Typical patterns:

```bash
# Option A — interactive setup with OAuth
npx ctx7 setup

# Option B — manual MCP server registration
claude mcp add context7 -- npx -y @upstash/context7-mcp@latest
```

**Verify install:**

```bash
claude mcp list | grep context7
```

After install, GSD's `mcp__context7__*` tool calls will resolve correctly.

### 2d. Install Graphify + Obsidian (required)

**Graphify** builds a JSON+HTML knowledge graph of the codebase (`graphify-out/`) so Claude can answer "how does X relate to Y" without re-reading files every session. Multimodal: also indexes images. Inspired by Andrej Karpathy's "raw folder" pattern.

**Obsidian** is a standalone GUI app that opens `graphify-out/` as a vault for visual inspection of the graph (no Claude integration; purely for the human).

**Install Graphify:** follow https://github.com/safishamsi/graphify for the current command. Then run `/graphify` once per project — produces `graphify-out/{graph.json, graph.html}`.

**Install Obsidian:** download from https://obsidian.md/download. After first `/graphify` run, open `graphify-out/` as a vault: Obsidian → Open folder as vault → select `graphify-out/`.

**Overlap note (important):** three "codebase knowledge" systems will coexist in this template, and that's intentional:

| System | Stores in | Strength |
|---|---|---|
| `gsd-graphify` (GSD built-in) | `.planning/graphs/graph.json` | Queryable graph for the GSD workflow (BFS expand, diff over time) |
| Third-party `/graphify` | `graphify-out/` | Multimodal input + Obsidian visual graph |
| `context-mode` | FTS5 index of recent tool outputs | Live keyword search across batched commands |

Different stores, different file paths — they don't fight. Pick the one that matches the question you're asking.

### 2e. Install Sequential Thinking MCP (required)

An MCP server that gives Claude a callable `sequentialthinking` tool — emits explicit step records as tool output. Distinct from Claude's native extended thinking, which stays inside the model. Use sequential-thinking when you want the chain of reasoning **recorded** in tool output (audit trail, post-hoc inspection, or for hard branching problems where Claude needs to revisit prior steps).

**Install:** follow https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking for the current command. Typical:

```bash
claude mcp add sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
```

**Verify install:**

```bash
claude mcp list | grep sequential
```

---

## 3. Install the GSD (Get-Shit-Done) workflow

GSD is not a Claude Code marketplace plugin — it is a separate skill bundle distributed via npm that installs under `~/.claude/get-shit-done/` and `~/.claude/skills/gsd-*`.

**Install (user scope, follows you across all repos):**

```bash
npx get-shit-done-cc --claude --global
```

Flags:
- `--claude` — installs for Claude Code
- `--global` — user-scope install so it follows across all repos. Drop this flag for project-scope.

**Verify install:**

```bash
cat ~/.claude/get-shit-done/VERSION 2>/dev/null
ls ~/.claude/skills/ | grep '^gsd-' | wc -l
```

Re-running is idempotent (acts as the upgrade path).

### What GSD gives you

A full workflow for phased projects: new project → milestones → phases → discuss → plan → execute → verify → ship. ~65 skills prefixed `gsd-`. Run `/gsd-help` for the current full list.

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

### 4a. Domain agents (already in this template)

The 6 domain agents are already at `.claude/agents/` in this template:
- `ceo.md` — strategic owner, scope/budget vetoes
- `tech-lead.md` — coordinator, decisions.md upkeep, doc-drift guard
- `system-designer.md` — caching, offline state machines, Free-tier realities
- `security.md` — RLS, auth, secrets, local-cache safety
- `data-architect.md` — Postgres schema, migrations, RPC design
- `ios-builder.md` — Swift/SwiftUI implementation. **If your project isn't iOS, follow [`.claude/agents/RENAME-IF-NOT-iOS.md`](.claude/agents/RENAME-IF-NOT-iOS.md).**

### 4b. Canonical doc set (already scaffolded in this template)

Every domain agent reads these five docs on every invocation. They're scaffolded with TEMPLATE markers — fill them in during `/gsd-new-project`:

1. `docs/product-context.md`
2. `docs/constraints.md`
3. `docs/phasing.md`
4. `docs/budget.md`
5. `docs/decisions.md`

`CLAUDE.md` at repo root references these five and provides the agent-routing table. Keep it in sync as the project evolves.

### 4c. Initialize GSD project

```
/gsd-new-project
```

Answer the deep-context questions honestly — produces `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`.

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
ls -la .claude/agents/         # should show 6 .md files + RENAME-IF-NOT-iOS.md
ls -la docs/                    # should show the 5 canonical docs
cat CLAUDE.md                   # should reference the 5 docs + routing table
claude mcp list                 # should include context7 and sequential-thinking
```

In Claude Code, type `/` and confirm you see `gsd-*`, `superpowers:*`, `context-mode:*`, `frontend-design:*`, `skill-creator:*`, and `graphify` skills/commands listed.

---

## 6. Day-one workflow

1. `/gsd-new-project` — establish PROJECT.md
2. Fill in `docs/product-context.md`, `docs/constraints.md`, `docs/phasing.md`, `docs/budget.md` (start `docs/decisions.md` empty)
3. `/graphify` — build the codebase knowledge graph (one-time per project; rerun after major changes)
4. `/gsd-new-milestone` — establish the first milestone
5. `/gsd-spec-phase 1` → `/gsd-discuss-phase 1` → `/gsd-plan-phase 1` → `/gsd-execute-phase 1` → `/gsd-verify-work 1` → `/gsd-ship`

That sequence is the durable pattern. Everything else is variation.
