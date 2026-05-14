# Agent Improvements — Lessons from Demetra iOS

What went wrong in the previous project, what the agents could have caught, and what to bake into the next iteration of these agents. Sourced from `docs/decisions.md` (D-001..D-013), `docs/superpowers/sprints/`, CLAUDE.md "Known regressions", and the git log of `phase-1`.

---

## 1. Doc-drift was the #1 recurring failure mode

**Symptoms in the repo:**
- Commit `1f72835` — `[doc-drift] Correct W1.1 scope to match actual code state`. The plan said modify `MainTabView.swift`; the code had already been changed in a prior commit. Plan and code disagreed.
- Commit `36c3fac` — `[doc-drift] Reconcile Phase 1 framing across CLAUDE.md, phase-1-plan.md, product-context.md`. Three canonical docs disagreed on what Phase 1 even *was*.
- Commit `4e75345` — `[doc-drift] Lock W3.1 design — all 7 §11 items accepted as drafted`. Decisions had to be retroactively locked because the design doc was treated as authoritative before any explicit lock.
- Commit `02d93ab` — `[doc-drift] Add schema transformation plan from data-architect`. The plan was generated *after* implementation considerations had drifted.

**Root cause:** Planner agents (and `gsd-planner` specifically) drafted plans against the doc set, not against the actual code. Specialists then ran with the plan without re-grounding.

**Fix in next iteration:**

