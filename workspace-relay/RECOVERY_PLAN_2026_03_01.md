---
OpenClaw Context Recovery — 2026-03-01
This replaces all previous context recovery documents. If you're a new session (Claude Code, agent, or chat) helping Robert with OpenClaw, read this first. Treat any older context recovery docs as obsolete.
---

Platform
- VPS: Hostinger, Ubuntu 24.04
- IP: 187.77.193.174
- Runtime: Docker container (openclaw-openclaw-gateway-1)
- OpenClaw version: 2026.2.27
- Node: 22+
- Config: /home/node/.openclaw/openclaw.json (inside container)
- Env: /app/.env (host: /root/openclaw/.env)
- Repo: /root/openclaw/ (host)

Architecture — Multi-Agent (3 layers)
Robert (Discord) → Relay → Captain → Specialists

┌───────────────┬───────────────┬─────────────────────────────────────────┬──────────────────────────┬───────────┐
│ Agent         │ ID            │ Role                                    │ Workspace                │ Context   │
│               │               │                                         │                          │ Size      │
├───────────────┼───────────────┼─────────────────────────────────────────┼──────────────────────────┼───────────┤
│               │ relay         │ Human interface. Only agent that talks  │                          │           │
│ Relay         │ (default)     │ to Robert. Translates human↔task.       │ workspace-relay/         │ ~60 lines │
│               │               │ Learns Robert's preferences.            │                          │           │
├───────────────┼───────────────┼─────────────────────────────────────────┼──────────────────────────┼───────────┤
│               │               │ Router/dispatcher. Receives structured  │                          │           │
│ Captain       │ main          │ tasks from Relay, routes to             │ workspace/               │ ~30 lines │
│               │               │ specialists. No personality.            │                          │           │
├───────────────┼───────────────┼─────────────────────────────────────────┼──────────────────────────┼───────────┤
│ Quartermaster │ spec-projects │ Project management. Decisions,          │ workspace-spec-projects/ │ ~40 lines │
│               │               │ channels, audits, archiving.            │                          │ + skills  │
├───────────────┼───────────────┼─────────────────────────────────────────┼──────────────────────────┼───────────┤
│ Repo-Man      │ spec-github   │ Infrastructure ops. Keys, backups,      │ workspace-spec-github/   │ ~80 lines │
│               │               │ drift checks, GitHub repos.             │                          │ + skills  │
└───────────────┴───────────────┴─────────────────────────────────────────┴──────────────────────────┴───────────┘

Routing Flow
1. Robert sends a message in Discord
2. Relay receives it (default agent, requireMention: false)
3. Relay translates into a structured task and dispatches to Captain
4. Captain routes to the correct specialist
5. Specialist executes with minimal context, returns structured result
6. Captain forwards result to Relay
7. Relay formats for Robert and responds in Discord

Agent-to-Agent Communication
- Enabled in config: tools.agentToAgent.enabled: true
- Relay has allowAgents: [main, spec-projects, spec-github]
- All agents in A2A allow list: [main, spec-github, spec-projects, relay]

