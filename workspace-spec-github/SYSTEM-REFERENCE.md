# SYSTEM-REFERENCE.md — Model Health & Failover System

> Created: 2026-03-01 by Claude Code
> Last updated: 2026-03-01

## Overview

The model health system has 3 layers:
1. **Hook** — `model-health-monitor` polls auth-profiles every 30s, writes structured data
2. **Skills** — `model-status`, `model-clear` (user), `model-failover-notify`, `model-auto-fallback` (internal)
3. **Config** — `openclaw.json` hook entry + cooldown tuning

## File Locations

| File | Path | Written by |
|------|------|------------|
| Hook HOOK.md | `~/.openclaw/hooks/model-health-monitor/HOOK.md` | Claude Code |
| Hook handler | `~/.openclaw/hooks/model-health-monitor/handler.ts` | Claude Code |
| Health state | `~/.openclaw/model-health.json` | Hook (atomic write) |
| Notifications | `~/.openclaw/model-health-notifications.jsonl` | Hook (append) |
| Notify cursor | `~/.openclaw/model-health-notify-cursor.txt` | model-failover-notify skill |
| Auth profiles | `~/.openclaw/agents/<id>/agent/auth-profiles.json` | OpenClaw core |
| Skills | `~/.openclaw/workspace/skills/model-*/skill.md` | Claude Code |

## JSON Schemas

### model-health.json
```json
{
  "version": 1,
  "lastChecked": "ISO-8601",
  "providers": {
    "<provider-name>": {
      "status": "healthy | rate-limited | quarantined",
      "reason": "none | billing | rate-limit | auth | errors | unknown",
      "since": "ISO-8601",
      "disabledUntil": "ISO-8601 (optional)",
      "failureCount": 0,
      "profiles": {
        "<profileId>": {
          "status": "ok | cooldown | disabled",
          "errorCount": 0,
          "lastUsed": "ISO-8601",
          "cooldownUntil": "ISO-8601 (optional)",
          "disabledUntil": "ISO-8601 (optional)",
          "disabledReason": "string (optional)",
          "failureCounts": {}
        }
      }
    }
  },
  "fallbackChain": {
    "configured": ["model/name", "..."],
    "quarantined": ["model/name", "..."],
    "emergency": ["model/name", "..."]
  }
}
```

### model-health-notifications.jsonl
```json
{"ts":"ISO-8601","type":"failure|recovery","provider":"name","reason":"string","message":"Human-readable"}
```

## Auth Profile UsageStats Fields

These are the fields in `auth-profiles.json → usageStats → <profileId>` that the hook monitors:

| Field | Type | Meaning |
|-------|------|---------|
| `lastUsed` | epoch ms | Last successful use |
| `cooldownUntil` | epoch ms | Temporary backoff expiry |
| `disabledUntil` | epoch ms | Hard disable expiry |
| `disabledReason` | string | Why disabled: `billing`, `rate_limit`, `auth`, `auth_permanent`, `timeout`, `model_not_found`, `unknown` |
| `errorCount` | number | Consecutive errors |
| `failureCounts` | object | Per-reason error counts |
| `lastFailureAt` | epoch ms | Last error timestamp |

## Provider Status Derivation Logic

1. If any profile has `disabledUntil` in the future → provider = `quarantined`
2. If any profile has `cooldownUntil` in the future → provider = `rate-limited`
3. If any profile has `errorCount >= 3` → provider = `rate-limited`
4. Otherwise → provider = `healthy`

## Hook Behavior

- Triggers on `gateway:startup` event
- Starts a `setInterval` at 30,000ms (30s)
- Reads ALL agent auth-profiles.json files each tick
- Compares current status to previous (held in memory)
- On state change: writes notification to JSONL, updates health JSON
- Atomic writes via temp file + rename
- Notification file rotated at 500 lines
- Timer is `unref()`'d so it doesn't prevent shutdown

## Debugging

### Hook not loading
```bash
# Check if hook is registered in logs
docker compose logs openclaw-gateway --since 5m 2>&1 | grep -i "model-health"

# Check hook files exist and are readable
ls -la ~/.openclaw/hooks/model-health-monitor/

# Check config has hook enabled
cat ~/.openclaw/openclaw.json | jq '.hooks.internal.entries["model-health-monitor"]'
```

### model-health.json not updating
```bash
# Check file exists and last modified time
ls -la ~/.openclaw/model-health.json
stat ~/.openclaw/model-health.json

# Check for errors in logs
docker compose logs openclaw-gateway --since 5m 2>&1 | grep "model-health-monitor.*ERROR"

# Check auth-profiles exist
ls ~/.openclaw/agents/*/agent/auth-profiles.json
```

### Provider incorrectly quarantined
```bash
# Check the raw usageStats
cat ~/.openclaw/agents/relay/agent/auth-profiles.json | jq '.usageStats'

# Clear manually with /model-clear <provider>
# Or clear directly:
# See model-clear skill.md for jq commands
```

### Hook crashing
The hook runs in the gateway process. If it throws, the error is caught and logged but the interval continues. Check:
```bash
docker compose logs openclaw-gateway --since 10m 2>&1 | grep "model-health-monitor.*ERROR\|model-health-monitor.*WARN"
```

## Emergency Backup Models (for auto-fallback)
- `openrouter/anthropic/claude-sonnet-4-5`
- `openrouter/google/gemini-2.0-flash`
- `openrouter/meta-llama/llama-4-maverick`

All route through OpenRouter. If OpenRouter itself is quarantined, no emergency models can be added.

## Cooldown Config (openclaw.json → auth.cooldowns)
- `billingBackoffHours`: 1 (initial backoff for billing failures)
- `billingMaxHours`: 12 (max backoff cap)
- `failureWindowHours`: 6 (window for counting failures)
