# Workflows — how to combine the toolchain across a project

This is the synthesis. Individual tools are documented in [`mcps.md`](mcps.md), [`plugins.md`](plugins.md), [`gsd.md`](gsd.md), [`other-skills.md`](other-skills.md). This file is about **sequences** — which tool to reach for in each part of a project's life, and how they chain.

## The mental model

Think of the toolchain as four layers:

```
┌─────────────────────────────────────────────────┐
│ Layer 4: Workflow orchestrators                 │
│   GSD (65 gsd-* skills) — full project loop     │
├─────────────────────────────────────────────────┤
│ Layer 3: Discipline + creative skills           │
│   superpowers (TDD, brainstorming, debugging)   │
│   frontend-design (UI quality)                  │
│   skill-creator (meta)                          │
├─────────────────────────────────────────────────┤
│ Layer 2: Knowledge / context tools              │
│   /graphify (visual graph), gsd-graphify        │
│   context-mode (FTS5 of recent outputs)         │
│   context7 (current library docs)               │
│   claude-mem (cross-session memory)             │
├─────────────────────────────────────────────────┤
│ Layer 1: Background infrastructure              │
│   sequential-thinking (audit-trail reasoning)   │
│   MCP servers (atlassian, drive, calendar, etc.)│
│   Hooks (PreToolUse, SessionStart)              │
└─────────────────────────────────────────────────┘
```

Higher layers compose lower layers. When you run `/gsd-execute-phase`, it internally invokes `superpowers:test-driven-development` (layer 3), which calls `mcp__context7__get-library-docs` (layer 2), which routes through `context-mode`'s MCP (layer 1). You only typed one command.

This means: **you mostly think at layer 4** (GSD commands). The rest fires automatically.

---

## Sequences by phase of a project

### Day 0 — fresh repo from this template

Goal: zero to "ready to plan Phase 1".

```
1. (One time) Run SETUP.md install — context7, sequential-thinking, /graphify, GSD bundle, all plugins
2. /gsd-config             ← set defaults: yolo, standard, research=yes, plan-check=yes, verifier=yes
3. /gsd-new-project        ← gathers deep context, writes PROJECT.md, REQUIREMENTS.md, ROADMAP.md, STATE.md
4. (Manual) Fill in docs/product-context.md, docs/constraints.md, docs/phasing.md, docs/budget.md
5. /graphify .             ← build the visual codebase graph (one-time)
6. /gsd-new-milestone      ← scope the first milestone (e.g. v0.1)
```

Tools invoked behind the scenes during step 3: `superpowers:brainstorming` (because new project = creative work), `mcp__context7__*` (if any frameworks were named), context-mode (every batched shell call).

---

### Greenfield phase — building something new

Goal: ship a new feature end-to-end with discipline gates.

```
1. /gsd-spec-phase 1                          ← clarify WHAT, score ambiguity
2. /gsd-discuss-phase 1                       ← gather context, ask questions
3. /gsd-plan-phase 1                          ← detailed task breakdown + dependency graph
   └─ auto-uses mcp__context7__* for current library docs
   └─ auto-uses superpowers:writing-plans
4. (Optional) /gsd-plan-review-convergence    ← extra rigor: cross-AI review until no HIGH concerns
5. /gsd-execute-phase 1                       ← waves of parallel tasks, atomic commits
   └─ auto-uses superpowers:test-driven-development
   └─ auto-uses superpowers:verification-before-completion
   └─ auto-uses subagent-driven-development for parallel waves
6. /gsd-verify-work 1                         ← conversational UAT
7. /gsd-code-review                           ← bugs / security / quality on changed files
   └─ for security-sensitive phases also: /gsd-secure-phase
8. /gsd-ship                                  ← create PR, run review, prep for merge
```

Add per phase type:
- **UI phase:** insert `/gsd-ui-phase` before step 1 (produces UI-SPEC.md), `/gsd-ui-review` after step 7
- **AI phase:** insert `/gsd-ai-integration-phase` before step 1, `/gsd-eval-review` after step 7
- **High-risk phase:** add `/graphify --update .` after step 5 to refresh the visual map; lets you spot architectural drift

---

### Brownfield onboarding — adopting GSD into an existing repo

Goal: get GSD running in a project that already has scattered planning docs.

```
1. /gsd-config                ← set defaults
2. /gsd-ingest-docs           ← surfaces conflicts between existing ADRs/PRDs/SPECs and GSD's expected shape
3. /gsd-new-project           ← only if the repo doesn't have PROJECT.md yet; merges with whatever exists
4. /gsd-map-codebase          ← parallel mapper agents produce .planning/codebase/ docs
5. /graphify .                ← visual graph for cross-cluster relationships
6. /gsd-graphify              ← second graph keyed to GSD phases (different stores, both useful)
7. /gsd-health                ← validate planning directory is healthy after import
```

