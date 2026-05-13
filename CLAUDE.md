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

After reading the five, **grep for every file path the plan names**. If any are missing or renamed, push back to `tech-lead` before continuing. (Doc-drift guard — issue #1 in `learnings-for-next-project/AGENT-IMPROVEMENTS.md`.)

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

The [`learnings-for-next-project/`](learnings-for-next-project/) directory is **reference material** — not project state. Do not cite it as authoritative for the current product; cite `docs/` and `decisions.md` instead.

---

## 6. Day-one workflow

1. `/gsd-new-project` — establishes PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md.
2. Fill in `docs/product-context.md`, `docs/constraints.md`, `docs/phasing.md`, `docs/budget.md`. Leave `docs/decisions.md` empty until the first locked decision.
3. `/gsd-new-milestone` — establish the first milestone.
4. `/gsd-spec-phase 1` → `/gsd-discuss-phase 1` → `/gsd-plan-phase 1` → `/gsd-execute-phase 1` → `/gsd-verify-work 1` → `/gsd-ship`.

If the next platform isn't iOS: rename `.claude/agents/ios-builder.md` to `.claude/agents/<platform>-builder.md` and rewrite the "What the existing repo already chose" section. Keep the rest of the structure.

---

## 7. Plugin / skill inventory

Installed at user scope (follows you across all repos):

- `superpowers` — TDD, brainstorming, plan-writing, code review, debugging, parallel-agent dispatch
- `frontend-design` — production-grade frontend code
- `skill-creator` — author, edit, eval skills
- `context-mode` — keep large tool outputs out of the context window (MCP server + `/ctx-*` commands)
- `claude-mem` — cross-session memory (start worker with `npx claude-mem start`)
- GSD (`~/.claude/get-shit-done/`) — ~65 `gsd-*` skills for the project lifecycle

Run `/gsd-help` for the GSD command surface.
