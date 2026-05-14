# MCP servers

Three MCP servers ship with this template. They're separate from plugins and skills — MCP servers expose **tools** Claude calls during a session, not skills you trigger. You generally don't activate them directly; you let Claude call the tools when relevant.

Verify which are connected: `claude mcp list`.

---

## 1. context-mode

**Category:** MCP server + plugin (the plugin bundles the server with skills + slash commands).

**What it does.** Keeps large tool outputs out of Claude's context window. When Claude runs a shell command that would produce more than ~20 lines of output, the output goes into an FTS5 search index inside a sandbox instead of into the conversation. Claude then queries the index for just the parts it needs. Same idea for fetched URLs — converted to markdown, indexed, searchable.

**Why it matters.** Without context-mode, a single `git log` or `npm test` or large `find` can dump thousands of tokens into the conversation, evicting earlier context. With context-mode, that same command leaves a 1-line summary + a searchable index — and Claude pulls only what's relevant.

**Activation:** **Automatic.** A PreToolUse hook intercepts most Bash/WebFetch calls and routes them through the MCP server. You see this in the chat as `mcp__plugin_context-mode_context-mode__ctx_batch_execute` / `ctx_search` / `ctx_fetch_and_index` calls instead of raw `Bash`. You don't have to do anything.

**How to use it manually:**

| Slash command | What it does |
|---|---|
| `/ctx-stats` | Shows context-window savings this session (token consumption, savings ratio, per-tool breakdown) |
| `/ctx-doctor` | Runs diagnostics — checks runtimes, hooks, FTS5, plugin registration, npm + marketplace versions |
| `/ctx-upgrade` | Updates context-mode from GitHub, fixes hooks/settings (currently outdated → 1.0.126 available) |
| `/ctx-purge` | Wipes the knowledge base. Destructive; cannot be undone |
| `/ctx-insight` | Opens analytics dashboard (session activity, parallel work patterns, project focus) |

**Skills the plugin adds:** `context-mode`, `context-mode:diagnose`, `context-mode:tdd`, `context-mode:grill-me`, `context-mode:grill-with-docs`, `context-mode:improve-codebase-architecture`, `context-mode:context-mode-ops`. These are auto-triggered by their descriptions:
- `diagnose` — "diagnose this", "debug this", reported bug, performance regression
- `grill-me` — "grill me", "stress-test my plan"
- `grill-with-docs` — same as grill-me but cross-checks against your CONTEXT.md / ADRs
- `improve-codebase-architecture` — "improve architecture", "find refactoring opportunities", "make this more testable"
- `context-mode-ops` — meta: managing the context-mode project itself (issues, PRs, releases)
- `tdd` — same red-green-refactor as superpowers:test-driven-development but routed through context-mode

**When to use it:** Always. It's invisibly active. The only reason to think about it is when (a) you want to see how much it's saving you (`/ctx-stats`) or (b) it's misbehaving (`/ctx-doctor`).

**When NOT to use it:** If you're piping a small command (<20 lines) and want the output verbatim in chat — sometimes you just want `git status --short` to be visible. context-mode usually still routes through but the summary is fine.

**Gotchas:**
- The knowledge base survives `/clear` and `/compact`. Use `/ctx-purge` if you genuinely want to start fresh.
- Some hooks reject WebFetch and route to `ctx_fetch_and_index` instead — fine, just means you call a different function. Same outcome.

---

## 2. context7

**Category:** MCP server (HTTP transport).

**What it does.** On-demand fetcher for up-to-date library/framework documentation. Claude's training has a cutoff date. When the planner needs to know how Next.js routing works *as of today*, or how a Supabase RPC signature changed last month, it calls Context7 instead of trusting its training data. The server resolves a library name → fetches current docs → returns relevant excerpts.

**Why it matters.** This is a hard dependency for GSD. Nine-plus GSD workflow files (`gsd-planner`, `gsd-executor`, `gsd-phase-researcher`, `gsd-ai-researcher`, `gsd-project-researcher`, `gsd-ui-researcher`, `gsd-roadmapper`, `gsd-advisor-researcher`, etc.) call `mcp__context7__*` tools. Without context7 installed, those calls fail silently and GSD planning degrades — Claude makes assumptions about libraries that might be 1–2 years out of date.

