# docs/tooling/ — reference for everything installed by SETUP.md

This folder is a thinking/lookup aid for the toolchain the template ships. It is **not** project state — it's a reference. Read it once to build a mental model; come back when you need to remember "what does X do" or "which thing should I reach for in this situation".

## Contents

| File | Covers |
|---|---|
| [`mcps.md`](mcps.md) | 3 MCP servers: `context-mode`, `context7`, `sequential-thinking` |
| [`plugins.md`](plugins.md) | 5 plugins from SETUP.md (`superpowers`, `frontend-design`, `skill-creator`, `context-mode`, `claude-mem`) + 5 extras pre-installed at user scope (`code-review`, `github`, `feature-dev`, `claude-code-setup`, `mcp-server-dev`) |
| [`gsd.md`](gsd.md) | The GSD bundle (~65 `gsd-*` skills) organized by category and phase |
| [`other-skills.md`](other-skills.md) | Utility skills installed at user scope: `update-config`, `keybindings-help`, `simplify`, `fewer-permission-prompts`, `loop`, `schedule`, `claude-api`, `init`, `review`, `security-review` |
| [`workflows.md`](workflows.md) | **Synthesis** — sequences of tools to use together across different parts of a project (day-zero, planning, executing, debugging, shipping, etc.) |

## Each entry follows the same shape

For every plugin/MCP/skill, you'll find:

1. **Category** — MCP server / Plugin / Skill bundle / Skill / Slash command
2. **What it does** — one-paragraph explanation
3. **Activation** — *automatic* (Claude triggers based on context, hook, or skill description) or *manual* (you type a slash command or explicitly ask for it)
4. **How to activate** — concrete examples (slash command, prompt phrasing, file context)
5. **When to use it** — situations where this is the right tool
6. **What to use instead** — when it's the wrong tool, what to reach for
7. **Notes / gotchas** — anything subtle about behavior, cost, or interaction with other tools

## Three activation models

The toolchain mixes three different ways things get triggered. Knowing which is which matters because it tells you whether to type something or just trust Claude to do it.

| Model | How it fires | Examples |
|---|---|---|
| **Slash command** | You type `/<name>` in the chat | `/graphify`, `/gsd-new-project`, `/ctx-stats`, `/init`, `/review` |
| **Skill auto-trigger** | Claude reads each skill's `description` frontmatter; if it matches the task, the skill is invoked via the Skill tool before any other action | `superpowers:brainstorming` (any creative work), `superpowers:test-driven-development` (any implementation), `frontend-design:frontend-design` (any UI request), `gsd-debug` (any reported bug) |
| **MCP tool / hook** | Claude calls an MCP tool when relevant, OR a hook fires on an event (PreToolUse, SessionStart, etc.) | `mcp__context7__get-library-docs` (when needing current docs), `mcp__plugin_context-mode_context-mode__ctx_batch_execute` (when running shell commands that may produce large output), claude-mem hooks (passive, on session lifecycle) |

You only really need to **type** slash commands. The other two models fire on Claude's judgment — though you can always nudge with phrasing like *"use sequential-thinking for this"* or *"check context7 for the current Next.js routing API"*.

## Three "knowledge of codebase" systems coexist

Worth flagging up-front because it's the most confusing thing in the toolchain:

| System | Stores at | Built by | Best at |
|---|---|---|---|
| **`/graphify`** (third-party) | `graphify-out/` | Manual `/graphify .` (LLM-driven, expensive first run) | Cross-cluster relationships, multimodal input, Obsidian visual map |
| **`gsd-graphify`** (built into GSD) | `.planning/graphs/graph.json` | GSD workflow step | Queryable graph keyed to GSD phases (BFS expand, diff over time) |
| **`context-mode`** | FTS5 index (in-memory per session, persisted in sandbox) | Auto, every batched shell command | Live keyword search across recent tool outputs |

They don't fight. Pick the one matching the question you're asking. See [`workflows.md`](workflows.md#picking-a-codebase-knowledge-system) for guidance.

## Where to next

- New to the toolchain → read in this order: `mcps.md` → `plugins.md` → `gsd.md` → `workflows.md`. Skip `other-skills.md` until you need a specific utility.
- Looking up a specific tool → use the table above to find the right file.
- "What should I do next in my project?" → go straight to [`workflows.md`](workflows.md).
