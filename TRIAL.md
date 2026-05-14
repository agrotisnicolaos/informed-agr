# TRIAL — smoke-test of SETUP.md

> **Branch:** `trial/smoke-test-setup`
> **Date started:** 2026-05-13
> **Result:** ✅ All required pieces installed and verified. SETUP.md updated to fold in the lessons.

## Summary

| Step | Item | Outcome |
|---|---|---|
| §0 | Prereqs | ✅ — but found Python 3.9 (Xcode CLT) is below Graphify's 3.10+ requirement |
| §1 | Marketplaces (`context-mode`, `thedotmack`) | ✅ Both already registered |
| §2 | 4 plugins (superpowers, frontend-design, skill-creator, context-mode) | ✅ All at user scope |
| §2a | claude-mem | ✅ Plugin registered (from prior session) |
| §2c | **Context7 MCP** | ✅ Added at user scope, HTTP transport, connected |
| §2d | **Graphify** | ✅ Installed via pipx (Python 3.14 from brew) — needed extra steps |
| §2d | **Obsidian** | ✅ Already installed at /Applications/Obsidian.app (1.12.7); brew skipped |
| §2e | **Sequential Thinking MCP** | ✅ Added at user scope, stdio via npx, connected |
| §3 | GSD bundle (v1.40.0, 65 skills) | ✅ Pre-installed |
| §5 | Final verify block | ✅ All 8 MCP servers connected; `graphify` skill present in `/` autocomplete |

## What was actually run

```bash
# Already installed (no-op): superpowers, frontend-design, skill-creator, context-mode, claude-mem, GSD

# Required: install pipx because Graphify needs Python 3.10+ but Xcode CLT only ships 3.9
brew install pipx                                                                # pulled python@3.14 as dep

# Three new requireds (parallel):
claude mcp add --transport http -s user context7 https://mcp.context7.com/mcp
claude mcp add -s user sequential-thinking -- npx -y @modelcontextprotocol/server-sequential-thinking
brew install --cask obsidian                                                     # already installed; skipped
pipx install graphifyy                                                           # PyPI name is graphifyy (typo intentional, name reclaim pending)
graphify install --platform claude                                               # registers ~/.claude/skills/graphify/SKILL.md
```

## Findings (folded back into SETUP.md)

### Finding 1 — Python 3.10+ is a hard prereq for Graphify, not documented

Xcode CLT ships Python 3.9. `pip install graphifyy` fails on 3.9 with the system Python. Workaround: install pipx via brew (which pulls a current Python automatically), then `pipx install graphifyy`. SETUP.md §2d updated to call this out.

### Finding 2 — Graphify install creates `~/.claude/CLAUDE.md` (user-scope global)

`graphify install --platform claude` writes a 3-line global instruction file at `~/.claude/CLAUDE.md` telling Claude to invoke the `graphify` skill on `/graphify`. This is benign (project-level CLAUDE.md still wins on conflicts), but the side-effect is undocumented in SETUP.md. Updated to note it.

### Finding 3 — Obsidian was already installed

`brew install --cask obsidian` errors out if `/Applications/Obsidian.app` already exists. Idempotent skip is fine — but SETUP.md §2d should say "skip if already installed."

### Finding 4 — Pre-existing `uv tool install graphify` on this machine

pipx warned: `File exists at ~/.local/bin/graphify and points to ~/.local/share/uv/tools/graphifyy/bin/graphify, not pipx's location. Not modifying.` This means the user previously installed graphify via `uv tool install` (or similar). The CLI works either way; pipx's binary symlink just isn't taking precedence. Not a blocker — `which graphify` resolves to the uv path. SETUP.md updated to mention `uv tool install graphifyy` as an alternative install path.

### Finding 5 — `claude mcp add` defaults to local scope

Without `-s user`, the MCP server is added to the project's local config and won't follow the user across repos. SETUP.md updated to use `-s user` in every `claude mcp add` example.

### Finding 6 — Context7 GitHub default branch is `master`, not `main`

The README is at `master/README.md`, not `main/README.md`. Doesn't affect install; but worth knowing if you're docs-spelunking.

## Final state — `claude mcp list`

```
claude.ai Atlassian Rovo: ✓ Connected
claude.ai Google Drive: ✓ Connected
claude.ai Google Calendar: ✓ Connected
claude.ai Gmail: ✓ Connected
plugin:context-mode:context-mode: ✓ Connected
plugin:claude-mem:mcp-search: ✓ Connected
context7: https://mcp.context7.com/mcp (HTTP) ✓ Connected
sequential-thinking: npx -y @modelcontextprotocol/server-sequential-thinking ✓ Connected
```

## Next step

Merge `trial/smoke-test-setup` back to `main` (the SETUP.md updates carry the lessons forward to anyone who clones from the template). This TRIAL.md can be deleted after merge or kept as a durable changelog of the first end-to-end test.
