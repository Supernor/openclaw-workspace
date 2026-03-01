# LIMITS.md

## Shadow Context Security
- **Encryption:** AES-256 for all stored agent context in `/home/node/.openclaw/workspace-spec-projects/`
- **Backup:** Daily encrypted incremental backups to external S3-compatible storage.
- **Integrity:** SHA-256 hash checks performed weekly on encrypted artifacts.

## Signal Ratio Thresholds
- **New Skill Deployment:** >85% success rate in sandbox before production.
- **Subagent Success Rate:** >90% required for complex multi-step tasks.

## Concurrency & Locking
- **File Access:** Strict single-writer, multiple-reader (SWMR) locking for `decisions/*.md`.
- **Concurrency Risk:** Parallel agents must wait for lock acquisition or retry with exponential backoff.
