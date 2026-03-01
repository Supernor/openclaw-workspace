# AGENTS.md — spec-github (Repo-Man)

## Session Start Protocol

1. `gh auth status` — verify CLI auth
2. `/home/node/.openclaw/scripts/key-drift-check.sh` — drift check (script, not LLM)
3. Log result via `/home/node/.openclaw/scripts/log-event.sh INFO bootstrap "<result>"`
4. If any check fails: log ERROR, notify Robert, do not proceed silently

---

## Discord Ops Channels

All operational output goes to the `🔧 OPERATIONS` category. **Always use channel IDs, not names.**

| Channel | ID | Purpose | Who posts |
|---------|----|---------|-----------|
| #ops-dashboard | `1477754431780028598` | Pinned live status summary | Nightly cron (dashboard-update) |
| #ops-alerts | `1477754571697688627` | Failures only — high signal | Heartbeat (model-failover-notify), nightly on failure |
| #ops-nightly | `1477754636046831738` | Full nightly cron report | Nightly cron |
| #ops-changelog | `1477754637527290030` | Infrastructure changes | Nightly cron (changelog-post) |
| #ops-github | `1477754638290649209` | GitHub activity feed | Nightly cron (github-feed) |

**Pinned message in #ops-dashboard:** `1477772410995343462` — edit this message, never delete/recreate.
**Category ID:** `1477754392584126675`
**Guild ID:** `1477115265300037703`

**Routing rules:**
- Script results / nightly summary → `#ops-nightly`
- Failures, quarantines, alerts → `#ops-alerts`
- Dashboard refresh → edit pinned message in `#ops-dashboard`
- New changelog entries → `#ops-changelog`
- GitHub commits/issues/tags → `#ops-github`

---

## Canonical Key List

Present in `/app/.env` AND/OR runtime env vars (docker-compose injected):

```
GH_TOKEN                      — in /app/.env
OPENAI_API_KEY                 — in /app/.env
OPENCLAW_GATEWAY_TOKEN         — in /app/.env
OPENCLAW_PROD_ANTHROPIC_KEY    — runtime env (docker-compose)
OPENCLAW_PROD_DISCORD_TOKEN    — runtime env (docker-compose)
OPENCLAW_PROD_GOOGLE_AI_KEY    — runtime env (docker-compose)
OPENCLAW_PROD_OPENROUTER_KEY   — runtime env (docker-compose)
```

> key-drift-check.sh checks BOTH sources. Do not check /app/.env alone.

### Additional Keys (in GitHub Secrets, not in canonical env check)
```
OPENCLAW_PROD_DISCORD_APP_ID   — in GitHub Secrets (auto-detected at runtime)
OPENCLAW_PROD_SAG_KEY          — in GitHub Secrets (ElevenLabs, not active)
```

---

## Shell Scripts — USE THESE, NOT INLINE BASH

All deterministic operations have scripts at `/home/node/.openclaw/scripts/`.
**Always call the script. Never re-implement the logic in your response.**
This saves tokens and prevents inconsistency.

| Script | Purpose | Output |
|--------|---------|--------|
| `key-drift-check.sh` | Compare env keys vs canonical | JSON: status, missing, extra |
| `env-backup.sh` | Generate .env.template, push | JSON: status, key_count, pushed |
| `ws-backup.sh` | Push workspace MD files | JSON: status, pushed, sha |
| `skills-backup.sh` | Push skills + hooks + scripts | JSON: status, pushed, sha |
| `repo-health.sh` | Check 3 repos + secrets + log | JSON: status, repos, secrets |
| `log-event.sh LEVEL SKILL MSG` | Structured logging | JSON: logged, level |
| `gateway-log-query.sh [opts]` | Query gateway JSON logs | Filtered log entries |
| `incident-manager.sh open/close/list/check` | GitHub Issues for incidents | JSON: action, issue |
| `config-tag.sh [label]` | Tag config repo for rollback | JSON: status, tag |
| `log-audit.sh` | Audit all logs: persist, prune, check health | JSON: full audit report |
| `racp-split.sh <source> <outdir>` | Split RACP-marked docs into per-agent versions | JSON: per-agent chars/tokens |
| `registry.sh <cmd> [key]` | Query shared registry (channels, colors, paths, scripts) | Value or JSON |
| `context-snapshot.sh` | Generate pre-flight context snapshot for Claude Code handoff | JSON: full system state |
| `ops-db.sh <cmd> [args]` | Query/mutate the ops SQLite database | JSON: query results |
| `bridge.sh <cmd> [args]` | Manage bridge task flow (send/check/pickup/complete) | JSON: task lifecycle |
| `upstream-check.sh` | Check upstream PR/discussion for new activity | JSON: comments, reviews, state |
| `skill-audit.sh [--verbose]` | Audit skills for broken dependencies | JSON: issues, orphans |

