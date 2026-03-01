# TOOLS.md — spec-github (Repo-Man)

## gh CLI

| Field | Value |
|-------|-------|
| Binary | `/usr/local/bin/gh` |
| Auth method | `GH_TOKEN` env var |
| Account | NowThatJustMakesSense |
| Scopes | repo, admin:repo_hook, secrets |
| Verify auth | `gh auth status` |

**Common commands:**
```bash
# Secrets
gh secret list --repo NowThatJustMakesSense/<repo>
gh secret set <KEY> --body "<value>" --repo NowThatJustMakesSense/<repo>

# Repo health
gh api repos/NowThatJustMakesSense/<repo>

# Push
git add -A && git commit -m "[skill] message" && git push
```

---

## git

| Repo | Local clone path | Remote |
|------|-----------------|--------|
| openclaw-config | `/home/node/.openclaw/workspace-spec-github/openclaw-config/` | `https://github.com/NowThatJustMakesSense/openclaw-config` |
| openclaw-workspace | `/home/node/.openclaw/workspace-spec-github/openclaw-workspace/` | `https://github.com/NowThatJustMakesSense/openclaw-workspace` |
| openclaw-skills | `/home/node/.openclaw/workspace-spec-github/openclaw-skills/` | `https://github.com/NowThatJustMakesSense/openclaw-skills` |

> If local clones don't exist yet: `git clone https://github.com/NowThatJustMakesSense/<repo>.git <path>`  
> Auth is handled by GH_TOKEN via git credential helper — no separate setup needed.

---

## Env Vars (Repo-Man cares about these)

| Var | Location | Purpose |
|-----|----------|---------|
| `GH_TOKEN` | `/app/.env` | gh CLI + git auth |
| `OPENCLAW_GATEWAY_TOKEN` | `/app/.env` | Gateway auth — verify present, never log value |

**Reading .env (for drift check only):**
```bash
grep -E '^[A-Z_]+=.' /app/.env | cut -d= -f1
```
This extracts key names only — never values.

---

## Log Paths

| File | Purpose |
|------|---------|
| `/home/node/.openclaw/workspace-spec-github/logs/repo-man.log` | Full verbose local log, append-only |
| `openclaw-config/logs/ERRORS.md` | WARN+ entries, human-readable, latest first |
| `openclaw-config/logs/LAST_RUN.md` | Last run of each skill — health at a glance |
| `openclaw-config/logs/DECISIONS.md` | Append-only decision audit trail |

---

## Gateway Control

```bash
# Restart gateway (needed after key rotation)
docker restart openclaw-openclaw-gateway-1

# Verify gateway is up
docker ps | grep openclaw-gateway

# Check gateway logs
docker logs openclaw-openclaw-gateway-1 --tail 50
```

---

## Error → Fix Reference

| Error | Likely cause | Fix |
|-------|-------------|-----|
| `gh auth status` fails | GH_TOKEN expired or missing | Log FATAL, alert Robert, do not proceed |
| `git push` rejected | Branch protection or auth | Check GH_TOKEN scopes, verify remote URL |
| Key missing from .env | Drift or corruption | Log ERROR, alert Robert with specific key name |
| Docker restart fails | Container not running | Log FATAL, check `docker ps -a` |
| Secret count mismatch | Key added/removed without updating canonical list | Log WARN, report to Robert for review |
