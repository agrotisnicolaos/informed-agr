# CLAUDE.md — informed-agr

Project-wide instructions for Claude Code agents. Read top to bottom on every new conversation.

---

## 1. First-instruction discipline (load-bearing)

**Every domain agent must read these 5 docs before acting, on every invocation:**

1. [docs/product-context.md](docs/product-context.md) — what the product is, who the user is, current stage
2. [docs/constraints.md](docs/constraints.md) — hard constraints, architectural rules
3. [docs/phasing.md](docs/phasing.md) — phase framing with reversibility tags
4. [docs/budget.md](docs/budget.md) — $0/mo ceiling, paid-recommendation approval flow
5. [docs/decisions.md](docs/decisions.md) — locked decisions log (D-001+), append-only

After reading the five, **grep for every file path the plan names**. If any are missing or renamed, push back to `tech-lead` before continuing. (Doc-drift guard — issue #1 in [`AGENT-IMPROVEMENTS.md`](AGENT-IMPROVEMENTS.md).)

---

## 2. Agent routing table

| Question class | Route to |
|---|---|
| Phase, scope, budget, $0-veto, "should this ship now or later?" | [`ceo`](.claude/agents/ceo.md) |
| Coordination, plugin/workflow choice, specialist disagreement, naming, decisions.md upkeep | [`tech-lead`](.claude/agents/tech-lead.md) |
| Caching, offline behavior, data freshness, Supabase Free-tier realities | [`system-designer`](.claude/agents/system-designer.md) |
| RLS, auth, secrets, Keychain, local-cache safety | [`security`](.claude/agents/security.md) |
| Postgres schema, migrations, multi-tenant shape, RPC design | [`data-architect`](.claude/agents/data-architect.md) |
| Swift / SwiftUI / iOS implementation (rename + rewrite if platform changes) | [`ios-builder`](.claude/agents/ios-builder.md) |

`tech-lead` is the default routing bottleneck. The user is asked only when no agent has authority.

---

## 3. Migration self-review checklist (pre-commit gate)

Any migration must pass this before commit:

- [ ] RLS enabled on the new table? (`ENABLE ROW LEVEL SECURITY`)
- [ ] Policies present, including `WITH CHECK`?
- [ ] `GRANT` to `authenticated` for the relevant verbs? *(RLS without the grant returns empty results silently — issue #2)*
- [ ] Row-count check before any data mutation?
- [ ] Rollback SQL block in the same commit?
- [ ] `RAISE EXCEPTION` placeholders match arguments?
- [ ] If dropping an index, is the matching constraint also dropped (or kept intentionally)?

---

## 4. Known regressions register

Fields/behaviors that have regressed before. Any agent touching one of these must verify the guard test exists; if not, the test ships with the fix.

| Field / behavior | Correct value / shape | Guard test |
|---|---|---|
| *(empty — fill as regressions accumulate)* | | |

**Hard rule:** when a fix is the *second* time for the same root cause, the next ship adds a guard test or it doesn't ship.

---

## 5. Bootstrap rule for imported artifacts

When importing artifacts from a previous repo or previous architectural era, mark them stale at the top of every file (`> STALE — superseded by D-NNN`) **before** any agent reads them. Code wins where docs disagree. *(Issue #5.)*

This repo is itself a starter kit (a GitHub template). The [`archive/learnings-for-next-project/`](archive/learnings-for-next-project/) directory is the **frozen origin snapshot** of the kit's distillation — read once for full context, but do not cite as authoritative for the current product. Cite `docs/` and `docs/decisions.md` instead.

---

## 6. Day-one workflow

0. **Run [`SETUP.md`](SETUP.md) install steps** if anything in the §7 inventory below is missing on this machine. Idempotent — already-installed pieces are skipped. See also [`USE-THIS-TEMPLATE.md`](USE-THIS-TEMPLATE.md) for the first-5-minutes path after spawning a new repo from this template.
1. `/gsd-new-project` — establishes PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md.
2. Fill in `docs/product-context.md`, `docs/constraints.md`, `docs/phasing.md`, `docs/budget.md`. Leave `docs/decisions.md` empty until the first locked decision.
3. `/graphify` — build the codebase knowledge graph (one-time per project; rerun after major refactors).
4. `/gsd-new-milestone` — establish the first milestone.
5. `/gsd-spec-phase 1` → `/gsd-discuss-phase 1` → `/gsd-plan-phase 1` → `/gsd-execute-phase 1` → `/gsd-verify-work 1` → `/gsd-ship`.

If the next platform isn't iOS, follow [`.claude/agents/RENAME-IF-NOT-iOS.md`](.claude/agents/RENAME-IF-NOT-iOS.md) and update the routing table in §2 above.

---

## 7. Plugin / MCP / skill inventory

Installed at user scope (follows you across all repos). Install via [`SETUP.md`](SETUP.md).

**Plugins:**
- `superpowers` — TDD, brainstorming, plan-writing, code review, debugging, parallel-agent dispatch
- `frontend-design` — production-grade frontend code
- `skill-creator` — author, edit, eval skills
- `context-mode` — keep large tool outputs out of the context window (MCP server + `/ctx-*` commands)
- `claude-mem` — cross-session memory (start worker with `npx claude-mem start`)

**MCP servers:**
- `context7` — up-to-date library/framework docs (**required**: GSD's planner/executor/researchers call `mcp__context7__*` in 9+ workflow files; without it, GSD silently degrades)
- `sequential-thinking` — structured-reasoning tool with explicit step records as tool output (audit trail; distinct from native extended thinking)
- `context-mode` MCP — batched shell + FTS5 search of recent tool outputs

**Slash commands / skills:**
- `/graphify` — third-party knowledge-graph builder; writes to `graphify-out/` (multimodal input, Obsidian-vault-compatible)
- GSD (`~/.claude/get-shit-done/`) — ~65 `gsd-*` skills for the project lifecycle, including `gsd-graphify` which writes a separate graph to `.planning/graphs/` for the GSD workflow

**GUI:** Obsidian — visualizes `graphify-out/` as a vault. Standalone app; no Claude integration.

**Three "codebase knowledge" systems coexist** (intentional, different stores, no conflict):

| System | Stores in | Strength |
|---|---|---|
| `gsd-graphify` | `.planning/graphs/graph.json` | Queryable graph for GSD workflow (BFS expand, diff over time) |
| `/graphify` (third-party) | `graphify-out/` | Multimodal input + Obsidian visual |
| `context-mode` | FTS5 index of recent tool outputs | Live keyword search across batched commands |

Run `/gsd-help` for the GSD command surface. Run `claude mcp list` to verify MCP servers.