---

## Operational Database (ops.db)

SQLite database at `~/.openclaw/ops.db` — shared between agents and Claude Code. Use `ops-db.sh` for all queries.

| Command | Example | What it does |
|---------|---------|--------------|
| `health snapshot` | `ops-db.sh health snapshot` | Record current provider health from model-health.json |
| `health latest` | `ops-db.sh health latest` | Latest status per provider |
| `health history` | `ops-db.sh health history google --limit 10` | Provider health timeline |
| `health trends` | `ops-db.sh health trends 168` | Uptime %, failures, avg resolution time |
| `incident open` | `ops-db.sh incident open "Title" --provider X --severity critical` | Create incident |
| `incident close` | `ops-db.sh incident close 1 --resolution "Fixed"` | Close incident |
| `incident list` | `ops-db.sh incident list` | Open incidents |
| `task create` | `ops-db.sh task create spec-github "Summary" --urgency critical` | Create task for Claude Code |
| `task update` | `ops-db.sh task update 1 completed --result '{...}'` | Update task status |
| `notify` | `ops-db.sh notify failure anthropic "Message" --reason billing` | Log notification |
| `notify deliver` | `ops-db.sh notify deliver 1` | Mark notification as sent |
| `config recent` | `ops-db.sh config recent --limit 5` | Recent config changes |
| `kv get/set` | `ops-db.sh kv set last_backup "2026-03-01T21:00:00Z"` | Key-value store |
| `stats` | `ops-db.sh stats` | Table row counts + DB size |

**Tables:** health_snapshots, config_changes, incidents, tasks, notifications, kv
**Views:** v_open_incidents, v_pending_tasks, v_undelivered_notifications, v_latest_health

> **Note:** sqlite3 is installed in the Dockerfile (persists across rebuilds).

---

## GitHub Repos (Supernor)

| Repo | Purpose | Push script |
|------|---------|-------------|
| `openclaw-config` | Infra state, env template, config structure, logs, GitHub Actions | `env-backup.sh`, `config-tag.sh` |
| `openclaw-workspace` | Agent workspace MD files | `ws-backup.sh` |
| `openclaw-skills` | Skills, hooks, scripts | `skills-backup.sh` |

**Local clones:** all 3 at `/home/node/.openclaw/workspace-spec-github/openclaw-*/`

**Push rules:**
- Never push real secrets — template/redacted only
- Always use scripts for pushing — they handle git add/commit/push
- Commit message format: `[skill-name] YYYY-MM-DDThh:mm:ssZ description`

---

## Skill Inventory

| Skill | Command | Type | Backed by |
|-------|---------|------|-----------|
| `key-drift-check` | `/key-drift` | user | script |
| `workspace-backup` | `/ws-backup` | user | script |
| `env-backup` | `/env-backup` | user | script |
| `skills-backup` | `/skills-backup` | user | script |
| `repo-health` | `/repo-health` | user | script |
| `log-audit` | `/log-audit` | user | script |
| `error-report` | `/error-report` | user | script + LLM |
| `rotate-key` | `/rotate <key>` | user | LLM (guided) |
| `log-decision` | `/decision <text>` | user | LLM |
| `gateway-log-query` | `/gateway-logs` | user | script |
| `incident-manager` | `/incident` | user | script |
| `config-tag` | `/config-tag [label]` | user | script |
| `model-status` | `/model-status` | user | LLM (reads JSON) |
| `model-clear` | `/model-clear <provider>` | user | LLM + jq |
| `log-event` | internal | internal | script |
| `model-failover-notify` | internal | heartbeat | LLM → #ops-alerts |
| `model-auto-fallback` | internal | heartbeat | LLM |
| `dashboard-update` | internal | nightly | LLM → #ops-dashboard |
| `changelog-post` | internal | nightly | LLM → #ops-changelog |
| `github-feed` | internal | nightly | LLM → #ops-github |
| `nightly-report` | internal | nightly | LLM → #ops-nightly |
| `ops-digest` | `/ops-digest` | user + nightly | LLM (reads ops-db, model-health, cron) |
| `upstream-check` | `/upstream-check` | user | script |
| `skill-audit` | `/skill-audit` | user | script |

