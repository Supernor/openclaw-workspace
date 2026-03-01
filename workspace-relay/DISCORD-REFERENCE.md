# Discord Reference

## Ops Channels

| Channel | ID | Purpose |
|---------|----|---------|
| #ops-dashboard | `1477754431780028598` | Pinned live status |
| #ops-alerts | `1477754571697688627` | Failures with action buttons |
| #ops-nightly | `1477754636046831738` | Nightly cron report |
| #ops-changelog | `1477754637527290030` | Infrastructure changes |
| #ops-github | `1477754638290649209` | GitHub webhook feed |

**Category:** `🔧 OPERATIONS` (`1477754392584126675`)
**Guild:** `1477115265300037703`
**Dashboard pinned message:** `1477754773951352903`

## Reading Ops Channels

When Robert asks about system health, nightly results, or recent changes, check ops channels first before dispatching to Repo-Man:

- "is everything ok?" → read #ops-dashboard
- "what broke?" → read #ops-alerts
- "what happened last night?" → read #ops-nightly
- "what changed?" → read #ops-changelog
- "what was pushed?" → read #ops-github

If the answer is in an ops channel, read it and summarize for Robert. Only dispatch to Repo-Man if Robert needs an action (clear, fix, investigate).

## Interpreting Alert Buttons

Alerts in #ops-alerts have buttons:
- **"Clear Quarantine"** → run `/model-clear <provider>`
- **"View Logs"** → run `gateway-log-query.sh --errors --limit 10`
- **"Silence 1h"** → update cursor to skip provider for 1 hour

When Robert clicks a button, you receive the interaction. Execute the action and reply in a thread.
When you see a ✅ reaction on an alert, treat it as `/model-clear` for that provider.

## Sending Messages

You can send messages with buttons for quick actions:
```json
{
  "action": "send",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "components": {
    "text": "Your message",
    "reusable": true,
    "blocks": [
      {
        "type": "actions",
        "buttons": [
          { "label": "Yes", "style": "success" },
          { "label": "No", "style": "danger" }
        ]
      }
    ]
  }
}
```
**Styles:** success (green), danger (red), primary (blue), secondary (gray). Max 5 buttons/row.

## Polls and Threads

**Polls** — for decisions, preferences, votes:
```json
{
  "action": "poll",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "question": "Question text",
  "answers": ["Option A", "Option B"],
  "durationHours": 24,
  "multiSelect": false
}
```

**Threads** — for investigations or discussions:
```json
{
  "action": "thread-create",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "messageId": "<MESSAGE_ID>",
  "name": "Thread title",
  "autoArchiveDuration": 1440
}
```
