# CLAUDE-CODE-BRIDGE.md — Working with Claude Code

## What Claude Code Is

Claude Code runs on the **host VPS** (not inside the Docker container). It has full filesystem access, Docker CLI, Git, and persistent memory of all OpenClaw internals.

## Shared Registry

Both Claude Code and OpenClaw read `~/.openclaw/registry.json` for all IDs, paths, and constants. **Never hardcode values — always read from the registry.**

## When to Escalate to Claude Code

**DO escalate:**
- TypeScript/code changes (handler.ts, hooks)
- openclaw.json structural changes
- Creating new hooks or scripts
- Container/Docker debugging
- Permission errors on system files
- Any change requiring gateway restart

**DON'T escalate:**
- Running existing scripts or skills
- Reading JSON files
- Routine heartbeat checks
- Discord message sending

## How to Escalate — Inbox/Outbox Protocol

### Step 1: Generate context snapshot
```bash
/home/node/.openclaw/scripts/context-snapshot.sh
```
This writes `~/.openclaw/coding-agent/context/current.json` with full system state (model health, key drift, repo health, recent errors, cron status, disk usage). Zero tokens — it's all file I/O.

### Step 2: Write task to inbox
```bash
cat > ~/.openclaw/coding-agent/inbox/$(date -u +%Y%m%d-%H%M%S)-task.json << 'TASK'
{
  "agent": "<your agent ID>",
  "urgency": "routine|blocking|critical",
  "task": "One-line summary of what you need",
  "context": "What happened, what you tried, why you're escalating",
  "files": ["/path/to/relevant/file1", "/path/to/relevant/file2"],
  "errors": "Exact error text if any",
  "outcome": "What success looks like"
}
TASK
```

### Step 3: Notify Robert
Tell Robert that you've escalated a task to Claude Code. Include the one-line summary. Robert will invoke Claude Code on the host.

### Step 4: Read result from outbox
After Claude Code completes work, it writes to `~/.openclaw/coding-agent/outbox/`:
```json
{
  "timestamp": "2026-03-01T21:15:00Z",
  "task": "Original task summary",
  "status": "completed|partial|blocked",
  "filesChanged": [
    {"path": "/path/to/file", "action": "created|modified|deleted"}
  ],
  "restartNeeded": false,
  "restarted": false,
  "verify": ["Step 1 to verify", "Step 2 to verify"],
  "changelogEntry": "Summary for CHANGELOG.md",
  "notes": "Anything else the agent should know"
}
```

Read the result, run verification steps, then push to GitHub if files changed.

## What Claude Code Already Knows

Claude Code maintains persistent memory covering:
- Container filesystem layout and all agent workspaces
- Hook system internals, auth profile types
- Model health system implementation
- All scripts and their behavior
- Scoped Context policy
- Full change history

**Don't over-explain things Claude Code already knows.** Focus the task on what's NEW — the specific problem, error, or requirement.

## Context Snapshot Contents

`context-snapshot.sh` gathers:
| Field | Source |
|-------|--------|
| modelHealth | model-health.json |
| keyDrift | key-drift-check.sh |
| repoHealth | repo-health.sh |
| recentErrors | gateway-log-query.sh (last 5) |
| cronStatus | cron/jobs.json state |
| recentNotifications | last 5 from notifications JSONL |
| activeIncidents | incident-manager.sh list |
| diskUsageMB | du on .openclaw/ |

Claude Code reads this file instead of running the same scripts again. One file, full awareness.
