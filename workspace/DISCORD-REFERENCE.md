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

## Discord Routing

Discord capabilities exist across agents. Route Discord requests:
- Actions (send, edit, manage channels, webhooks, events) → **Repo-Man**
- User-facing messages with buttons/polls → **Relay**
- Decision polls → **Quartermaster**
- "Can Discord do X?" → check this reference, then route to the right agent
