# informed-agr

> Starter kit for Claude-Code-driven projects. Click **"Use this template"** on GitHub to start a new repo with this layout.

## What this is

A precursor for any new project. The repo ships:

- A 6-agent domain team (`ceo`, `tech-lead`, `system-designer`, `security`, `data-architect`, `ios-builder`) wired into `.claude/agents/`.
- 5 canonical doc scaffolds in `docs/` that every agent reads on every invocation (product-context, constraints, phasing, budget, decisions).
- `CLAUDE.md` with routing table, doc-drift guard, migration self-review checklist, and known-regressions register.
- `SETUP.md` — one-shot install instructions for plugins, MCP servers, GSD, and tooling (Context7, Graphify+Obsidian, Sequential Thinking).
- An archived snapshot of the original distillation in `archive/learnings-for-next-project/`.

## What you get out of the box

Once `SETUP.md` runs end-to-end, your new project has:

- **Plugins:** `superpowers`, `frontend-design`, `skill-creator`, `context-mode`, `claude-mem`
- **MCP servers:** `context7` (library docs), `sequential-thinking` (audit-trail reasoning), `context-mode` (FTS5-indexed batched output)
- **Slash commands:** `/graphify` (codebase knowledge graph)
- **Skill bundle:** GSD (~65 `gsd-*` skills covering project lifecycle, phase workflow, quality gates, diagnostics)
- **GUI:** Obsidian for visualizing the Graphify output

## Day-one workflow

1. Click "Use this template" on GitHub → create your new repo.
2. Follow [`USE-THIS-TEMPLATE.md`](USE-THIS-TEMPLATE.md) — first-5-minutes path from clone to ready-to-plan-Phase-1.
3. Hand `SETUP.md` to Claude in the new repo.
4. Run `/gsd-new-project` and fill in the 5 canonical docs.
5. Plan + ship Phase 1 via the GSD loop.

## Required reading before first phase

- [`AGENT-IMPROVEMENTS.md`](AGENT-IMPROVEMENTS.md) — 8 failure modes the agents are designed to catch (doc-drift, migration SQL bugs, regression-prone fields, scope-creep through future-proofing). Distilled from the prior Demetra iOS build.

## Origin

This kit is distilled from the **Demetra iOS** build (solo founder, $0/mo Supabase Free tier, offline-first, multi-tenant). The agent role boundaries, decision-ledger discipline, and Phase 1 vetoes were validated against that build's git history. The original distillation lives in `archive/learnings-for-next-project/` as a frozen snapshot — read once for full context.

## Layout

```
.claude/agents/        # 6 domain agents + RENAME-IF-NOT-iOS guidance
docs/                  # 5 canonical doc scaffolds
SETUP.md               # plugin + MCP + skill install script
USE-THIS-TEMPLATE.md   # first-5-minutes after spawning a new repo
AGENT-IMPROVEMENTS.md  # 8 failure modes + fixes
CLAUDE.md              # routing table + project-wide rules
archive/               # frozen origin snapshot — reference only
```
