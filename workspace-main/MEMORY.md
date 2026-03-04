# MEMORY.md - Long-Term Memory

## Architectural Decisions

### 2026-03-01: Human-Agent Interface & Token Efficiency
- **Definition**: 1 Human + Relay (PA).
- **Efficiency**: Separating human-readable chat from stored agent-only logs.
- **Principle**: Information should only be processed or sent if it specifically benefits the recipient (human or agent) to minimize token waste.
