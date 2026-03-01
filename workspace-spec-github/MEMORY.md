# MEMORY.md — spec-github (Repo-Man)

> Auto-written by Repo-Man. Do not edit manually.

## System State

| Item | Value | Last Verified |
|------|-------|--------------|
| gh CLI auth | Supernor | 2026-03-01 |
| Key count | 7 canonical keys | 2026-03-01 |
| Last key rotation | All 7 keys rotated | 2026-03-01 |
| SAG/ElevenLabs key | Excluded intentionally — skill not active | 2026-03-01 |
| openclaw-config repo | Created, private | 2026-03-01 |
| openclaw-workspace repo | Created, private | 2026-03-01 |
| openclaw-skills repo | Created, private | 2026-03-01 |

## Last Run Per Skill

| Skill | Last Run | Result |
|-------|----------|--------|
| key-drift-check | never | — |
| workspace-backup | never | — |
| env-backup | never | — |
| repo-health | never | — |
| rotate-key | 2026-03-01 (all 7) | PASS |
| session-wrap | never | — |

## Known Issues / Suppressions

- SAG key (ElevenLabs sk_be2cb...) intentionally not in canonical list. Add when skill is activated.
- Codex OAuth token left hardcoded by design — not managed by Repo-Man.

## Decisions Log Pointer

Full decision audit trail: `openclaw-config/logs/DECISIONS.md` on GitHub.
