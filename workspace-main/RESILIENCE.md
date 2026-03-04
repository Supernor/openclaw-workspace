# Resilience & Specialist Evolution Proposal

## [ROLE]
The Captain (Chief of Staff)

## [ACTION]
Standardizing the evolution of subagents into permanent specialists and clarifying operational boundaries between Local and API contexts to ensure system resilience.

## [CONTEXT]
As the OpenClaw ecosystem matures, the distinction between ephemeral task-handling and permanent specialist roles becomes critical. This proposal addresses task ambiguity, the lifecycle of specialists ("Seasoning"), and the standard workflow for complex operations.

---

## 1. Local vs. API Split (Ambiguity Prevention)

To prevent confusion and optimize token usage, agents must follow a strict "Inside-Out" heuristic:

### Local Execution (The Sanctum)
- **Scope:** File system operations, code analysis, workspace organization, data transformation, and logic derivation.
- **Preference:** ALWAYS preferred if the data exists within the workspace or can be generated via `exec`.
- **Resilience:** Ensures operations continue even if external APIs are throttled or down.

### API Execution (The Horizon)
- **Scope:** Real-time data (Web Search), external state (GitHub, Discord), and high-compute offloading (Vision, TTS).
- **Preference:** Used only when "Ground Truth" is external or specialized hardware/models are required.
- **Protocol:** API calls should be batched and the results cached locally in `memory/` to prevent redundant calls.

---

## 2. "Seasoning": Specialist Evolution Lifecycle

Temporary roles should not remain ephemeral if they solve recurring patterns. They must be "seasoned" into MD-backed specialists.

1.  **Candidate Discovery:** A subagent task that repeats >3 times with **verified successful outcomes** is flagged for seasoning. Failures are logged as antipatterns but do not contribute to seasoning.
2.  **Identity Mapping:** The subagent's prompt, tools used, and "vibe" are distilled.
3.  **MD-Backing:** Create a specialist file in `specialists/{NAME}.md` (or a dedicated Skill).
4.  **Integration:** The Captain updates `AGENTS.md` and its internal routing logic to include the new specialist in the "Manifest."
5.  **Deployment:** Future tasks of this type are routed to the specialist rather than a generic subagent.

---

## 3. Workflow: Parallelism -> Verification -> Reporting

For high-stakes or complex goals, the following pipeline is mandated:

### Phase A: Parallelism
- Break the goal into atomic sub-tasks.
- Spawn subagents for each sub-task simultaneously.
- *Constraint:* Subagents must not have write-conflicts; isolate their work-paths.

### Phase B: Verification
- A "Verifier" subagent (or the Captain) aggregates the outputs.
- Checks against the original [GOAL] and [CONTEXT] markers.
- If verification fails, the specific branch is re-run with corrective feedback.

### Phase C: Reporting
- Verified data is distilled into a "Captain's Brief."
- **Format:** RACP markers preferred.
- **Delivery:** Pushed to the Relay Agent for Human consumption.

---

## 4. Shadow Context (Failure Profiling)

To maximize token efficiency, we maintain a **Shadow Context**—local knowledge of system limitations that is NEVER sent to the target LLM but is used to optimize the task before dispatch.

### The Limits Log (`LIMITS.md`)
Each agent/specialist maintains a `LIMITS.md` profile:
- **Identifier**: [Model/Agent/Skill]
- **Weakness**: Specific task types where failures recur (e.g., "Gemini Flash: Complex regex logic").
- **Shadow Directive**: The local correction applied to the task (e.g., "If regex, break into 3 step-by-step subtasks").

### Token Efficiency Logic
By correcting the task *locally* based on the Shadow Context, we prevent the tokens spent on a failed execution and the subsequent tokens spent on a "retry" prompt. The target LLM only ever sees the "Pre-Optimized" task.
