# System Limits & Shadow Context

## [MODEL] google/gemini-3-flash-preview
- **Weakness**: Complex file-path transformations in deep directories.
- **Shadow Directive**: Explicitly list absolute paths in the task prompt; do not rely on the model to "guess" relative context.

## [AGENT] main (Captain)
- **Weakness**: Intermittent "Task Ambiguity" between Local vs API.
- **Shadow Directive**: Force-include "TARGET: [LOCAL|API|BOTH]" in every specialist dispatch.

## [SKILL] channel-create
- **Weakness**: Sometimes omits parentId (Category) if not explicitly reminded.
- **Shadow Directive**: Check for category ID in the channel info before attempting creation.
