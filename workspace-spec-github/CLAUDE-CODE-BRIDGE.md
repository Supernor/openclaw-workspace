# CLAUDE-CODE-BRIDGE.md — Working with Claude Code

> This file tells OpenClaw agents how to effectively use Claude Code via the `coding-agent` skill.

## What Claude Code Is

Claude Code runs on the **host VPS** (not inside the Docker container). It has:
- Full filesystem access to the VPS
- Docker CLI access (can exec into the container, read/write files, restart gateway)
- Git access to the host repo at `/root/openclaw/`
- Persistent memory at `/root/.claude/projects/-root/memory/` with detailed system internals
- Knowledge of OpenClaw hook system internals, auth profile types, skill format

## What Claude Code Knows

Claude Code maintains reference docs covering:
- Hook system (how managed hooks differ from bundled, import constraints, HOOK.md format)
- Auth profile types and all ProfileUsageStats fields
- Container filesystem layout and agent workspace mapping
- Model health system implementation details
- All changes made to the system (see CHANGELOG.md)

## When to Call Claude Code (via `coding-agent`)

**DO call Claude Code for:**
- Modifying handler.ts or any TypeScript/code file
- Debugging hook failures (it knows the hook loader internals)
- Changing openclaw.json structure (it backs up first)
- Creating new hooks or modifying existing ones
- Container-level debugging (Docker logs, process inspection)
- Anything that requires reading/writing files outside your workspace
- Pushing changes to GitHub repos

**DON'T call Claude Code for:**
- Running `/model-status` or `/model-clear` — you have those skills
- Reading model-health.json or notifications JSONL — you can cat those directly
- Routine heartbeat checks — follow the monitoring duty in your AGENTS.md
- Decisions about whether to notify Robert — that's your judgment

## How to Call Effectively

When using `coding-agent`, provide:
1. **What you need** — specific action, not vague "help me"
2. **What you've already tried** — so Claude Code doesn't repeat your work
3. **Error messages** — exact text, not summaries
4. **Which files are involved** — paths inside the container

Example good call:
```
The model-health-monitor hook stopped writing to model-health.json 10 minutes ago.
Logs show: [model-health-monitor] [ERROR] Health check failed: EACCES permission denied
File: ~/.openclaw/model-health.json
I verified the hook is still registered. Need Claude Code to fix the file permissions.
```

Example bad call:
```
Models are broken, fix it.
```

## After Claude Code Makes Changes

Claude Code will:
1. Document changes in `CHANGELOG.md`
2. Update its own memory files
3. Restart gateway if needed (with confirmation)

You should:
1. Verify the changes work (run relevant skills, check logs)
2. Push to GitHub repos if the changes touch config, skills, or workspace files
3. Update `LAST_RUN.md` with what happened
