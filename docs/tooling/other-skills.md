# Other skills

Skills that aren't part of GSD, the main plugins, or MCPs — installed at user scope and worth knowing about. Most are utilities; a couple are powerful enough to deserve their own section.

All are activated by typing the slash command (or, for some, by phrasing matching the auto-trigger description).

---

## Configuration / setup utilities

### `update-config`
**Activation:** Manual, but Claude can also dispatch when the user asks for config changes.

**What it does.** Manages `~/.claude/settings.json` and project-scope `.claude/settings.json` — adds permissions, hooks, env vars, model preferences. Anything that ends in *"from now on, when X happens, do Y"* needs a hook configured here, not memory or preferences.

**When to use:**
- *"Allow npm commands without prompting."*
- *"Set DEBUG=true in the env."*
- *"When Claude finishes, show me a notification."* — this needs a Stop hook in settings.
- *"Move this permission from project to user scope."*

**Don't use for:** Theme/model toggles — `/config` is faster for those.

---

### `keybindings-help`
**Activation:** Manual. Triggers on phrasing like "rebind", "chord shortcut", "keybinding".

**What it does.** Edits `~/.claude/keybindings.json` — keyboard shortcut customization for the Claude Code TUI/IDE.

**When to use:** *"Rebind ctrl+s to send."*, *"Add a chord shortcut for /gsd-progress."*, *"Change the submit key."*

---

### `fewer-permission-prompts`
**Activation:** Manual.

**What it does.** Scans your transcripts for common read-only Bash and MCP tool calls, then proposes an allowlist for `.claude/settings.json` to reduce future permission prompts.

**When to use:** Periodically — when you notice you're getting prompted for the same Bash commands over and over.

**Cost/benefit:** Cuts friction significantly after a few sessions. Run it once or twice in a new project, then forget.

---

### `init`
**Activation:** Slash command `/init`.

**What it does.** Initializes a new CLAUDE.md by analyzing the codebase. Useful for greenfield projects without a CLAUDE.md.

**When to use:** Brand-new project, no CLAUDE.md exists. **Skip in this template** — we already ship a hand-tuned CLAUDE.md.

**Don't use for:** Refreshing an existing CLAUDE.md — `/gsd-docs-update` is more careful about preserving project-specific decisions.

---

## Code review utilities

### `review`
**Activation:** Slash command `/review`.

**What it does.** Reviews a pull request — fetches the PR, walks the diff, comments on issues. Lighter than `/gsd-code-review`.

**When to use:** Quick second-pass on someone else's PR, or your own pre-merge.

### `security-review`
**Activation:** Slash command `/security-review`.

**What it does.** Security-focused review of pending changes on the current branch. Looks for common OWASP categories, secret leakage, injection risks.

**When to use:** Before merging anything that touches auth, user input, file uploads, external API calls, secret handling. Distinct from `/gsd-secure-phase` which verifies a *threat model* you wrote in PLAN.md.

---

## Workflow utilities

### `loop`
**Activation:** Slash command. `/loop 5m /<command>` runs the command on a 5-minute interval. Omit the interval and Claude self-paces.

**What it does.** Recurring task runner. Runs a prompt or slash command on an interval until you stop it.

**When to use:**
- *"`/loop 5m /gsd-progress`"* — keep checking phase progress.
- *"`/loop /babysit-prs`"* — recurring PR check.
- *"`/loop` poll the deploy status until done"* — interval polling.

**Don't use for:** One-off tasks. The whole point is recurrence.

---

### `schedule`
**Activation:** Slash command. Manages remote routines that run on a cron schedule (cloud-side, not local).

**What it does.** Creates, updates, lists, runs scheduled remote agents that execute on a cron schedule. Also handles one-off scheduled runs ("run this once at 3pm").

**When to use:**
- *"Schedule `/gsd-stats` to run every Monday at 9am."*
- *"Remind me to check the deploy tomorrow at noon."*
- *"Run this nightly: pull recent PRs, summarize, post to Slack."*

**Different from `loop`:** `loop` is a local recurring task in this session. `schedule` runs cloud-side, persistent across sessions.

---

## Code-quality utilities

### `simplify`
**Activation:** Manual.

**What it does.** Reviews changed code for reuse, quality, efficiency, then fixes any issues found.

**When to use:** After a phase that you suspect is over-engineered or has duplication. Cheaper than full `/gsd-code-review`; less thorough.

---

## API / framework utilities

### `claude-api`
**Activation:** Auto-triggered when files import `anthropic` / `@anthropic-ai/sdk`, or when you ask for Claude API / Anthropic SDK / Managed Agents work.

**What it does.** Builds, debugs, optimizes Claude API / Anthropic SDK apps. Includes prompt caching by default. Handles migrating between Claude model versions (4.5 → 4.6 → 4.7, retired-model replacements).

**When to use:** Any code that imports the Anthropic SDK. Especially:
- Adding/tuning a Claude feature (caching, thinking, compaction, tool use, batch, files, citations, memory)
- Migrating from old to new Claude model versions
- Optimizing cache hit rate

**Don't use for:** Files importing OpenAI or other providers — wrong skill.

---

## Skill utilities (meta)

### `skill-creator:skill-creator`
Already covered in [`plugins.md`](plugins.md#3-skill-creator). The plugin's main skill.

### `superpowers:writing-skills`
Already covered in [`plugins.md`](plugins.md#1-superpowers). Lighter discipline checklist for editing existing skills.

---

## When to use which utility

| Situation | Reach for |
|---|---|
| Permission prompts annoying you | `fewer-permission-prompts` |
| Need to add a hook / env var / settings change | `update-config` |
| Recurring task in this session | `loop` |
| Recurring task across sessions / cloud-side | `schedule` |
| Quick PR review | `/review` |
| Security pass on pending changes | `/security-review` |
| Tighten up over-engineered code | `simplify` |
| Anything Anthropic-SDK | `claude-api` |
| Forgot a Claude Code keyboard shortcut | `keybindings-help` |
| Brand-new project needs CLAUDE.md | `/init` (skip in this template) |

Most of these are situational — you won't reach for them weekly. Worth skimming once so you remember they exist.
