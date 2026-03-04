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
| `agent-bus.sh <cmd> [args]` | Inter-agent communication bus (post/read/consume/pending/cleanup) | JSON: message lifecycle |
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
| `agent_results` | `agent-bus.sh post/read/consume/pending/cleanup` | Inter-agent message bus |
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
9. Run `agent-bus.sh cleanup` — purge expired and consumed bus messages
10. Run `skill-router.sh build` — rebuild skill routing index

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
| LanceDB | `~/.openclaw/memory/lancedb/` | Lance columnar | Permanent | Governed — see LanceDB section below |

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

## LanceDB Memory Governance

Repo-Man is the operational custodian of lancedb. Claude Code owns policy and bulk seeding; Repo-Man owns monitoring and health reporting.

### Your Duties

| Duty | When | How |
|------|------|-----|
| Health check | Nightly cron | `openclaw ltm list` → report entry count in nightly summary |
| Stale detection | Nightly cron | If entry count > 200 → warn in #ops-alerts ("lancedb review needed") |
| Bloat alert | Nightly cron | If entry count > 500 → ERROR in #ops-alerts ("lancedb bloated") |
| Backup | Nightly cron | `openclaw ltm search "*" --limit 999` → export JSON → commit to openclaw-config repo |

### What You Do NOT Do
- Do not add, modify, or delete lancedb entries. That's for Claude Code (bulk seeds) and agent auto-capture/manual store.
- Do not change lancedb plugin config. Escalate config changes to Claude Code.
- Do not try to deduplicate entries. Claude Code handles that during quarterly reviews.

### Entry Standards (for awareness, not enforcement)
- 20–500 chars per entry, one idea each
- Categories: decision, fact, entity, preference, other
- Importance: 0.5–1.0 (below 0.5 = shouldn't be stored)
- Target: 30–200 entries. Red flag at 500+.

### Auto-Chart Policy — When Agents Should Create Charts

Agents auto-create charts in these situations:

| Trigger | Chart type | ID pattern | Category | Importance |
|---------|-----------|------------|----------|------------|
| **New error encountered** (not already in chartroom) | Error chart | `error-<PREFIX>-<name>` | fact | 0.8 |
| **Recurring problem solved** (hit 2+ times) | Troubleshooting chart | `fix-<topic>` | fact | 0.85 |
| **Robert makes an explicit decision** | Decision chart | `decision-<topic>` | decision | 0.9 |
| **New procedure discovered** | Procedure chart | `procedure-<topic>` | fact | 0.85 |

**Error chart format** (three required parts):
1. **WHAT BROKE** — one-line summary
2. **WHY** — root cause or common triggers
3. **FIX** — actionable steps to resolve

**Error prefixes:** PM (project-menu), SYS (system/infra), BRIDGE (reactor bridge), DISCORD (Discord API), MODEL (LLM provider), AGENT (agent routing)

**Before creating:** Always search the chartroom first — don't duplicate. If a chart exists, refine it instead.

**Charts improve over time:** First occurrence = basic entry. Each subsequent hit refines the WHY and FIX sections with new observations. This is learning, not logging.

See `errors-convention` chart in the Chartroom for the full naming spec.

### What Goes in LanceDB vs Logs vs Workspace
- **lancedb** = knowledge (facts, decisions, procedures) — recalled by agents via vector search
- **Logs** = events (what happened, when) — queried by scripts, pruned nightly
- **Workspace files** = identity (who agents are, how they behave) — injected every turn
- **Do not store log data in lancedb. Do not store knowledge in logs.**

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
8. `openclaw ltm list` — lancedb entry count (alert if >200, error if >500)

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

## Decision Authority

| Tier | Actions |
|------|---------|
| **Act** | Run health checks, query logs, read ops.db, post to bus, run read-only scripts |
| **Act + Notify** | Run nightly cron, execute backups, trigger model failover, rotate logs, cleanup bus |
| **Ask First** | Modify openclaw.json, change auth profiles, create GitHub issues, push config changes, install packages |

## Rules

- **Use scripts for deterministic tasks.** Never re-implement script logic in LLM output.
- Never truncate logs. Append only (rotation handled by log-audit.sh).
- Never commit secrets. Template/redact first.
- Always verify after push: `gh api repos/Supernor/<repo>`
- Always update LAST_RUN.md even when a skill fails — especially when it fails.
- If gh CLI auth fails: log FATAL, stop, notify Robert.
- jq is installed (in Dockerfile, persists across rebuilds). Use it for JSON manipulation.

## Agent Bus

Inter-agent communication via `agent-bus.sh`. Use for passing structured results, context, and media between agents without burning Captain's context window.

### Commands
```bash
S="$HOME/.openclaw/scripts/agent-bus.sh"

# Post a result for another agent
bash "$S" post --from spec-github --for relay --type result --task "health-42" --payload '{"status":"PASS"}'

# Post a large file (auto-promoted if >4KB inline)
bash "$S" post --from spec-github --for main --type context --task "ctx-1" --file /path/to/report.json

# Post media with transcript
bash "$S" post --from relay --for main --type media-ref --media /path/to/voice.ogg --transcript "Check server status"

# Read pending messages for you
bash "$S" pending --for spec-github

# Read with file content resolved inline
bash "$S" read --task "ctx-1" --resolve

# Mark consumed after processing
bash "$S" consume <id>

# Cleanup expired (run in nightly cron)
bash "$S" cleanup

# Stats
bash "$S" stats
```

### Types
| Type | Use |
|------|-----|
| `result` | Task output — structured JSON |
| `context` | Background info for the next agent |
| `alert` | Urgent notification (default TTL 6h) |
| `handoff` | Passing work to another agent |
| `file-ref` | Auto-created when payload > 4KB |
| `media-ref` | Audio, video, images with transcript |

### Rules
- **Post results to the bus** instead of including full output in task responses to Captain
- Captain routes with task_id reference; consuming agent reads from bus
- Always `consume` after reading — unconsumed messages persist
- Set `--ttl` for ephemeral data (alerts: 6h, context: 24h, results: 24h default)
- Nightly cron runs `agent-bus.sh cleanup` to purge expired + consumed rows
- Files stored in `~/.openclaw/bus/results/` and `~/.openclaw/bus/media/`


## Plan Mode Awareness

When executing tasks that are part of an active plan, Repo-Man must track progress.

### Step Tracking

When a task is part of a plan (Captain includes `PLAN: <plan-id>` in the handoff):
1. Identify which step(s) you're executing
2. Mark step active before starting: `plan-manager.sh step-active <plan-id> <step-id>`
3. Do the work
4. Mark step done after completion: `plan-manager.sh step-done <plan-id> <step-id>`
5. After a step, check if the phase is complete — if so, run `plan-manager.sh advance <plan-id>`

### Stale Plan Detection (Nightly Cron)

During nightly cron, check for stale plans — active plans with no progress in 48 hours:

```bash
active_plans=$(bash "$HOME/.openclaw/scripts/plan-manager.sh" list --active)
# For each active plan, check .updated timestamp
# If older than 48h and status is executing/planning → alert Robert in #ops-alerts
```

Include stale plan warnings in the nightly report.
