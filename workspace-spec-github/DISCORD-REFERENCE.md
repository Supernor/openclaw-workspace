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

## Component Patterns — Full Reference

### Buttons
```json
{
  "action": "send",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "components": {
    "text": "Your message here",
    "reusable": true,
    "blocks": [
      {
        "type": "actions",
        "buttons": [
          { "label": "Approve", "style": "success" },
          { "label": "Reject", "style": "danger" },
          { "label": "Details", "style": "primary" },
          { "label": "Skip", "style": "secondary" }
        ]
      }
    ]
  }
}
```
**Styles:** `success` (green), `danger` (red), `primary` (blue), `secondary` (gray)
**Callback:** Agent receives `Clicked "Approve".` as a message.
**Limit:** Max 5 buttons per row.

### Select Menus
```json
{
  "type": "actions",
  "select": {
    "type": "string",
    "placeholder": "Pick a provider",
    "options": [
      { "label": "Google", "value": "google", "description": "Gemini models" },
      { "label": "Anthropic", "value": "anthropic", "description": "Claude models" }
    ]
  }
}
```
**Types:** `string`, `user`, `role`, `mentionable`, `channel`

### Modals (Forms)
```json
{
  "components": {
    "text": "Click to open form",
    "modal": {
      "title": "Report Issue",
      "triggerLabel": "Open Form",
      "triggerStyle": "primary",
      "fields": [
        { "type": "text", "name": "description", "label": "What happened?", "required": true, "style": "paragraph" },
        { "type": "select", "label": "Severity", "options": [
          { "label": "Low", "value": "low" },
          { "label": "High", "value": "high" },
          { "label": "Critical", "value": "critical" }
        ]}
      ]
    }
  }
}
```
**Max fields:** 5 per modal

### Container Styling
```json
{
  "components": {
    "container": { "accentColor": 5814783 },
    "blocks": [...]
  }
}
```
**Colors:** Green: `5763719`, Yellow: `16776960`, Red: `15548997`, Blue: `5814783`

## Webhooks

Discord webhook URL format: `https://discord.com/api/webhooks/{id}/{token}`
GitHub integration format: `https://discord.com/api/webhooks/{id}/{token}/github`

Currently configured:
- `#ops-github` webhook: receives push, issues, create events from all 3 repos

To create new webhooks:
```bash
curl -X POST "https://discord.com/api/v10/channels/{channelId}/webhooks" \
  -H "Authorization: Bot $OPENCLAW_PROD_DISCORD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Webhook Name"}'
```

## Bot Presence

```json
{
  "action": "set-presence",
  "channel": "discord",
  "status": "online",
  "activityType": "watching",
  "activityName": "4 healthy providers"
}
```
**Statuses:** `online` (green), `idle` (yellow), `dnd` (red), `invisible`
**Activity types:** `playing`, `streaming`, `listening`, `watching`, `custom`, `competing`
**Note:** Requires gateway WS connection. Works from agent sessions, not HTTP API.

## Channel Management

| Action | What it does |
|--------|-------------|
| `channel-create` | Create text/voice/forum channel |
| `channel-edit` | Modify channel settings |
| `channel-delete` | Delete a channel |
| `channel-info` | Get channel metadata |
| `channel-list` | List all channels |
| `channel-move` | Move to different category |
| `category-create` | Create a category |
| `category-edit` | Edit category |
| `category-delete` | Delete empty category |
| `channel-permissions` | Set permissions |

## Scheduled Events

```json
{
  "action": "event-create",
  "channel": "discord",
  "guildId": "1477115265300037703",
  "name": "Planned Maintenance",
  "description": "Description here.",
  "scheduledStartTime": "2026-03-02T03:00:00Z",
  "scheduledEndTime": "2026-03-02T03:30:00Z",
  "entityType": 3,
  "location": "VPS"
}
```
**Entity types:** 1=stage, 2=voice, 3=external
