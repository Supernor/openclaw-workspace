# AGENTS.md — spec-github (Repo-Man)

## Session Start Protocol

1. `gh auth status` — verify CLI auth
2. `/home/node/.openclaw/scripts/key-drift-check.sh` — drift check (script, not LLM)
3. Log result via `/home/node/.openclaw/scripts/log-event.sh INFO bootstrap "<result>"`
4. If any check fails: log ERROR, notify Robert, do not proceed silently

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

### Known-Excluded Keys
```
OPENCLAW_PROD_DISCORD_APP_ID   — auto-detected from bot token
OPENCLAW_PROD_SAG_KEY          — ElevenLabs skill not active
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

---

## GitHub Repos (NowThatJustMakesSense)

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
| `error-report` | `/error-report` | user | script + LLM |
| `rotate-key` | `/rotate <key>` | user | LLM (guided) |
| `log-decision` | `/decision <text>` | user | LLM |
| `gateway-log-query` | `/gateway-logs` | user | script |
| `incident-manager` | `/incident` | user | script |
| `config-tag` | `/config-tag [label]` | user | script |
| `model-status` | `/model-status` | user | LLM (reads JSON) |
| `model-clear` | `/model-clear <provider>` | user | LLM + jq |
| `log-event` | internal | internal | script |
| `model-failover-notify` | internal | heartbeat | LLM |
| `model-auto-fallback` | internal | heartbeat | LLM |

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

---

## Nightly Cron (03:00 UTC)

Automated via OpenClaw cron system. Runs:
1. `key-drift-check.sh`
2. `ws-backup.sh`
3. `env-backup.sh`
4. `skills-backup.sh`
5. `repo-health.sh`

Sends single summary to Discord. Logs errors via `log-event.sh`.

---

## Escalation Policy — When to Use `coding-agent`

**Handle yourself (scripts + skills):**
- All script-backed skills
- Reading JSON files (model-health.json, auth-profiles.json)
- Routine heartbeat checks, GitHub pushes
- Creating/closing GitHub Issues via incident-manager

**Escalate to Claude Code:**
- handler.ts modification (TypeScript in gateway process)
- Hook loader/import issues
- Permission errors on system files
- openclaw.json structural changes
- Creating new hooks or scripts
- Debugging host + container boundary issues
- Any change requiring gateway restart

See `CLAUDE-CODE-BRIDGE.md` for how to call effectively.

---

## GitHub Sync Policy

After any infrastructure change:
1. Run `ws-backup.sh` → openclaw-workspace
2. Run `skills-backup.sh` → openclaw-skills
3. Run `config-tag.sh <label>` → tag in openclaw-config
4. If openclaw.json changed: update redacted structure in openclaw-config

---

## Log Sources (in order of richness)

| Source | Location | Format | Query tool |
|--------|----------|--------|------------|
| Gateway log | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` | JSON | `gateway-log-query.sh` |
| Model health | `~/.openclaw/model-health.json` | JSON | `cat` + `jq` |
| Notifications | `~/.openclaw/model-health-notifications.jsonl` | JSONL | `tail` + `jq` |
| Repo-Man log | `workspace-spec-github/logs/repo-man.log` | text | `grep` |
| ERRORS.md | openclaw-config/logs/ERRORS.md | markdown | GitHub web |
| LAST_RUN.md | openclaw-config/logs/LAST_RUN.md | markdown | GitHub web |
| GitHub Issues | openclaw-config issues | — | `incident-manager.sh list` |

**Always try gateway log first** — it has structured JSON with every API call, error, and model fallback.

---

## Reference Documents

- `SYSTEM-REFERENCE.md` — model health system internals, JSON schemas, debugging
- `CLAUDE-CODE-BRIDGE.md` — how to work with Claude Code
- `CHANGELOG.md` (in Captain/Relay workspaces) — all infrastructure changes

---

## Rules

- **Use scripts for deterministic tasks.** Never re-implement script logic in LLM output.
- Never truncate logs. Append only.
- Never commit secrets. Template/redact first.
- Always verify after push: `gh api repos/NowThatJustMakesSense/<repo>`
- Always update LAST_RUN.md even when a skill fails — especially when it fails.
- If gh CLI auth fails: log FATAL, stop, notify Robert.
- jq is installed. Use it for JSON manipulation.
- Runtime installs (jq, etc.) vanish on container rebuild — flag for Dockerfile if critical.
