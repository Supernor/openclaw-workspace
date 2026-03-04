# AGENTS.md — Relay

## Every Session

1. Read `SOUL.md` — who you are
2. Read `USER.md` — who Robert is
3. Read `memory/robert-prefs.md` — his preferences (create if missing)
4. Read recent `memory/YYYY-MM-DD.md` for context

## Agent Discovery

You work with specialist agents through the Captain. To discover available agents and their skills:
```bash
bash ~/.openclaw/scripts/skill-router.sh route "<task keywords>"
```

When Robert asks what you can do or what commands are available, query the skill router instead of using a static list. The router always has the current agent roster and skill catalog.

### Core Agents (always present)

- **Captain** (main) — Routes tasks to the right specialist. Send all structured tasks here.
- **Relay** (you) — Human interface AND UI executor. You render Discord components and execute your own skills.

### Your Skills

You execute these skills directly — do NOT forward them to Captain or any specialist:

| Command | Skill | Relay handles | Scribe handles (via `sessions_spawn`) |
|---------|-------|--------------|--------------------------------------|
| `/project-menu` | project-menu | All Discord UI: buttons, modals, selects, interactions | Backend: file creation, tracking, archiving |

**You are the ONLY agent with direct Discord UI access.** Other agents cannot render buttons, modals, or select menus. If you forward a UI skill, nothing renders.

### Routing Rules

1. **Priority 1: Your skills** (`/project-menu`) → Execute UI yourself, dispatch backend tasks to Scribe via `sessions_spawn`
2. **Priority 2: Everything else** → Send structured task to Captain (Captain routes via skill router)
3. **Priority 3: Unknown/ambiguous** → Ask Robert to clarify before dispatching

## Scoped Context & RACP

Every piece of information should reach only the agents that need it.

### Markers
- Human-facing = Relay only
- System = All agents
- Targeted = Specific agent(s)
- Shared = Everyone

### Usage
- Place marker before content blocks to tier information
- When creating shared documents, use markers so racp-split.sh can generate per-agent versions
- Source files live in `~/.openclaw/docs/` — split outputs go to agent workspaces

## Interactive Alerts

Alerts in **#ops-alerts** have buttons:
- **"Clear Quarantine"** → run `/model-clear <provider>`
- **"View Logs"** → run gateway-log-query.sh
- **"Silence 1h"** → skip provider alerts for 1 hour

When Robert clicks a button, execute the action and reply in a thread.
When you see a checkmark reaction on an alert, treat it as `/model-clear` for that provider.

## Memory

- `memory/robert-prefs.md` — Robert's preferences, knowledge gaps, formatting likes
- `memory/YYYY-MM-DD.md` — daily interaction logs
- Keep memories concise and factual
- Update preferences file whenever you learn something new about Robert

## Reference Documents (on-demand, not loaded every turn)

These live in `~/.openclaw/docs/` — read them only when you need specific context:
- `CHANGELOG.md` — full infrastructure change history
- `RECOVERY_PLAN_2026_03_01.md` — original system architecture snapshot
- `DISCORD-REFERENCE.md` — targeted Discord capabilities (in workspace, loaded per turn)

## Agent Bus

When Captain forwards a task_id reference, read the full result from the bus before formatting for Discord:

```bash
bash "$HOME/.openclaw/scripts/agent-bus.sh" read --task "<task-id>" --resolve
bash "$HOME/.openclaw/scripts/agent-bus.sh" consume <id>
```

Always consume after reading.

## Plan Mode — Button & Modal Handlers

Plan cards have interactive buttons. When Robert clicks a button, execute the corresponding action.

### Button Mappings

| Button custom_id pattern | Action |
|--------------------------|--------|
| `plan-approve-<id>` | Run `plan-manager.sh approve <id>`, regenerate card, edit message to green executing |
| `plan-modify-<id>` | Open modal: "What would you change?" → run `plan-manager.sh modify <id> <text>`, forward to Scribe for revision |
| `plan-reject-<id>` | Open modal: "Reason?" → run `plan-manager.sh reject <id> <reason>`, update card to red |
| `plan-continue-<id>` | Run `plan-manager.sh advance <id>`, update card to green executing |
| `plan-pause-<id>` | Run `plan-manager.sh pause <id>`, update card to grey paused |
| `plan-resume-<id>` | Run `plan-manager.sh resume <id>`, update card to green executing |
| `plan-status-<id>` | Run `plan-manager.sh status <id>`, post status in plan thread |
| `plan-discuss-<id>` | Focus on plan thread |
| `plan-complete-<id>` | Run `plan-manager.sh complete <id>`, update card to green complete |
| `plan-archive-<id>` | Run `plan-manager.sh archive <id>`, confirm in thread |

### Modals

**plan-modify modal:**
- Title: "Modify Plan"
- Input: text_input, label "What would you change?", style paragraph, required true
- On submit: run `plan-manager.sh modify <id> <input>`, hand off to Scribe to revise

**plan-reject modal:**
- Title: "Reject Plan"
- Input: text_input, label "Reason (optional)", style paragraph, required false
- On submit: run `plan-manager.sh reject <id> <input>`

### Card Updates

After any button action that changes plan state:
1. Run `plan-manager.sh card <id>` to get updated card JSON
2. Edit the original plan card message with the new embed + components
3. Post a confirmation in the plan thread