- **`tech-lead`** now has a mandatory "Doc-drift guard" section (added in the generalized version) — before accepting any plan that names files/functions, grep the codebase for every identifier. Surface drift before approving.
- **All specialists** should add a sentence to their "First instruction" block: *"After reading the five docs, grep for every file path the plan names. If any are missing or renamed, push back to `tech-lead` before continuing."* (Not added to every agent file by default — it's prescriptive enough that it belongs in CLAUDE.md as a project-wide rule.)
- **`agrotis-ceo`/`ceo`** explicitly veto plans that reference docs older than the most recent merged commit on the affected file.

---

## 2. Migration drafts shipped with avoidable SQL bugs

**Symptoms in the repo:**
- Commit `272d2b9` — `[W1.3b-fix] Correct RAISE EXCEPTION placeholder mismatch in 006`. Placeholder count vs argument count was wrong.
- Commit `ff209a3` — `[W1.3b-fix] Drop the UNIQUE constraint, not just the index, in 006`. The migration tried to drop the index without dropping the matching UNIQUE constraint — they are separate database objects.
- Commit `8ab1205` — `[W2.1-fix] Grant SELECT to authenticated on family tables`. The initial migration added the table + RLS policies but forgot the `GRANT SELECT` — RLS without the grant returns empty results silently. Took a second commit to discover.

**Root cause:** `data-architect` and `security` produced migrations that compiled but had a class of "easy mistake" that wasn't structurally guarded.

**Fix in next iteration:**

- **`data-architect`** now has explicit migration-discipline bullets covering: RAISE EXCEPTION placeholder verification, drop-constraint-when-dropping-index, and the constraint-vs-index distinction. (Added to the generalized `data-architect.md`.)
- **`security`** now has an explicit "Grants matter" line: when a new RLS-protected table is added, the same migration must `GRANT SELECT [, INSERT, UPDATE, DELETE] ON <table> TO authenticated` — RLS without the grant is silently broken. (Added to the generalized `security.md`.)
- **Migration self-review checklist** — add to project CLAUDE.md as a pre-commit gate for any migration:
  - [ ] RLS enabled on the new table? (`ENABLE ROW LEVEL SECURITY`)
  - [ ] Policies present, including `WITH CHECK`?
  - [ ] `GRANT` to `authenticated` for the relevant verbs?
  - [ ] Row-count check before any data mutation?
  - [ ] Rollback SQL block in the same commit?
  - [ ] `RAISE EXCEPTION` placeholders match arguments?
  - [ ] If dropping an index, is the matching constraint also dropped (or kept intentionally)?

---

## 3. The same regression keeps coming back when there's no test guarding it

**Symptoms in the repo:**
- CLAUDE.md "Known regressions" — `HRV defaultAggregation must be p50 (right-skewed distribution). Has regressed to avg twice.`
- Sprint C spec: `This is the third time this fix has been in scope`.
- Also flagged: SpO2 / body fat fraction-vs-percent (×100), sleep-stage aggregation (single segments vs nightly sessions).

**Root cause:** When a regression-prone field exists, the codebase needs a test that fails on the *wrong* implementation. None of these had one.

**Fix in next iteration:**

- **`ios-builder` / client-platform specialist** now has explicit guidance on regression-guard tests with a `would_pass_with_wrong_implementation = false` assertion. (Added to the generalized `ios-builder.md` and to `data-architect.md`.)
- **`tech-lead`** maintains a "Known regressions" register in CLAUDE.md (or `docs/regressions.md`) and asks any agent touching a flagged field whether the guard test exists. If not, the test ships with the fix.
- **Convention:** When a fix is the *second* time for the same root cause, the next ship adds a guard test or it doesn't ship. Hard rule.

---

## 4. Speculative columns / "future-proofing" leaked through despite a CEO veto

**Symptoms in the repo:**
- D-013 had to explicitly carve scope: *"`get_vital_comparison` intentionally NOT cached in W3.1 — scope-creep guard."* Without the explicit guard, the cache spec would have grown.
- D-005 required defending "no agent infrastructure in Phase 1" against repeated future-proofing attempts ("even framed as 'future-proofing for Phase 2'").
- D-010 had to explicitly veto `family_id` on `vital_observation` despite multiple drafts proposing it for "schema readiness."

**Root cause:** `agrotis-ceo`'s veto was effective only when an explicit question reached them. Specialists were quietly adding speculative scope inside their proposals that didn't get surfaced as a phasing question.

**Fix in next iteration:**

- **`tech-lead`** explicitly checks every specialist proposal for: (a) new columns/tables that aren't required by the current phase spec, (b) new infrastructure not required for current-phase user flows. If found, surface as a phasing question to `ceo` *before* approving the proposal.
- **`ceo`'s** Phase Vetoes section now includes the line **"even framed as 'future-proofing'"** explicitly — was originally implicit, now load-bearing. (Carried into the generalized `ceo.md`.)

---

## 5. Stale `.planning/` directory poisoned later decisions

**Symptoms in the repo:**
- D-006 had to be issued: *"`.planning/` directory is partially stale (residual 3-tier design). Ground truth = code + docs/. Agents must cross-check before relying on it."*
- This was issued *after* multiple early decisions had been made citing `.planning/` as authoritative.

**Root cause:** Inherited planning artifacts from a previous architecture (abandoned 3-tier Next.js plan) were treated as current truth by agents that read them.

**Fix in next iteration:**

- **Bootstrap rule for new projects** (add to CLAUDE.md): when importing artifacts from a previous repo or previous architectural era, mark them stale at the top of every file (`> STALE — superseded by D-NNN`) before any agent reads them.
- **All specialists** have a "cross-check against code" instruction in their first-instruction block. This is now stronger than before — code wins where docs disagree.
- **`gsd-ingest-docs`** workflow is helpful here — it surfaces conflicts before they propagate.

---

## 6. Phasing framing drifted across canonical docs

**Symptoms in the repo:**
- Commit `36c3fac` had to reconcile what "Phase 1" meant across CLAUDE.md, phase-1-plan.md, product-context.md.
- D-002 explicitly says "the 3-tier plan in `.planning/` is abandoned" — this had to be locked because earlier docs implied otherwise.

**Root cause:** When phasing changed (D-001..D-003 reshaped the architecture), only some docs were updated. The rest kept their pre-pivot framing.

**Fix in next iteration:**

- **`ceo` / `tech-lead`** treat a phasing change as a multi-file edit: any decision that reshapes a phase reaches into `docs/product-context.md`, `docs/phasing.md`, `docs/phase-N-plan.md`, and CLAUDE.md in the same commit. Single-file phasing changes are vetoed.
- **`gsd-docs-update`** can be invoked after any phasing-shaping decision to surface stale references.

---

## 7. UI / visual quality was deferred so far it became a backlog

**Symptoms in the user's auto-memory:**
- `project_ui_debt.md` — "UI needs significant improvement; flagged after first device run in Phase 4; fix in Phase 5 (Dashboard) and Phase 6 (Chat UI); run /gsd-ui-review after each."
- `feedback_healthkit_supabase.md` — "force light mode, global sync banner, HealthKit auth via Settings only" — visual feedback that surfaced only after device testing.

**Root cause:** Agents respected "visual / UX → user" routing strictly, but no agent surfaced a "this should ship to device for taste-feedback now" check. By the time the user saw the UI on-device, the debt had accumulated.

**Fix in next iteration:**

- **`ios-builder` / client-platform specialist** now has an explicit "ship to device for taste-feedback" trigger: when more than one phase has shipped without device testing, surface it as a request. Not a decision they make — just a flag they raise. (Not added to the generalized file by default — too project-specific. Add per-project in CLAUDE.md if relevant.)
- **`/gsd-ui-review`** invocation is the recommended discipline after any phase that ships UI changes.

---

## 8. The agent role descriptions improved, but `tech-lead` was the bottleneck and didn't always know it

**Symptoms in the repo:**
- `tech-lead` was added explicitly because the user was being asked too many routing questions (CLAUDE.md routing rules + "Plugin interaction rule"). This worked.
- But `tech-lead` didn't always *check* whether specialists had read all five docs before passing along their output. Caused doc-drift (issue #1).

**Fix in next iteration:**

- **`tech-lead`** explicitly: "If a specialist returns advice that contradicts a locked decision, push back and ask for a re-read; do not pass that advice along." (Already in the original — keep, do not soften.)
- **`tech-lead`** adds the doc-drift guard (new in the generalized version): grep the code before approving a plan.
- **Routing observability** — `tech-lead` writes a one-line note in `decisions.md` ("D-NN routed to system-designer + ios-builder, synthesized by tech-lead, ceo notified for budget") so the user can audit where decisions came from after the fact.

---

## Cross-cutting: what was *right* about the agent setup

For the next project, **keep**:

- **Five-doc first-instruction discipline.** Every agent reads `product-context.md`, `constraints.md`, `phasing.md`, `budget.md`, `decisions.md` before acting. This is the highest-leverage rule in the whole system.
- **Decisions.md as append-only ledger with D-IDs.** Cite-by-ID is fast and durable.
- **Reversibility tags on every recommendation.** "Reversible-cheap" vs "expensive-to-undo" is the right primitive for solo-founder velocity.
- **`tech-lead` as the routing bottleneck.** Removes the user from technical routing 95% of the time.
- **`ceo` separated from `tech-lead`.** Phasing/budget is a different decision class than implementation routing — separating them keeps each one's veto credible.
- **Specialists have explicit "what I do NOT do" sections.** Forced clean handoffs. Did not see specialists going out of bounds in the git log — that's the proof this worked.

---

## How to adopt these in the new repo

1. The generalized agent files in `agents/` already incorporate the fixes from issues #1–4. Drop them into `.claude/agents/`.
2. Copy the **Migration self-review checklist** (issue #2) into the new project's CLAUDE.md.
3. Copy the **Known regressions register** scaffold (issue #3) into the new project's CLAUDE.md — start empty, fill as regressions accumulate.
4. Add a **Bootstrap rule** (issue #5) to the new project's CLAUDE.md: any imported artifact gets a stale-banner until reviewed.
5. Set a calendar reminder / convention: after every two completed phases, run `/gsd-ui-review` (issue #7) and ship a build to TestFlight / device.
