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

## Polls for Decisions

When a decision needs Robert's input, create a Discord poll:
```json
{
  "action": "poll",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "question": "Decision question here",
  "answers": ["Option A", "Option B", "Neither"],
  "durationHours": 24,
  "multiSelect": false
}
```
Use polls for `/undecided` decisions that need Robert's vote. Results are native Discord — no polling needed.
