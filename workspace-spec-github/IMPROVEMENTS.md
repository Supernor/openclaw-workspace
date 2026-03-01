# IMPROVEMENTS.md — spec-github (Repo-Man)

> Tracked defects and enhancement backlog.
> Status: OPEN | IN-PROGRESS | DONE

## Bugs

### BUG-001 — Canonical key list uses wrong names
**Status:** DONE (2026-03-01)

### BUG-002 — Discord routing defaults to spec-github instead of pa-admin
**Status:** DONE (2026-03-01)

### BUG-003 — Bootstrap fires on any inbound Discord message
**Status:** DONE (2026-03-01)

### BUG-004 — Key drift check only reads /app/.env, misses runtime env vars
**Status:** DONE (2026-03-01)
**Root cause:** Provider keys (OPENCLAW_PROD_*) are injected via docker-compose env vars, not in /app/.env.
**Fix:** key-drift-check.sh now checks both file keys and runtime env vars.

### BUG-005 — workspace-backup uses wrong directory name and misses 2 workspaces
**Status:** DONE (2026-03-01)
**Root cause:** Skill used `workspace-main/` but repo has `workspace/`. Only backed up 2 of 4 workspaces.
**Fix:** ws-backup.sh script backs up all 4 workspaces with correct directory names.

### BUG-006 — log-event never actually pushed WARN+ to ERRORS.md
**Status:** DONE (2026-03-01)
**Root cause:** Skill was LLM instructions only — LLM never executed the GitHub push steps.
**Fix:** log-event.sh script handles local + GitHub push deterministically.

## Enhancements

### ENH-001 — Drift report should show which keys passed
**Status:** DONE (2026-03-01) — script outputs full JSON with present/missing/extra

### ENH-004 — session-wrap skill not yet wired
**Status:** OPEN

### ENH-005 — Nightly cron not yet configured
**Status:** DONE (2026-03-01) — cron job `repo-man-nightly` added at 03:00 UTC

### ENH-006 — Skills rewritten as script wrappers
**Status:** DONE (2026-03-01)
All deterministic skills now call shell scripts. Scripts output JSON. LLM only involved for formatting and error judgment. Saves ~90% tokens on routine operations.

### ENH-007 — Gateway log query capability
**Status:** DONE (2026-03-01)
New script `gateway-log-query.sh` parses structured JSON gateway logs. Filters by level, module, time. Replaces grepping docker compose logs.

### ENH-008 — GitHub Issues for incidents
**Status:** DONE (2026-03-01)
New script `incident-manager.sh` creates/closes GitHub Issues automatically for model health incidents. Deduplicates. Robert gets email notifications for free.

### ENH-009 — Config versioning with git tags
**Status:** DONE (2026-03-01)
New script `config-tag.sh` creates timestamped tags in openclaw-config for rollback reference.

### ENH-010 — Skills backup script
**Status:** DONE (2026-03-01)
New script `skills-backup.sh` pushes all skills, hooks, and scripts to openclaw-skills repo.

### ENH-011 — GitHub Actions health check
**Status:** DONE (template, 2026-03-01)
Workflow at openclaw-config/.github/workflows/health-check.yml. Needs gateway URL configured. Runs every 6h, creates/closes issues on failure/recovery.

### ENH-012 — jq installed in container
**Status:** DONE (2026-03-01, runtime install — needs Dockerfile for persistence)

### ENH-013 — Missing repo clones
**Status:** DONE (2026-03-01)
openclaw-workspace and openclaw-skills now cloned in workspace-spec-github.

### ENH-014 — HEARTBEAT.md enabled for model health
**Status:** DONE (2026-03-01)

### ENH-015 — LanceDB for operational patterns
**Status:** OPEN
LanceDB is running with autoCapture. Could store error signatures and resolutions as embeddings for pattern matching on new errors.