Tools invoked: context-mode heavily (large output from mapper agents), sequential-thinking (if conflicts in step 2 are non-trivial).

---

### Bug investigation — something broke

Goal: root cause + fix + regression test, without ad-hoc guessing.

```
1. /gsd-debug                          ← spawns gsd-debugger; persistent state across context resets
   ├─ auto-uses superpowers:systematic-debugging (scientific method)
   ├─ auto-uses context-mode:diagnose (reproduce → minimize → hypothesize → instrument → fix → regression-test)
   └─ if multi-hypothesis: nudge sequential-thinking ("walk through the 3 candidate causes step by step")
2. /graphify query "what touches <module>?"   ← cheap BFS query against the graph
3. (Implement the fix using superpowers:test-driven-development)
4. /gsd-add-tests                      ← regression-guard test for this specific bug
5. /gsd-verify-work                    ← UAT to confirm fix
```

Critical: the fix isn't done until step 4 (regression guard test) lands. AGENT-IMPROVEMENTS.md issue #3 is exactly this trap.

---

### Refactor — improving what exists

Goal: tighten code without breaking it.

```
1. /gsd-spec-phase N                          ← phrase as "refactor X" not "improve" (more concrete)
2. /graphify .                                ← refresh visual graph BEFORE refactor
3. context-mode:improve-codebase-architecture ← surface refactor opportunities informed by docs/adr/
4. /gsd-plan-phase N                          ← incremental plan; reversible-cheap steps preferred
5. /gsd-execute-phase N                       ← atomic commits per refactor
6. /graphify --update .                       ← incremental graph re-extract (no LLM cost)
   └─ compare: did clusters consolidate or fragment?
7. simplify                                   ← cleanup pass on changed code
8. /gsd-verify-work N + /gsd-code-review
```

The before/after graph comparison (steps 2 and 6) is the unique value. If clusters got tighter, the refactor worked.

---

### UI work

Goal: ship UI that doesn't look AI-generic.

```
1. /gsd-ui-phase                             ← produces UI-SPEC.md (design contract)
2. /gsd-sketch                               ← throwaway HTML mockups to compare directions
3. /gsd-spec-phase N + /gsd-discuss-phase N  ← lock the chosen direction
4. /gsd-plan-phase N
5. /gsd-execute-phase N
   └─ frontend-design auto-fires on UI requests (distinctive over generic)
6. /gsd-verify-work N
7. /gsd-ui-review                            ← retroactive 6-pillar visual audit
```

Special: after step 7, ship to a real device for taste-feedback. AGENT-IMPROVEMENTS.md issue #7 (UI debt accumulating without device testing) is the trap to avoid.

---

### AI integration phase

Goal: add an LLM call / agent / classifier without it silently degrading.

```
1. /gsd-ai-integration-phase    ← produces AI-SPEC.md (eval plan, guardrails, monitoring)
2. /gsd-spec-phase N
3. /gsd-plan-phase N            ← auto-uses claude-api skill for SDK best practices
4. /gsd-execute-phase N
5. /gsd-verify-work N
6. /gsd-eval-review             ← audits eval coverage against AI-SPEC.md plan
```

Don't skip step 6. Without an eval, you're shipping an AI feature that *seems* to work without measurable evidence.

---

### Shipping cadence — when ready to merge

```
1. /gsd-pr-branch          ← clean PR branch without .planning/ noise
2. /gsd-ship               ← creates PR, runs review, preps for merge
   ├─ auto-runs /review for code review pass
   └─ for security-sensitive PRs: /security-review
3. (Manual: review CI, address feedback)
4. (Merge via gh CLI or GitHub UI)
5. /gsd-extract-learnings  ← capture decisions, lessons, patterns from this phase
   └─ updates docs/decisions.md (D-NNN entries) where relevant
6. /gsd-pause-work         ← clean handoff if pausing here, OR
   /gsd-progress           ← move to next phase
```

---

### End of milestone

```
1. /gsd-audit-uat                 ← cross-phase UAT debt check
2. /gsd-audit-milestone           ← compare actual to original milestone intent
3. /gsd-milestone-summary         ← stakeholder-readable summary
4. /gsd-complete-milestone        ← archive and prep next version
5. /gsd-cleanup                   ← clean up phase directories
6. /gsd-new-milestone             ← scope the next one
```

---

## Picking a "codebase knowledge" system

Three coexist; pick by question shape.

