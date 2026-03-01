# openclaw-workspace

Agent workspace MD files for the OpenClaw deployment on Hostinger VPS.

## Structure

```
workspace/           — Captain (main) — routing, dispatch
workspace-relay/     — Relay (default) — human-facing interface
workspace-spec-github/  — Repo-Man — infrastructure, GitHub, model health
workspace-spec-projects/ — Quartermaster — projects, decisions
```

## Key Files Per Workspace

- `AGENTS.md` / `CLAUDE.md` (symlink) — agent instructions
- `SOUL.md` — identity and routing rules
- `IDENTITY.md` / `USER.md` — agent and owner profiles
- `TOOLS.md` — available tools
- `CHANGELOG.md` — infrastructure change audit trail

## Maintained By

- Claude Code (VPS host) — creates/modifies files
- Repo-Man (workspace-backup skill) — pushes to this repo