Key Design Principles
- Relay is the ONLY agent that speaks "human" — all others use structured task/result format
- Each agent loads only the context it needs (prevents model timeouts from large bootstrap payloads)
- Specialists never talk to Robert directly
- Architecture supports future additional Relay agents (e.g., Robert's wife) — same specialists, different human preferences

Models
┌────────────┬───────────────────────────────┬───────────────────┐
│ Priority   │ Model                         │ Role              │
├────────────┼───────────────────────────────┼───────────────────┤
│ Primary    │ google/gemini-3-flash-preview │ Daily driver      │
├────────────┼───────────────────────────────┼───────────────────┤
│ Fallback 1 │ google/gemini-3.1-pro-preview │ Project reasoning │
├────────────┼───────────────────────────────┼───────────────────┤
│ Fallback 2 │ openai-codex/gpt-5.3-codex    │ System ops        │
├────────────┼───────────────────────────────┼───────────────────┤
│ Fallback 3 │ openrouter/auto               │ Emergency         │
└────────────┴───────────────────────────────┴───────────────────┘
Cost control: Default thinking level is LOW. Subagents run at LOW with max 8 concurrent.

Discord
- Bot: @The Captain (app ID 1477403010068910171)
- Guild: 1477115265300037703
- User: 187662930794381312 (Robert)
- Bot permissions: Administrator
- Group policy: Allowlist (guild-level)
- Require mention: false (bot responds to all messages from allowlisted users)
- Streaming: off
- Channels: #general, #openclaw-setup (in Projects category)

Project Management Skills (Quartermaster)
┌─────────────────────────┬───────────────────────────────────────────────────────────────────────────────┐
│ Command                 │ What it does                                                                  │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /decide <status> <text> │ Log a decision (DONE, WONT-WORK, UNDECIDED, SAVE-FOR-LATER, DECIDED-NOT-DONE) │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /decisions [filter]     │ Display decision board for current channel                                    │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /pin                    │ Pin decision board as Discord message                                         │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /audit [deep]           │ Cross-reference decisions against chat + uploaded files                       │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /project <name>         │ Create new project channel in Projects category                               │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /archive [force]        │ Archive completed project to monthly Archive category                         │
├─────────────────────────┼───────────────────────────────────────────────────────────────────────────────┤
│ /topic [text]           │ Show/set channel scope                                                        │
└─────────────────────────┴───────────────────────────────────────────────────────────────────────────────┘
Rules: Decisions scoped to project channels only. /decide in general chat redirects.

Repo-Man Skills (Infrastructure)
┌───────────────────┬──────────────────────────────────────────────────────┐
│ Command           │ What it does                                         │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /key-drift        │ Compare .env against 7 canonical keys                │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /ws-backup        │ Commit + push workspace MD files                     │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /env-backup       │ Push .env.template (key names only, never values)    │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /repo-health      │ Verify 3 GitHub repos reachable, check secrets count │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /rotate <key>     │ Guided key rotation with backup, restart, verify     │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /error-report [N] │ Pull recent WARN+ errors from log                    │
├───────────────────┼──────────────────────────────────────────────────────┤
│ /decision <text>  │ Append to infra DECISIONS.md audit trail             │
└───────────────────┴──────────────────────────────────────────────────────┘

Environment Keys (.env)
Canonical keys (7 — managed by Repo-Man drift check):
OPENCLAW_GATEWAY_TOKEN
OPENAI_API_KEY
GH_TOKEN
OPENCLAW_PROD_ANTHROPIC_KEY
OPENCLAW_PROD_GOOGLE_AI_KEY
OPENCLAW_PROD_OPENROUTER_KEY
OPENCLAW_PROD_DISCORD_TOKEN

Known-excluded (present but intentionally not in canonical list):
OPENCLAW_PROD_DISCORD_APP_ID — Discord auto-detects from bot token
OPENCLAW_PROD_SAG_KEY — ElevenLabs skill not active

GitHub Secrets (openclaw-config repo):
9 total — all 7 canonical + APP_ID + SAG_KEY

GitHub Repos (NowThatJustMakesSense)
┌────────────────────┬─────────────────────────────────────────────────┬──────────────────────────────────────────┐
│ Repo               │ Purpose                                         │ Status                                   │
├────────────────────┼─────────────────────────────────────────────────┼──────────────────────────────────────────┤
│ openclaw-config    │ .env.template, logs (DECISIONS, ERRORS,         │ Live, seeded                             │
│                    │ LAST_RUN)                                       │                                          │
├────────────────────┼─────────────────────────────────────────────────┼──────────────────────────────────────────┤
│ openclaw-workspace │ Workspace MD backups                            │ Created, not yet populated by backup     │
│                    │                                                 │ skill                                    │
├────────────────────┼─────────────────────────────────────────────────┼──────────────────────────────────────────┤
│ openclaw-skills    │ Custom skills                                   │ Created, not yet populated               │
└────────────────────┴─────────────────────────────────────────────────┴──────────────────────────────────────────┘
Auth: GH_TOKEN via env var, gh CLI authenticated as NowThatJustMakesSense

File Inventory (inside container, /home/node/.openclaw/)
workspace-relay/ (Relay — human interface)
  SOUL.md, AGENTS.md, USER.md, CLAUDE.md→AGENTS.md
  memory/
  decisions/
  projects/

workspace/ (Captain — router)
  SOUL.md, AGENTS.md, CLAUDE.md→AGENTS.md

workspace-spec-projects/ (Quartermaster — project management)
  SOUL.md, AGENTS.md, CLAUDE.md→AGENTS.md
  skills/{decide,decisions,pin-decisions,audit,project,archive,topic}/skill.md
  decisions/
  projects/

workspace-spec-github/ (Repo-Man — infrastructure)
  SOUL.md, AGENTS.md, TOOLS.md, BOOTSTRAP.md, MEMORY.md, IMPROVEMENTS.md
  skills/{log-event,key-drift-check,workspace-backup,env-backup, repo-health,rotate-key,error-report,log-decision}/skill.md
  logs/repo-man.log

openclaw-config/ (git clone)

Known Issues / Open Items
- Gemini Flash timeouts: Intermittent — model hangs without error, fallback chain doesn't trigger (timeout, not rejection). Reduced context sizes help.
- Nightly cron not configured: workspace-backup, env-backup, repo-health should run at 03:00 UTC (ENH-005)
- session-wrap skill not wired: (ENH-004)
- Audit attachment reading: /audit skill instructs agent to read uploaded files via web_fetch but agent may not extract Discord attachment URLs from message metadata
- Relay preference learning: memory/robert-prefs.md not yet created — will populate as Relay learns

Container Commands (run from /root/openclaw/ on host)
docker compose exec openclaw-gateway <cmd> # run as node
docker compose exec --user root openclaw-gateway <cmd> # run as root
docker compose restart openclaw-gateway # restart (required after config changes)
docker compose logs -f openclaw-gateway # follow logs
docker logs openclaw-openclaw-gateway-1 --tail 50 # recent logs

Owner
Robert Supernor | nowthatjustmakessense@gmail.com | GitHub: NowThatJustMakesSense

---
Generated 2026-03-01T09:20Z by Claude Code. This is the current source of truth.
