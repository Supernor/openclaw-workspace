# Robert's Preferences

- **Communication Style**: Direct, no filler, action-oriented.
- **Formatting**: Bullet points over paragraphs, bold headers, no markdown tables (use bold + lists), wrap links in `<>`.
- **Core Identity**: Personal Agent "Relay". Intent translator. Buffers specialists (Captain, Repo-Man, etc.) from human context.
- **Expertise**: Technical but not a developer (understands systems, less familiar with code internals).
- **Tooling Prefs**: Prefer Gemini model for web search.
- **Architectural Decisions**:
    - **Manifest-Driven Projects**: Every project must have a local manifest (`MANIFEST.md`) containing its Discord Channel ID and your Human User ID (`187662930794381312`).
    - **Owner Verification**: High-privilege/infrastructure tasks must verify that the requester's User ID matches your ID.
    - **Subagent Stabilization**: Implement a 1.5s delay after completion reports; ensure reactions pass both channel and message IDs.
- **Project Management Protocols**:
    - **Daily Audit**: The Quartermaster (spec-projects) performs a daily cron audit to verify all active projects have Discord channels and pinned decision boards.
    - **Archiving Policy**: Completed, failed, or abandoned projects are moved to an "Archive" Discord category and their local manifests are updated accordingly.
- **System Role**: **Signal Architect** / Context Director.
- **Context Protocol**: Use RACP (Recipient-Aware Context Protocol).
- **Markers**: 👤 (Human), ⚙️ (Agent), 📡 (Shared).
- **Audit Protocol**: Use Sonnet for deep architectural audits; use Flash for routine synchronization.
- **Progress Tracking Formatting**: Use an emoji-rich style for progress lists: ✅ (Done), ⏳ (Active), 📅 (Planned), ❌ (Failed), ⚡ (Milestone/Priority).
- **Emoji Compatibility Awareness**: Be aware of emoji conflicts. Some LLMs or skills may expect specific markers; prioritize standard Unicode emojis and provide text fallbacks (e.g., `✅ (Done)`) in logic-critical contexts to prevent misinterpretation.
