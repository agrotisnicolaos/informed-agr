# learnings-for-next-project

Drop-in starter kit for the next Claude-Code-driven project. Distilled from the Demetra iOS build.

## What's in here

| File | Purpose |
|---|---|
| [SETUP.md](SETUP.md) | Day-one instructions for Claude: install plugins, marketplaces, GSD, and wire up the 6 domain agents. |
| [AGENT-IMPROVEMENTS.md](AGENT-IMPROVEMENTS.md) | 8 observed failure modes from the Demetra build with concrete fixes already applied to the agents in this folder. Read once before starting. |
| [agents/ceo.md](agents/ceo.md) | Strategic owner — phasing, scope, $0-budget vetoes. (Was `agrotis-ceo` in the previous repo.) |
| [agents/tech-lead.md](agents/tech-lead.md) | Coordinator — routes specialist questions, maintains decisions.md, doc-drift guard. |
| [agents/system-designer.md](agents/system-designer.md) | Architecture — caching, offline state machines, Supabase Free-tier realities. |
| [agents/security.md](agents/security.md) | RLS, auth, secrets, local-cache safety. Defers regulatory/legal to qualified humans. |
| [agents/data-architect.md](agents/data-architect.md) | Postgres schema, migrations, multi-tenant shape, RPC design. |
| [agents/ios-builder.md](agents/ios-builder.md) | Client-platform implementation. Rename to `<platform>-builder.md` if the next project isn't iOS. |

## How to use

1. Hand `SETUP.md` to Claude in the new repo — it walks Claude through plugin install + day-one wiring.
2. Copy `agents/*.md` into the new repo's `.claude/agents/`. Update the `name:` frontmatter in `ceo.md` if you want your personal handle back.
3. Create the 5 canonical docs (`product-context.md`, `constraints.md`, `phasing.md`, `budget.md`, `decisions.md`) under `docs/`. The agents read them on every invocation.
4. Read `AGENT-IMPROVEMENTS.md` before your first phase so you know what to watch for.

## What's been generalized vs preserved

- **Preserved:** solo-founder context, Supabase Free-tier constraints, $0/mo budget ceiling, 5-doc first-instruction discipline, reversibility tags, `decisions.md` D-ID ledger.
- **Generalized away:** all health-domain references (HealthKit, vitals, HRV, etc.), all Demetra-specific file paths, D-001..D-013 specifics, sprint A..J references, all ADR-numbered cross-references.
- **Adapt-per-project:** `ios-builder.md` defaults to Swift/SwiftUI. Rename + rewrite the "What the existing repo already chose" section if the new project is Kotlin / React Native / web. The rest of the structure transfers.
