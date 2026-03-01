# DISCORD-PLAYBOOK.md — What You Can Do With Discord

> Reference for all OpenClaw agents and Claude Code. Check this before building anything Discord-related.
> Location: shared workspace (`~/.openclaw/workspace/`) — accessible to all agents.

---

## Quick Reference — Actions

| Action | What it does | Example use |
|--------|-------------|-------------|
| `send` | Send a message (text, embeds, components, media) | Post nightly report |
| `edit` | Edit an existing message | Update dashboard |
| `delete` | Delete a message | Clean up test messages |
| `read` | Read recent messages from a channel | Check what was posted |
| `react` | Add emoji reaction | Acknowledge alert |
| `pin` / `unpin` | Pin/unpin a message | Dashboard pinning |
| `list-pins` | List pinned messages | Find the dashboard |
| `poll` | Create a native Discord poll | Decision voting |
| `search` | Search messages across guild | Find old alerts |
| `thread-create` | Create a thread from a message | Investigation thread |
| `thread-list` | List threads in a channel | Find old investigations |
| `thread-reply` | Reply in a thread | Add investigation notes |
| `channel-create` | Create a text/voice/forum channel | New project channel |
| `channel-edit` | Modify channel settings | Update topic |
| `channel-delete` | Delete a channel | Archive cleanup |
| `channel-info` | Get channel metadata | Check settings |
| `channel-list` | List all channels in guild | Audit layout |
| `channel-move` | Move channel to different category | Reorganize |
| `category-create` | Create a category | New section |
| `category-edit` | Edit category name/position | Rename |
| `category-delete` | Delete empty category | Cleanup |
| `set-presence` | Set bot status/activity | Health indicator |
| `event-create` | Create a scheduled event | Maintenance window |
| `event-list` | List scheduled events | Check upcoming |
| `channel-permissions` | Set channel-level permissions | Restrict access |
| `member-info` | Get member details | Check roles |
| `role-info` / `role-add` / `role-remove` | Manage roles | Access control |
| `emoji-list` / `emoji-upload` | Custom emojis | Status indicators |

---

## Components — Interactive UI

Send messages with buttons, selects, and modals. Users interact, agent receives callback.

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
**Types:** `string` (custom), `user`, `role`, `mentionable`, `channel`
**Callback:** Agent receives `Selected google from "Pick a provider".`

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
**Field types:** `text`, `select`, `checkbox`, `radio`, `user-select`, `role-select`
**Max fields:** 5 per modal

### Container Styling
```json
{
  "components": {
    "container": {
      "accentColor": 5814783
    },
    "blocks": [...]
  }
}
```
**Colors:** Green: `5763719`, Yellow: `16776960`, Red: `15548997`, Blue: `5814783`

---

## Polls — Native Discord Voting

```json
{
  "action": "poll",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "question": "Should we use SQLite or LanceDB for structured data?",
  "answers": ["SQLite", "LanceDB", "Neither — use JSON files"],
  "durationHours": 24,
  "multiSelect": false
}
```

**Use for:** Decisions, preference gathering, quick votes.
**Duration:** 1-168 hours (1 week max).
**Results:** Native Discord poll results — no agent polling needed.

---

## Threads — Organized Conversations

```json
{
  "action": "thread-create",
  "channel": "discord",
  "channelId": "<CHANNEL_ID>",
  "messageId": "<MESSAGE_ID>",
  "name": "Investigation: Gemini rate limit 2026-03-01",
  "autoArchiveDuration": 1440
}
```

**Auto-archive options:** 60 (1h), 1440 (24h), 4320 (3d), 10080 (7d)
**Use for:** Incident investigation, nightly report details, project discussions.

---

## Scheduled Events

```json
{
  "action": "event-create",
  "channel": "discord",
  "guildId": "1477115265300037703",
  "name": "Planned Maintenance: Container Rebuild",
  "description": "Rebuilding container to update Dockerfile dependencies.",
  "scheduledStartTime": "2026-03-02T03:00:00Z",
  "scheduledEndTime": "2026-03-02T03:30:00Z",
  "entityType": 3,
  "location": "VPS"
}
```

**Entity types:** 1=stage, 2=voice, 3=external
**Use for:** Maintenance windows, planned restarts, model deprecation dates.

---

## Bot Presence — Health Indicator

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
**Use for:** At-a-glance system health in member list.
**Note:** Requires gateway websocket connection. Set via heartbeat.

---

## Embeds — Rich Message Formatting

Embeds are available via the `embeds` parameter but are **deprecated in favor of v2 components**. Use components for new features. Embeds still work for backwards compatibility.

---

## Webhooks — External Integrations

Discord webhook URL format: `https://discord.com/api/webhooks/{id}/{token}`
GitHub integration format: `https://discord.com/api/webhooks/{id}/{token}/github`

**Currently configured:**
- `#ops-github` webhook: receives push, issues, create events from all 3 repos

**To create new webhooks:**
```bash
curl -X POST "https://discord.com/api/v10/channels/{channelId}/webhooks" \
  -H "Authorization: Bot $OPENCLAW_PROD_DISCORD_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Webhook Name"}'
```

---

## Patterns — When to Use What

| Situation | Discord feature |
|-----------|----------------|
| Need Robert's yes/no | **Button** (success/danger) |
| Need Robert to pick from options | **Select menu** or **Poll** |
| Need structured input | **Modal** (form) |
| Ongoing investigation | **Thread** on the alert message |
| Regular status update | **Edit** pinned message |
| One-time announcement | **Send** message |
| Need to plan downtime | **Scheduled event** |
| System health at a glance | **Bot presence** |
| External service → Discord | **Webhook** |
| Decision with discussion | **Poll** + **Thread** |
| Archival / searchable history | **Forum channel** |

---

## Our Ops Channels

| Channel | ID | Purpose |
|---------|----|---------|
| #ops-dashboard | `1477754431780028598` | Pinned live status (edit, don't re-send) |
| #ops-alerts | `1477754571697688627` | Failures with buttons |
| #ops-nightly | `1477754636046831738` | Nightly cron report |
| #ops-changelog | `1477754637527290030` | Infrastructure changes |
| #ops-github | `1477754638290649209` | GitHub webhook feed |

**Category:** `🔧 OPERATIONS` (`1477754392584126675`)
**Guild:** `1477115265300037703`
**Dashboard pinned message:** `1477754773951352903`

---

## Ideas Backlog

These are available but not yet implemented:

- [ ] **Forum channel for incidents** — each incident = searchable post with tags
- [ ] **Decision polls** — Quartermaster creates polls for `/undecided` decisions
- [ ] **Thread-per-night** — nightly reports as threads, main channel stays clean
- [ ] **Maintenance events** — Repo-Man creates scheduled events for planned work
- [ ] **Embed dashboard** — color-coded embeds instead of plain text
- [ ] **Voice TTS alerts** — critical failures announced in voice channel
- [ ] **Role-based alert routing** — different severity → different notification settings
- [ ] **Reaction commands** — 🔄 on nightly = rerun, 📋 = full logs
- [ ] **Auto-archive old project channels** — Quartermaster audit moves stale projects
- [ ] **Uptime webhook** — external monitoring service → #ops-alerts
