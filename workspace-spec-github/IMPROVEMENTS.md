# IMPROVEMENTS.md — spec-github (Repo-Man)

> Tracked defects and enhancement backlog. Reviewed at the start of each development session.
> Status: OPEN | IN-PROGRESS | DONE

## Bugs (fix these first)

### BUG-001 — Canonical key list uses wrong names
**Status:** DONE
**Discovered:** 2026-03-01 first boot
**Fixed:** 2026-03-01
**Symptom:** Drift check reported 3/7 keys found. Keys are present but names didn't match.
**Root cause:** AGENTS.md canonical list used short names (ANTHROPIC, GOOGLE, etc.) instead of full env var names.
**Fix applied:** Canonical list in AGENTS.md now uses exact env var names. Added Known-Excluded Keys section for OPENCLAW_PROD_DISCORD_APP_ID and OPENCLAW_PROD_SAG_KEY so drift check skips them.

### BUG-002 — Discord routing defaults to spec-github instead of pa-admin
**Status:** DONE
**Discovered:** 2026-03-01 first boot
**Fixed:** 2026-03-01
**Symptom:** User said "hi" in Discord, Repo-Man responded instead of pa-admin.
**Root cause:** openclaw.json agents.list only contained spec-github — OpenClaw defaults to the first agent in the list.
**Fix applied:** Added `{"id": "main", "default": true}` as first entry in agents.list. pa-admin now handles all Discord messages by default. spec-github only responds when explicitly invoked.

### BUG-003 — Bootstrap fires on any inbound Discord message
**Status:** DONE
**Discovered:** 2026-03-01 first boot
**Fixed:** 2026-03-01
**Symptom:** Boot report triggered by a "hi" message, not by session start.
**Root cause:** spec-github was the default agent (BUG-002), so every Discord message started a spec-github session, which triggered BOOTSTRAP.md. The bootstrap hook itself (bootstrap-extra-files) correctly fires on session start — the problem was routing, not hook scoping.
**Fix applied:** BUG-002 fix resolves this. Also added scope note to BOOTSTRAP.md clarifying it only runs when spec-github is explicitly invoked.

## Enhancements

### ENH-001 — Drift report should show which keys passed, not just which failed
**Status:** OPEN

### ENH-002 — Known-excluded keys trigger false WARN
**Status:** DONE (resolved by BUG-001 fix)

### ENH-003 — Boot report should direct user to pa-admin for general chat
**Status:** DONE (resolved by BUG-002 fix — pa-admin is now default)

### ENH-004 — session-wrap skill not yet wired
**Status:** OPEN

### ENH-005 — Nightly cron not yet configured
**Status:** OPEN
