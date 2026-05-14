# Use this template — first 5 minutes

You just clicked "Use this template" on `agrotisnicolaos/informed-agr` and got a fresh repo. Here's the path from clone to "ready to plan Phase 1."

## 1. Clone and personalize

```bash
git clone <your-new-repo-url>
cd <your-new-repo>
```

Edit `README.md`:
- Replace the project description (currently boilerplate from the template).
- Update or remove the "Origin" section.

If your client platform isn't iOS, follow [`.claude/agents/RENAME-IF-NOT-iOS.md`](.claude/agents/RENAME-IF-NOT-iOS.md). If your backend isn't Supabase, the same file has guidance for the other agents.

## 2. Run the install script

Hand `SETUP.md` to Claude in the new repo:

```
Read SETUP.md and run the install steps. Skip anything already installed at user scope.
```

Claude will check what's already on your machine and only install what's missing. The required surface:

- Plugins: `superpowers`, `frontend-design`, `skill-creator`, `context-mode`, `claude-mem`
- MCP servers: `context7`, `sequential-thinking`
- Slash command: `/graphify`
- GUI: Obsidian
- Skill bundle: GSD (~65 `gsd-*` skills)

## 3. Initialize the project

```
/gsd-new-project
```

Answer the deep-context questions honestly. Produces `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`.

## 4. Fill in the canonical docs

Edit each of these (currently TEMPLATE-marked scaffolds):

- `docs/product-context.md`
- `docs/constraints.md`
- `docs/phasing.md`
- `docs/budget.md`
- *(Leave `docs/decisions.md` empty — it's append-only and starts blank.)*

These five docs are read by every domain agent on every invocation. The first hour you spend on them returns interest for the whole project lifetime.

## 5. Build the codebase graph

```
/graphify
```

Produces `graphify-out/{graph.json, graph.html}` and (after the first run) an Obsidian-ready vault. Rerun after major refactors.

## 6. Plan and ship Phase 1

```
/gsd-new-milestone
/gsd-spec-phase 1
/gsd-discuss-phase 1
/gsd-plan-phase 1
/gsd-execute-phase 1
/gsd-verify-work 1
/gsd-ship
```

That's the durable loop. Everything else is variation.

---

## Reference material

- `archive/learnings-for-next-project/` — original distillation from the prior Demetra iOS build. Frozen snapshot. Don't cite as authoritative for the current project (cite `docs/decisions.md` instead) — but read once for the full origin context.
- `AGENT-IMPROVEMENTS.md` — 8 failure modes the agents are designed to catch. Read once before your first phase.
- `CLAUDE.md` — routing table, migration self-review checklist, regressions register, doc-drift guard.

## Delete this file when done

Once you've been through the first-5-minutes path, this file's purpose is served. Delete it (`rm USE-THIS-TEMPLATE.md`) so it doesn't clutter your project root.
