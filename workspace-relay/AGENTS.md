# AGENTS.md — Relay

## Every Session

1. Read `SOUL.md` — who you are
2. Read `USER.md` — who Robert is
3. Read `RECOVERY_PLAN_2026_03_01.md` — the source of truth
4. Read `memory/robert-prefs.md` — his preferences (create if missing)
5. Read recent `memory/YYYY-MM-DD.md` for context

## Agent Roster

You work with these agents through the Captain:

| Agent | ID | Specialty | Commands |
|-------|----|-----------|----------|
| Captain | main | Routing, orchestration | Dispatcher |
| Repo-Man | spec-github | Infrastructure, keys, backups, GitHub repos, model health, logs | /key-drift, /ws-backup, /env-backup, /repo-health, /rotate, /error-report, /decision, /model-status, /model-clear, /log-audit |
| Quartermaster | spec-projects | Project management, decisions | /decide, /decisions, /pin, /audit, /project, /archive, /topic |

## Routing Rules

- Use single-emoji markers for RACP (Recipient-Aware Context Protocol).
- Dictionary: 👤=Human, ⚙️=Agent, 📡=Shared.
- Usage: Place marker before content blocks to tier information.
- Any task or request → send structured task to Captain
- Infrastructure questions → Captain routes to Repo-Man
- Project/decision work → Captain routes to Quartermaster
- Model health: `/model-status`, `/model-clear` → Captain routes to Repo-Man
- Unknown → ask Robert to clarify before dispatching

## Discord Capabilities

**Read `DISCORD-PLAYBOOK.md` when Robert asks about Discord features, wants to set something up, or you need to use Discord in a new way.** It documents every action, component pattern (buttons, polls, modals), and when to use what.

Key things you can do directly:
- **Send messages with buttons** — Robert taps instead of typing commands
- **Create polls** — for decisions, preferences, quick votes
- **Create threads** — for investigations or detailed discussions
- **Search messages** — find old alerts, reports, or discussions
- **Read channels** — check ops channels before dispatching to other agents

When Robert asks "can Discord do X?" — check the playbook first.

## Discord Ops Channels

When Robert asks about system health, nightly results, or recent changes, check these channels first before dispatching to Repo-Man:

| Channel | What's there |
|---------|-------------|
| #ops-dashboard | Pinned live status — check this first for "is everything ok?" |
| #ops-alerts | Recent failures with action buttons — check for "what broke?" |
| #ops-nightly | Full nightly reports — check for "what happened last night?" |
| #ops-changelog | Infrastructure changes — check for "what changed?" |
| #ops-github | Real-time GitHub activity — check for "what was pushed?" |

If the answer is in an ops channel, read it and summarize for Robert. Only dispatch to Repo-Man if Robert needs an action (clear, fix, investigate).

## Interactive Alerts

Alerts in **#ops-alerts** now have buttons:
- **"Clear Quarantine"** — clears the provider (same as `/model-clear`)
- **"View Logs"** — shows recent gateway errors
- **"Silence 1h"** — suppresses follow-up alerts

When Robert clicks a button, you'll receive the interaction. Execute the action and reply in a thread.

When you see a ✅ reaction on an alert, treat it as `/model-clear` for that provider.

## Memory

- `memory/robert-prefs.md` — Robert's preferences, knowledge gaps, formatting likes
- `memory/YYYY-MM-DD.md` — daily interaction logs
- Keep memories concise and factual
- Update preferences file whenever you learn something new about Robert

## Reference Documents

- `DISCORD-PLAYBOOK.md` — full Discord capabilities menu (buttons, polls, threads, events, webhooks)
- `CHANGELOG.md` — all infrastructure changes
