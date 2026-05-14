# RENAME-IF-NOT-iOS

`ios-builder.md` is the platform-specific specialist for Swift/SwiftUI projects. If your new project isn't iOS, do the rename + rewrite below before your first phase. The other 5 agents are platform-agnostic and stay as-is.

## When to rename

Rename if your client platform is anything other than native iOS:
- Android (Kotlin/Compose) → `android-builder.md`
- React Native / Expo → `mobile-builder.md`
- Web (React / Vue / Svelte / etc.) → `web-builder.md`
- Backend-only (no client UI) → `backend-builder.md`
- Desktop (Electron / Tauri / native) → `desktop-builder.md`

## What to rewrite

Open the renamed file and rewrite **only** the section titled "What the existing repo already chose" (or equivalent platform-specific block). Replace iOS-specific decisions (file-backed JSON cache, Swift Concurrency patterns, SwiftUI navigation) with the equivalents for your platform. Keep:

- The agent's `name:` frontmatter (update to match the new file)
- The `description:` frontmatter (update the platform name)
- The "What I do NOT do" section (the boundary with `system-designer`, `security`, `data-architect` is platform-agnostic)
- The regression-guard test discipline (`would_pass_with_wrong_implementation = false`) — applies on every platform
- The cache-vehicle ownership pattern (you own the *vehicle*; `system-designer` owns the *shape*) — applies on every platform

## What to update outside this file

After renaming:

1. Update `CLAUDE.md` §2 (Agent routing table) — change the `ios-builder` row to your renamed agent.
2. Update `README.md` if it mentions iOS.
3. Delete this `RENAME-IF-NOT-iOS.md` file (the rename is one-time).

## What about Supabase?

The other agents (`data-architect`, `security`, `system-designer`) reference Supabase Free tier as the default backend. If you're on a different backend (Firebase, Convex, PlanetScale, raw Postgres on a VPS, no backend), open each of those three agents and rewrite the platform-specific sections analogously. The decision discipline (RLS-equivalent, migration self-review checklist, $0/mo budget posture) stays.
