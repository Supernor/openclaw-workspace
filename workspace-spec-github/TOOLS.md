# TOOLS.md — spec-github (Repo-Man)

## Shell Scripts (`~/.openclaw/scripts/`)

Primary tooling. All output structured JSON. Zero LLM tokens for execution.

| Script | Usage |
|--------|-------|
| `key-drift-check.sh` | `./key-drift-check.sh` |
| `env-backup.sh` | `./env-backup.sh` |
| `ws-backup.sh` | `./ws-backup.sh` |
| `skills-backup.sh` | `./skills-backup.sh` |
| `repo-health.sh` | `./repo-health.sh` |
| `log-event.sh` | `./log-event.sh LEVEL SKILL MESSAGE [EXIT_CODE] [STDERR]` |
| `gateway-log-query.sh` | `./gateway-log-query.sh --errors --summary --limit 20` |
| `incident-manager.sh` | `./incident-manager.sh open\|close\|list\|check [args]` |
| `config-tag.sh` | `./config-tag.sh [label]` |

## gh CLI

| Field | Value |
|-------|-------|
| Binary | `/usr/local/bin/gh` |
| Auth | `GH_TOKEN` env var |
| Account | NowThatJustMakesSense |

**Common:**
```bash
gh auth status
gh secret list --repo NowThatJustMakesSense/<repo>
gh secret set <KEY> --body "<value>" --repo NowThatJustMakesSense/<repo>
gh api repos/NowThatJustMakesSense/<repo>
gh issue list --repo NowThatJustMakesSense/openclaw-config --label incident --state open
gh issue create --repo NowThatJustMakesSense/openclaw-config --title "..." --body "..." --label "incident,provider:X"
```

## jq

Installed. Use for JSON manipulation:
```bash
cat file.json | jq '.field'
jq '.usageStats | with_entries(select(.key | startswith("anthropic")))' auth-profiles.json
```

## git

| Repo | Local path | Remote |
|------|-----------|--------|
| openclaw-config | `workspace-spec-github/openclaw-config/` | NowThatJustMakesSense/openclaw-config |
| openclaw-workspace | `workspace-spec-github/openclaw-workspace/` | NowThatJustMakesSense/openclaw-workspace |
| openclaw-skills | `workspace-spec-github/openclaw-skills/` | NowThatJustMakesSense/openclaw-skills |

Auth via `gh auth setup-git` credential helper.

## Log Locations

| Log | Path | Tool |
|-----|------|------|
| Gateway (JSON) | `/tmp/openclaw/openclaw-YYYY-MM-DD.log` | `gateway-log-query.sh` |
| Repo-Man | `workspace-spec-github/logs/repo-man.log` | `grep` |
| Model health | `~/.openclaw/model-health.json` | `jq` |
| Notifications | `~/.openclaw/model-health-notifications.jsonl` | `tail` + `jq` |

## Gateway Control

```bash
# Check status (from inside container)
# NOTE: docker commands must be run from HOST, not container
# Use coding-agent for gateway restarts
```

## Env Vars (keys to care about, never log values)

| Var | Source | Purpose |
|-----|--------|---------|
| `GH_TOKEN` | /app/.env | gh CLI + git auth |
| `OPENCLAW_GATEWAY_TOKEN` | /app/.env | Gateway auth |
| `OPENCLAW_PROD_*` | runtime env | Provider API keys |
| `OPENAI_API_KEY` | /app/.env | Embeddings + OpenAI tools |
