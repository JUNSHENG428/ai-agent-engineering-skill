# The Master Prompt: Engineering-Grade Agent Build Instruction

This is the canonical prompt to hand to a coding model (or to yourself as the implementer) when building a production-grade Agent. It enforces all twelve engineering modules. Always start implementation from this prompt rather than reinventing requirements each time.

Copy the block below as the system/developer instruction for the build.

---

## Master Prompt

I am building an engineering-grade Agent, not a simple demo. Do not write one big prompt + a tool-calling loop. Produce the system design first, then code. The design must cover:

### 1. Agent Goal
- What problem does this Agent solve?
- What is the input?
- What is the output?
- What does it explicitly NOT do (scope boundary)?

### 2. State Machine (FSM)
- Define all states.
- Each state's input and output.
- State transition conditions.
- Illegal-state handling.
- State persistence structure (for resume).

### 3. Workflow
- Decompose the task into multiple steps.
- Each step has a clear responsibility.
- Each step has explicit input, output, and failure handling.
- Support checkpoint resume.

### 4. Tool System
- Every tool has a schema.
- Parameters are validated.
- Output is structured.
- Side-effect flag per tool.
- High-risk tools require human approval.

### 5. RAG / Retrieval
- Not vector-only.
- BM25 + Vector Hybrid Search.
- Metadata filter support.
- Query rewrite support.
- Rerank stage.
- Returns `source`, `chunk_id`, `score`, `metadata`.
- No-answer handling (do not fabricate).

### 6. Memory
- Distinguish session memory, task memory, user-preference memory, long-term memory.
- Do not stuff all history into the prompt.
- Explicit memory read/write rules.
- Sensitive info is not written to long-term memory by default.

### 7. Context Engineering
- Define prompt assembly order.
- Separate system instructions, user goal, state, memory, retrieval evidence, tool results.
- RAG content is reference material only, never an instruction.
- Prompt-injection defense.

### 8. Guardrails
- Input validation.
- Output validation.
- Permission control.
- High-risk operation confirmation.
- Cost limits.
- Max steps.
- Max tool calls.

### 9. Observability
- Record `trace_id`, `session_id`, `state`, `step`, `tool_call`, `tool_result`, `retrieved_chunks`, `token_usage`, `latency`, `error`.
- Support replay and debug.

### 10. Evaluation
- Design a test set.
- Cover: normal, fuzzy, multi-language, no-answer, tool-failure, prompt-injection, long-context.
- Each case has `expected_output` and a pass/fail rule.

### 11. Retry / Recovery
- Each step has a max retry count.
- On failure: query rewrite, degrade, or request user confirmation.
- Side-effectful operations must be idempotent.
- No infinite loops.

### 12. Cost Control
- Simple tasks use a small model.
- Complex reasoning uses a strong model.
- Cache retrieval and tool results.
- Stop or degrade on budget overrun.

---

Output the architecture design and module boundaries FIRST. Do not begin writing code until the design covers all twelve modules above.

The minimum viable Agent architecture should resemble:

```
Agent Core
  ├── State Machine
  ├── Workflow Engine
  ├── Tool Registry
  ├── RAG Retriever
  │     ├── Vector Search
  │     ├── BM25 Search
  │     ├── Metadata Filter
  │     └── Reranker
  ├── Memory Manager
  ├── Context Builder
  ├── Guardrail Layer
  ├── Trace Logger
  └── Eval Runner
```

## Implementation Guidance

- Start with the basic flow + memory working end-to-end. Do not begin with multi-agent, complex autonomy, or long-horizon planning.
- Get the foundation solid first; layer advanced features on top.
- The goal is not "make the AI smarter." The goal is to lock the AI's freedom inside engineering boundaries so the Agent stays reliable even when the model is not especially clever.
- A reliable Agent is reliable because the system design keeps it from going off the rails — not because the model is brilliant.