**Activation:** **Automatic.** Claude calls `mcp__context7__resolve-library-id` to map "react-router" → a stable library ID, then `mcp__context7__get-library-docs` to fetch current docs. Triggered when:
- A GSD planner needs current API surface for a library it's about to use
- You ask "how do I do X in [framework] as of today?"
- You explicitly say "check context7 for…"

**How to use it manually (prompt-level, not tool-level):**
- *"What's the current Next.js App Router pattern for parallel routes? Use context7."*
- *"Look up the latest Supabase Edge Functions invoke API via context7."*
- *"Check context7 for the current SwiftUI ScrollView APIs."*

**When to use it:** Any time you need confidence that the API you're recommending is current. Especially for fast-moving frameworks (Next.js, React, Supabase, modern AI SDKs).

**When NOT to use it:** Slow-moving or stable libraries (POSIX, Postgres core, standard library). The cost (latency, token overhead) isn't worth it when training data is already accurate.

**Gotchas:**
- Default install (HTTP transport, free tier) has rate limits. Heavy use → switch to local stdio variant or get an API key from https://context7.com.
- Returns Markdown excerpts, not full docs. For exhaustive reading, `mcp__plugin_context-mode_context-mode__ctx_fetch_and_index` on the canonical URL is sometimes a better fit.
- Library coverage varies. If `resolve-library-id` returns nothing useful, fall back to context-mode's URL fetch.

---

## 3. sequential-thinking

**Category:** MCP server (stdio transport).

**What it does.** Adds a callable `sequentialthinking` tool that lets Claude emit explicit numbered thinking steps as **tool output** — visible in the chat, not hidden in the model's `<thinking>` block. Each step can be revised, branched, or extended. The model decides when to invoke it.

**Why it matters.** Native extended thinking happens inside the model and isn't always visible. For hard problems where you want an audit trail — to verify reasoning later, to branch and compare paths, or to give a stakeholder a paper trail — sequential-thinking externalizes the chain of thought as durable output.

**Activation:** **Automatic, but conservative.** Claude only invokes the tool when it judges the problem warrants explicit step tracking. You can nudge it.

**How to nudge it:**
- *"Use sequential-thinking to break this down."*
- *"Walk through this step by step using the sequentialthinking tool — I want to see your reasoning."*
- *"Think about this carefully with sequentialthinking; explore at least two alternative approaches."*

**When to use it:**
- **Hard, branching problems** where you might explore multiple solutions and want to compare them
- **Audit-required reasoning** — when a stakeholder needs to inspect *why* a decision was made (compliance, post-mortem, contested call)
- **Plan validation** — give a draft plan, ask "use sequential-thinking to find weaknesses in steps 3, 5, 7"
- **Multi-hypothesis debugging** — when there are 3+ candidate root causes and you want each evaluated separately

**When NOT to use it:**
- Simple problems — adds noise without adding signal
- Speed-critical tasks — extra tool calls add latency
- Tasks where native extended thinking is sufficient (most code edits, straightforward implementations)

**Gotchas:**
- Output is verbose. Each thought becomes a tool result. For a 10-step decomposition you'll see 10 tool calls.
- Doesn't replace `superpowers:systematic-debugging` or `gsd-debug` — those are *workflows* with state. sequential-thinking is a single-call reasoning aid.
- Distinct from Claude's internal extended thinking (the `<thinking>` blocks). Both can be active in the same session.

---

## Cross-MCP interactions

- **context-mode + context7:** When context7 returns large doc chunks, context-mode auto-indexes them so subsequent searches stay cheap. They compose well.
- **sequential-thinking + GSD:** Phase planning often benefits from sequential-thinking — *"Plan this phase using sequentialthinking; for each task, evaluate the rollback path."*
- **All three + Graphify:** After a `/graphify` build, you can ask context7 for current docs on a library that appeared in the graph, then use sequential-thinking to plan how to upgrade it. The graph tells you *what's there*, context7 tells you *what's current*, sequential-thinking tells you *how to bridge*.