| Question shape | Use |
|---|---|
| "What relates to X? Show me the cluster around it." | `/graphify query "..."` (third-party graph in `graphify-out/`) |
| "What does the GSD planner know about phases connecting to this concept?" | `/gsd-graphify` (GSD's graph in `.planning/graphs/`) |
| "What did `npm test` print 5 minutes ago? What was in that gh PR view?" | `mcp__plugin_context-mode_context-mode__ctx_search` (recent tool outputs, FTS5) |
| "What are the current Next.js App Router APIs?" | `mcp__context7__get-library-docs` (live external docs) |
| "What did we decide about Supabase last week in the *other* project?" | `claude-mem` (cross-session memory; currently disabled, enable via env var) |

You can use multiple. Common combo: `/graphify query "..."` to find the cluster, then `mcp__context7__get-library-docs` to verify the API is current.

---

## Common composition patterns

### Pattern: "Plan a hard phase that depends on a fast-moving library"
```
/gsd-spec-phase N
↓
/gsd-discuss-phase N
↓
/gsd-plan-phase N  ← auto-fetches via mcp__context7__* during planning
↓
sequential-thinking  ← "find weaknesses in steps 3, 5, 7 of this plan"
↓
/gsd-plan-review-convergence
```

### Pattern: "Debug a regression that keeps coming back"
```
/gsd-debug  ← gsd-debugger spawned with persistent state
  ├─ scientific-method investigation (superpowers:systematic-debugging)
  └─ hypothesis branching with sequential-thinking
↓
Fix
↓
/gsd-add-tests  ← regression guard with would_pass_with_wrong_implementation=false
↓
Update CLAUDE.md "Known regressions register" ← per AGENT-IMPROVEMENTS.md issue #3
```

### Pattern: "Inherit a chaotic repo and figure out what's there"
```
/gsd-ingest-docs  ← surface existing planning conflicts
↓ (parallel)
/gsd-map-codebase     /graphify .
↓
/gsd-graphify  ← GSD-aware second pass
↓
Read `graphify-out/GRAPH_REPORT.md` god-nodes section ← spot the structural anchors
↓
Open `graphify-out/obsidian/` as a vault ← visual exploration in Obsidian
```

### Pattern: "Tighten up an over-engineered phase"
```
simplify ← code-quality pass on changed code
↓
context-mode:improve-codebase-architecture ← surfaces consolidation opportunities
↓
/gsd-spec-phase N+1  ← scope the cleanup as a phase, not ad-hoc
```

### Pattern: "Build UI from sketch to ship"
```
/gsd-explore  ← Socratic ideation
↓
/gsd-sketch  ← throwaway HTML mockups
↓
/gsd-ui-phase  ← lock into UI-SPEC.md
↓
/gsd-spec-phase N + /gsd-discuss-phase N + /gsd-plan-phase N
↓
/gsd-execute-phase N  ← frontend-design auto-fires for distinctive output
↓
/gsd-ui-review  ← 6-pillar audit
↓
Ship to device for taste-feedback
```

---

## Anti-patterns to avoid

- **Skipping `/gsd-discuss-phase`**: You'll write a plan against assumptions that don't match reality. AGENT-IMPROVEMENTS.md issue #1 (doc-drift) starts here.
- **Skipping `/gsd-verify-work`**: Tests passing ≠ UAT passing. Without UAT you ship features the user didn't ask for.
- **Running `/gsd-autonomous` early**: Powerful but risky. Use only after you've validated the roadmap manually for 2–3 phases.
- **Building UI without `/gsd-ui-phase`**: You'll iterate forever on aesthetics that should have been locked at spec time.
- **Adding an LLM call without `/gsd-ai-integration-phase`**: Ships eval-less AI features. Issue #7 of AGENT-IMPROVEMENTS for AI work.
- **Letting `claude-mem` stay disabled forever**: It's free cross-project memory. Enable it eventually.
- **Treating `/graphify` and `gsd-graphify` as the same thing**: They're not. Both are useful. Use both.
- **Using `simplify` as a substitute for `/gsd-code-review`**: `simplify` is a quick pass; code-review is rigorous. Don't conflate.
- **Burning context on raw tool outputs**: If you find yourself reading multi-page command outputs, you're bypassing context-mode. Let it route through.

---

## A note on what NOT to over-document

This doc deliberately doesn't list every gsd-* skill's argument syntax — `/gsd-help` does that and stays current. It also doesn't try to predict every situation; that's what judgment is for. The patterns above are starting points, not recipes to follow rigidly.

When in doubt: **`/gsd-progress`** — the unified situational command. It looks at where you are and suggests what's next.
