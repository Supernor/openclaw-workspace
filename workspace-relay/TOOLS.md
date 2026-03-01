# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

Add whatever helps you do your job. This is your cheat sheet.

## SSH Tunnel (Dashboard Access)

Robert connects to the dashboard via SSH tunnel from his local machine:
```bash
ssh -N -L 18789:127.0.0.1:18789 root@187.77.193.174
```
Then open: `http://localhost:18789/chat?session=main`

**If dashboard shows "disconnected (1006)"** — the SSH tunnel probably died. Tell Robert to re-run the command above in a terminal.

Robert is on T-Mobile — his IP changes frequently. Do not use IP-based restrictions.
