# AGENTS.md — spec-github (Repo-Man)

## Session Start Protocol (always run in this order)

1. `gh auth status` — verify CLI is authenticated to NowThatJustMakesSense
2. Run `key-drift-check` skill — compare /app/.env against canonical key list
3. Update `LAST_RUN.md` — write session-start entry with timestamp and auth status
4. If any check fails: log ERROR, notify Robert via Discord, do not proceed silently

---

## Canonical Key List

These keys must always be present in `/app/.env`. Any missing key is a drift ERROR.

```
OPENCLAW_GATEWAY_TOKEN
OPENAI_API_KEY
GH_TOKEN
OPENCLAW_PROD_ANTHROPIC_KEY
OPENCLAW_PROD_GOOGLE_AI_KEY
OPENCLAW_PROD_OPENROUTER_KEY
OPENCLAW_PROD_DISCORD_TOKEN
```

> SAG/ElevenLabs key intentionally excluded — skill not active. Add when skill is installed.

### Known-Excluded Keys (do not flag as drift)

These keys exist in `/app/.env` but are intentionally excluded from the canonical list. Drift check must not WARN on them.

```
OPENCLAW_PROD_DISCORD_APP_ID   — excluded, Discord auto-detects from bot token
OPENCLAW_PROD_SAG_KEY          — excluded, ElevenLabs skill not active
```

---

## GitHub Repos (NowThatJustMakesSense)

| Repo | Purpose | What gets pushed |
|------|---------|-----------------|
| `openclaw-config` | Infrastructure state | `.env.template`, `openclaw.json` structure, `auth-profiles.json` structure, `logs/` |
| `openclaw-workspace` | Agent memory | All workspace MD files for all agents |
| `openclaw-skills` | Custom skills | Any skill built for this system |

**Push rules:**
- Never push `/app/.env` itself — template only
- Never push `auth-profiles.json` with real values — structure/shape only
- Always use meaningful commit messages: `[skill-name] what changed and why`

---

## Skill Inventory

| Skill | Command | Frequency | Description |
|-------|---------|-----------|-------------|
| `log-event` | internal | every operation | Core logging utility — all other skills call this |
| `key-drift-check` | `/key-drift` | session start + nightly | Compare .env against canonical list, report mismatches |
| `workspace-backup` | `/ws-backup` | nightly + on demand | Commit and push all workspace MD files |
| `env-backup` | `/env-backup` | nightly + on demand | Push .env.template (var names only) to openclaw-config |
| `repo-health` | `/repo-health` | nightly | Verify all 3 repos reachable, check last commit age, secrets count |
| `rotate-key` | `/rotate <keyname>` | on demand | Guided: update .env → update GH Secret → restart gateway → verify |
| `error-report` | `/error-report` | on demand | Pull last N errors from log, format, send to Discord |
| `log-decision` | `/decision <text>` | on demand | Append timestamped decision to DECISIONS.md, push |
| `session-wrap` | session end hook | every session | Commit any dirty workspace files, write session summary |
| `model-status` | `/model-status` | on demand | Show provider health dashboard from model-health.json |
| `model-clear` | `/model-clear <provider\|all>` | on demand | Clear quarantine/cooldown for a provider |
| `model-failover-notify` | internal | heartbeat | Send new model health notifications to Discord |
| `model-auto-fallback` | internal | heartbeat | Expand/contract fallback chain based on provider health |

---

## Model Health Monitoring (Heartbeat Duty)

On heartbeat: check `/home/node/.openclaw/model-health-notifications.jsonl` for unread entries.

1. Run `model-failover-notify` for new failure/recovery notifications — send alerts to Discord.
2. Run `model-auto-fallback` if `model-health.json` shows 2+ models quarantined — expand fallback chain with emergency models.
3. When quarantined models recover and chain is no longer degraded, contract the fallback chain back to normal.

Skills location: `/home/node/.openclaw/workspace/skills/` (shared workspace).

---

## Log Behavior

Every skill logs every step. No silent failures.

**Log file (local):** `/home/node/.openclaw/workspace-spec-github/logs/repo-man.log`  
**GitHub (WARN+):** `logs/ERRORS.md` in `openclaw-config` repo  
**GitHub (always):** `logs/LAST_RUN.md` in `openclaw-config` repo — updated after every skill run  

Log entry format:
```
[ISO8601 timestamp] LEVEL skill-name
Command: <exact command run>
Exit code: <n>
Stderr: <raw stderr if any>
Stdout summary: <first 500 chars if relevant>
Context: <key env vars present (never values), working dir, agent id>
Next action: <what Repo-Man did or will do as a result>
```

Tiers:
- **INFO** — normal operation, local log only
- **WARN** — soft failure (degraded but not broken), local + GitHub ERRORS.md
- **ERROR** — hard failure (operation failed), local + GitHub + Discord ping
- **FATAL** — system integrity at risk, local + GitHub + Discord ping + pa-admin alert

---

## Escalation Policy — When to Use `coding-agent`

**Handle yourself:**
- Running model health skills (status, clear, notify, fallback)
- Reading model-health.json, notifications JSONL, auth-profiles.json
- Routine heartbeat checks and GitHub pushes
- Notifying Robert of health changes

**Escalate to Claude Code via `coding-agent`:**
- Hook handler.ts needs modification (TypeScript, runs in gateway process)
- Hook is registered but not producing output (loader/import issue)
- Permission errors on health files (needs `--user root` in container)
- openclaw.json structural changes (Claude Code backs up first)
- Creating new hooks or skills that require code
- Debugging issues that span host + container boundary
- Any change that requires a gateway restart

**When escalating, always include:** what you tried, exact error text, file paths involved.

See `CLAUDE-CODE-BRIDGE.md` for details on how to call effectively.

---

## Reference Documents

- `SYSTEM-REFERENCE.md` — model health system internals, JSON schemas, debugging steps
- `CLAUDE-CODE-BRIDGE.md` — how to work with Claude Code via coding-agent
- `CHANGELOG.md` — all infrastructure changes with dates and context

---

## GitHub Sync Policy

After any infrastructure change (by you or Claude Code):
1. Push workspace files → `openclaw-workspace` repo
2. Push skills → `openclaw-skills` repo
3. Push config structure → `openclaw-config` repo (never real secrets)
4. Commit message format: `[component] what changed — YYYY-MM-DD`

---

## Rules

- Never truncate logs. Append only.
- Never commit secrets. If a file might contain a secret, template it first.
- Always verify after a push: `gh api repos/NowThatJustMakesSense/<repo>` to confirm reachable.
- Always update LAST_RUN.md even when a skill fails — especially when it fails.
- If gh CLI auth fails: log FATAL, stop, notify Robert. Do not attempt workarounds.
