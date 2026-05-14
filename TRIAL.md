# TRIAL — smoke-test of SETUP.md

> **Branch:** `trial/smoke-test-setup`
> **Date started:** 2026-05-13
> **Goal:** Walk through `SETUP.md` end-to-end on this machine to verify the template's install path actually works. Document anything that needs fixing.

## What's being tested

Running `SETUP.md` on a machine that already has most pieces at user scope. The real test is the three new required tools added in this template:

| Section | Item | Expected pre-test state | What to verify |
|---|---|---|---|
| §0 | Prereqs | All commands runnable | Listed plugins/skills match |
| §1 | Marketplaces (`context-mode`, `thedotmack`) | Both registered | `ls ~/.claude/plugins/marketplaces/` shows both |
| §2 | 4 plugins (superpowers, frontend-design, skill-creator, context-mode) | All installed at user scope | `installed_plugins.json` lists all 4 |
| §2a | claude-mem | Installed last session | Plugin entry present |
| §2c | **Context7** | NOT installed | `claude mcp list` shows context7 after install |
| §2d | **Graphify** | NOT installed | `/graphify` slash command available; `graphify install` succeeds |
| §2d | **Obsidian** | NOT installed | `brew list --cask obsidian` returns success |
| §2e | **Sequential Thinking** | NOT installed | `claude mcp list` shows sequential-thinking after install |
| §3 | GSD bundle | Installed v1.40.0 | VERSION file present, ~65 skills |
| §5 | Final verify block | All ✓ | All checks pass |

## Findings (filled in as we go)

*(Will be appended below as the install runs.)*

## Out of scope for this trial

- §4c `/gsd-new-project` — would write project-specific artifacts; we're testing install, not starting a real project.
- §4d `/gsd-map-codebase` — same reason; the template has no production code to map.
