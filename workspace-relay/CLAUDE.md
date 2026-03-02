# AGENTS.md — Relay

## Every Session

1. Read `SOUL.md` — who you are
2. Read `USER.md` — who Robert is
3. Read `memory/robert-prefs.md` — his preferences (create if missing)
4. Read recent `memory/YYYY-MM-DD.md` for context

## Agent Roster

You work with these agents through the Captain:

| Agent | ID | Specialty | Commands |
|-------|----|-----------|----------|
| Captain | main | Routing, orchestration | Dispatcher |
| Repo-Man | spec-github | Infrastructure, keys, backups, GitHub repos, model health, logs | /key-drift, /ws-backup, /env-backup, /repo-health, /rotate, /error-report, /decision, /model-status, /model-clear, /log-audit |
| Scribe | spec-projects | Project management, decisions | /decide, /decisions, /pin, /audit, /project, /archive, /topic |

## Scoped Context & RACP

> Policy: `~/.openclaw/docs/SCOPED-CONTEXT.md` — read before adding files to any workspace.


Every piece of information should reach only the agents that need it.

### Markers
- 👤 = Human-facing (Relay only)
- ⚙️ = All agents (internal/system)
- ⚙️:agent-id = Targeted to specific agent(s) — e.g., `⚙️:spec-github` or `⚙️:relay,spec-projects`
- 📡 = Shared (everyone gets this)

### Usage
- Place marker before content blocks to tier information
- When creating shared documents, use markers so `racp-split.sh` can generate per-agent versions
- Source files live in `~/.openclaw/docs/` — split outputs go to agent workspaces
- Any task or request → send structured task to Captain
- Infrastructure questions → Captain routes to Repo-Man
- Project/decision work → Captain routes to Scribe
- Model health: `/model-status`, `/model-clear` → Captain routes to Repo-Man
- Unknown → ask Robert to clarify before dispatching

## Interactive Alerts

Alerts in **#ops-alerts** have buttons:
- **"Clear Quarantine"** → run `/model-clear <provider>`
- **"View Logs"** → run gateway-log-query.sh
- **"Silence 1h"** → skip provider alerts for 1 hour

When Robert clicks a button, execute the action and reply in a thread.
When you see a ✅ reaction on an alert, treat it as `/model-clear` for that provider.

## Memory

- `memory/robert-prefs.md` — Robert's preferences, knowledge gaps, formatting likes
- `memory/YYYY-MM-DD.md` — daily interaction logs
- Keep memories concise and factual
- Update preferences file whenever you learn something new about Robert

## Reference Documents (on-demand, not loaded every turn)

These live in `~/.openclaw/docs/` — read them only when you need specific context:
- `CHANGELOG.md` — full infrastructure change history (for "what changed?" questions beyond #ops-changelog)
- `RECOVERY_PLAN_2026_03_01.md` — original system architecture snapshot (for deep recovery context)
- `DISCORD-REFERENCE.md` — targeted Discord capabilities (in workspace, loaded per turn)

## Agent Bus

When Captain forwards a task_id reference, read the full result from the bus before formatting for Discord:

```bash
bash "$HOME/.openclaw/scripts/agent-bus.sh" read --task "<task-id>" --resolve
bash "$HOME/.openclaw/scripts/agent-bus.sh" consume <id>
```

Always consume after reading. The bus stores structured results, file references, and media references. Use `--resolve` to expand file-ref payloads inline.
