# State Machine, Workflow & Planner/Executor/Verifier

## State Machine (FSM)

State transitions must NOT be left to the LLM's discretion. Free-form "the model decides what to do next" produces chaos: it may research, then write, then rename the title, then loop back — with no guarantee of progress.

Use a Finite State Machine (or an equivalent explicit state manager: LangGraph stateful graph, a workflow engine, etc.) to define:

- What is the current state?
- Which next states are allowed?
- What condition triggers a transition?
- How are illegal states handled?

### Example FSM (Article-Writing Agent)

```
IDLE
  ↓
COLLECT_REQUIREMENTS
  ↓
RESEARCH
  ↓
OUTLINE
  ↓
DRAFT
  ↓
REVIEW
  ↓
REVISE ←──┐
  ↓        │
FINAL      │ (revise loops back to REVIEW)
  ↓
IDLE
```

| State | Purpose | Allowed next states |
|-------|---------|---------------------|
| IDLE | Wait for user input | COLLECT_REQUIREMENTS |
| COLLECT_REQUIREMENTS | Extract requirements | RESEARCH / OUTLINE |
| RESEARCH | Search materials | OUTLINE |
| OUTLINE | Generate outline | DRAFT |
| DRAFT | Write first draft | REVIEW |
| REVIEW | Self-check issues | REVISE / FINAL |
| REVISE | Apply fixes | REVIEW / FINAL |
| FINAL | Output result | IDLE |

### FSM Design Requirements

Define for every Agent:

1. All states.
2. Each state's input.
3. Each state's output.
4. State transition conditions.
5. Illegal-transition handling.
6. Whether human confirmation is required before a transition.
7. The persisted state fields (so a run can resume after interruption).

### Prompt phrasing for the implementer

> This Agent must manage state with an FSM (or equivalent explicit state manager). Do not let the LLM freely decide the flow. Define: all states; each state's input; each state's output; transition conditions; illegal-transition handling; whether human confirmation is needed; and the persisted state fields.

## Workflow: Complex Tasks Must Not Rely on a Single Prompt

FSM handles state flow; Workflow handles process orchestration. A workflow decomposes a task into fixed deterministic steps.

### Example: Analyze a GitHub Project

Do NOT prompt "analyze this project." Decompose:

```
Read README
  ↓
Identify project goals
  ↓
Read package/build config
  ↓
Analyze directory structure
  ↓
Find core entry points
  ↓
Check recent commits
  ↓
Check issues / PRs
  ↓
Output conclusions
```

### When to Use a Fixed Workflow

Use a fixed workflow for any of:
- Document analysis
- Code review
- Article generation
- Data processing
- Competitive research
- Multi-step tool calling
- Automated testing
- CI fix pipelines

### Workflow Step Requirements

Each step must have:
- Explicit input.
- Explicit output.
- Failure handling.
- Whether it is retryable.
- Its artifact persisted (for checkpoint resume and replay).

### Prompt phrasing

> Do not write one big prompt. Design a workflow where each step has explicit input, output, failure handling, and a retry flag. Each step's artifact must be persisted for checkpoint resume and replay.

## Planner / Executor / Verifier

Separate planning from execution. This does not require three separate models — it can be one model acting under different prompts, permissions, and output contracts at different steps. The point is clear responsibility boundaries.

### Planner
- Decomposes the task into an ordered step list.
- Does NOT call tools.
- Output: a plan (steps with acceptance criteria).

### Executor
- Executes only the current step.
- Does NOT change the goal or the plan.
- Output: a structured result for that step.

### Verifier
- Checks whether the result meets the step's acceptance criteria.
- On failure: retry the specific step (never restart from scratch).
- Checks for: treated marketing claims as fact? cited code evidence? missed limitations?

### Example

User: "Analyze whether librepods really lets Android fully support AirPods."

Planner output:
1. Read README capability list.
2. Inspect protocol implementation.
3. Inspect Android-side Bluetooth API usage.
4. Check issue tracker for compatibility feedback.
5. Compare against Apple native capability gaps.
6. Output conclusions.

Executor only executes: "Read README and extract the feature list."

Verifier checks: "Did it cite code evidence? Did it confuse marketing claims with facts? Did it miss limitations?"

### Prompt phrasing

> Adopt a Planner / Executor / Verifier architecture:
> - Planner only decomposes the task; it does not call tools.
> - Executor only executes the current step; it does not change the goal.
> - Verifier checks whether the result meets acceptance criteria.
> - On failure, retry the specific step, not from the beginning.

## Tool Routing

Do not dump all tools into the model's context and let it pick freely. That produces: unnecessary searches, answering from memory when retrieval is required, guessing when a code tool is needed, and executing when confirmation is required.

Design a Router with explicit rules:

1. When must it query RAG?
2. When must it search the web?
3. When can it answer directly?
4. When must it call a code tool?
5. When does it need user confirmation?
6. When is it forbidden to call any tool?

### Prompt phrasing

> Design tool routing rules. Do not rely entirely on the LLM picking tools freely. Specify: when RAG is mandatory; when web search is mandatory; when direct answer is allowed; when a code tool is mandatory; when user confirmation is required; when tool calls are forbidden.