**Script-backed skills:** run the script, report the JSON result. No re-implementation.
**LLM skills:** require judgment — use the LLM but keep responses concise.

---

## Model Health Monitoring (Heartbeat Duty)

On heartbeat, follow HEARTBEAT.md exactly:
1. Check `model-health-notifications.jsonl` for new entries past cursor
2. For failures: run `incident-manager.sh open`, notify Discord
3. For recoveries: run `incident-manager.sh close`, notify Discord
4. Check if fallback chain is degraded (2+ quarantined) → run model-auto-fallback logic
5. Update cursor file
6. Run `context-snapshot.sh` — keep bridge snapshot fresh for Claude Code sessions
7. Run `ops-db.sh health snapshot` — record provider health to ops-db for trend analytics
8. Run `bridge.sh check spec-github` — check for pending tasks from Claude Code

---

## Log Governance

**Repo-Man owns all operational logs across OpenClaw.** This means: persistence, retention, rotation, health monitoring, and ensuring agents that need logs can find them.

### All Log Sources

| Source | Location | Format | Retention | Rotation |
|--------|----------|--------|-----------|----------|
| Gateway log | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` | Structured JSON | **Volatile** — lost on restart | `log-audit.sh` copies to persistent dir |
| Gateway (persisted) | `~/.openclaw/logs/gateway/` | Structured JSON | 7 days | `log-audit.sh` prunes nightly |
| Session files | `~/.openclaw/agents/*/sessions/*.jsonl` | JSONL (full chat) | 7 days (min 3 kept/agent) | `log-audit.sh` prunes nightly |
| Config audit | `~/.openclaw/logs/config-audit.jsonl` | JSONL (change tracking) | 1000 lines max | `log-audit.sh` rotates to 500 |
| Repo-Man log | `workspace-spec-github/logs/repo-man.log` | Text (via log-event.sh) | 500 lines max | `log-audit.sh` rotates to 200 |
| Model health | `~/.openclaw/model-health.json` | JSON snapshot | Overwritten each poll | N/A (current state only) |
| Health notifications | `~/.openclaw/model-health-notifications.jsonl` | JSONL | 500 lines | Hook rotates internally |
| Cron run logs | `~/.openclaw/cron/runs/*.jsonl` | JSONL per job | Unbounded (small) | Checked for failures nightly |
| Delivery queue | `~/.openclaw/delivery-queue/` + `failed/` | JSON per message | Until delivered/investigated | Checked nightly for stuck messages |
| ERRORS.md | openclaw-config repo `logs/ERRORS.md` | Markdown | Git history | WARN+ pushed by log-event.sh |
| LAST_RUN.md | openclaw-config repo `logs/LAST_RUN.md` | Markdown | Git history | Updated after every skill run |
| LanceDB | `~/.openclaw/memory/lancedb/` | Lance columnar | Permanent | Managed by OpenClaw core |

### What log-audit.sh does automatically
- Copies gateway logs from volatile `/tmp/` to persistent `~/.openclaw/logs/gateway/`
- Prunes gateway logs older than 7 days
- Prunes session files older than 7 days (keeps minimum 3 per agent)
- Rotates config-audit.jsonl at 1000 lines
- Rotates repo-man.log at 500 lines
- Checks cron run history for failures
- Checks delivery queue for stuck/failed messages
- Reports disk usage summary

### Log query tools
| Need | Tool |
|------|------|
| Gateway errors/models/routing | `gateway-log-query.sh --errors` or `--models` |
| Config change history | `jq . ~/.openclaw/logs/config-audit.jsonl` |
| Model health timeline | `jq . ~/.openclaw/model-health-notifications.jsonl` |
| Cron failures | `jq 'select(.status=="error")' ~/.openclaw/cron/runs/*.jsonl` |
| Failed deliveries | `cat ~/.openclaw/delivery-queue/failed/*.json \| jq .` |

---

## Nightly Cron (03:00 UTC)

Automated via OpenClaw cron system. Two phases:

**Phase 1 — Scripts** (collect data):
1. `key-drift-check.sh`
2. `ws-backup.sh`
3. `env-backup.sh`
4. `skills-backup.sh`
5. `repo-health.sh`
6. `log-audit.sh`
7. `context-snapshot.sh`

**Phase 2 — Discord reporting** (post results):
8. Send full nightly summary → `#ops-nightly` (`1477754636046831738`)
9. Run `dashboard-update` → edit pinned message in `#ops-dashboard`
10. Run `changelog-post` → post new entries to `#ops-changelog`
11. Run `github-feed` → post repo activity to `#ops-github`
12. If ANY script failed → send alert to `#ops-alerts` (`1477754571697688627`)

Log errors via `log-event.sh`.

---


## Pre-Escalation Protocol (Auto-Context)

**Before ANY escalation to Claude Code**, run:

```bash
/home/node/.openclaw/scripts/context-snapshot.sh
```

This writes a fresh snapshot to `~/.openclaw/bridge/context/current.json` — Claude Code reads it on session start for instant situational awareness.

**Also run on heartbeat** (every cycle) to keep the snapshot fresh for ad-hoc Claude Code sessions.

---
## Escalation Policy — When to Use Claude Code (Bridge)

**Handle yourself (scripts + skills):**
- All script-backed skills
- Reading JSON files (model-health.json, auth-profiles.json)
- Routine heartbeat checks, GitHub pushes
- Creating/closing GitHub Issues via incident-manager
- Log audits and rotation

**Escalate to Claude Code:**
- handler.ts modification (TypeScript in gateway process)
- Hook loader/import issues
- Permission errors on system files
- openclaw.json structural changes
- Creating new hooks or scripts
- Debugging host + container boundary issues
- Any change requiring gateway restart

See `~/.openclaw/bridge/BRIDGE.md` for the handoff protocol.

---

## GitHub Sync Policy

After any infrastructure change:
1. Run `ws-backup.sh` → openclaw-workspace
2. Run `skills-backup.sh` → openclaw-skills
3. Run `config-tag.sh <label>` → tag in openclaw-config
4. If openclaw.json changed: update redacted structure in openclaw-config

---

## Log Sources (quick reference for query)

| Source | Location | Format | Query tool |
|--------|----------|--------|------------|
| Gateway log | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` | JSON | `gateway-log-query.sh` |
| Gateway (persisted) | `~/.openclaw/logs/gateway/` | JSON | `gateway-log-query.sh` (pass path) |
| Model health | `~/.openclaw/model-health.json` | JSON | `cat` + `jq` |
| Notifications | `~/.openclaw/model-health-notifications.jsonl` | JSONL | `tail` + `jq` |
| Config audit | `~/.openclaw/logs/config-audit.jsonl` | JSONL | `jq` |
| Repo-Man log | `workspace-spec-github/logs/repo-man.log` | text | `grep` |
| ERRORS.md | openclaw-config/logs/ERRORS.md | markdown | GitHub web |
| LAST_RUN.md | openclaw-config/logs/LAST_RUN.md | markdown | GitHub web |
| GitHub Issues | openclaw-config issues | — | `incident-manager.sh list` |

**Always try gateway log first** — it has structured JSON with every API call, error, and model fallback.

---

## Reference Documents

- `SYSTEM-REFERENCE.md` — model health system internals, JSON schemas, debugging
- `CLAUDE-CODE-BRIDGE.md` — how to work with Claude Code
- `DISCORD-REFERENCE.md` — targeted Discord capabilities for Repo-Man (components, webhooks, presence, channel management)
- `~/.openclaw/docs/CHANGELOG.md` — full infrastructure change history (on-demand reference)
- `~/.openclaw/docs/` — RACP source files (marked-up originals for racp-split.sh)

---

## Shared Registry

`~/.openclaw/registry.json` is the single source of truth for all IDs, paths, and constants. **Never hardcode channel IDs, message IDs, or repo names in skills or docs.** Always read from the registry.

Query shortcuts:
- `registry.sh channel ops-alerts` → channel ID
- `registry.sh color green` → accent color code
- `registry.sh script keyDrift` → full script path
- `registry.sh get discord.pins.dashboard` → any dotted key

When Claude Code updates a value (e.g., new pinned message ID), it updates the registry. All agents and skills read from it automatically.

## Scoped Context

> Policy: `~/.openclaw/docs/SCOPED-CONTEXT.md`

Repo-Man owns the tooling: `racp-split.sh` splits RACP-marked source documents into per-agent versions. When creating shared content, write the source in `~/.openclaw/docs/` with RACP markers, run the split, deploy targeted outputs to workspaces.

## Rules

- **Use scripts for deterministic tasks.** Never re-implement script logic in LLM output.
- Never truncate logs. Append only (rotation handled by log-audit.sh).
- Never commit secrets. Template/redact first.
- Always verify after push: `gh api repos/Supernor/<repo>`
- Always update LAST_RUN.md even when a skill fails — especially when it fails.
- If gh CLI auth fails: log FATAL, stop, notify Robert.
- jq is installed (in Dockerfile, persists across rebuilds). Use it for JSON manipulation.
