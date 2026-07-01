# Agent Architecture Design Document

> Fill in this template before writing any code. Copy it as the starting point for every new Agent design. Delete the guidance notes (italic blockquotes) as you complete each section.

---

## 1. Agent Goal

**Problem statement:**
_What problem does this Agent solve?_

**Input:**
_What goes in? (format, source, constraints)_

**Output:**
_What comes out? (format, delivery, constraints)_

**Out of scope (explicitly does NOT do):**
_What boundaries does this Agent refuse to cross?_

---

## 2. State Machine (FSM)

**States:**

| State | Purpose | Input | Output | Allowed next states | Human confirmation? |
|-------|---------|-------|--------|---------------------|---------------------|
| | | | | | |

**Transition conditions:**
_What triggers each transition?_

**Illegal-state handling:**
_What happens on an invalid transition?_

**Persisted state fields:**
_What is saved so a run can resume?_

---

## 3. Workflow

**Steps:**

| Step | Responsibility | Input | Output | Failure handling | Retryable? | Artifact persisted? |
|------|----------------|-------|--------|-------------------|------------|---------------------|
| | | | | | | |

**Checkpoint resume strategy:**
_How does a run pick up after interruption?_

---

## 4. Tool System

**Tool registry:**

| Tool | Description | Input schema | Output schema | Permission level | Side effects? | Requires confirmation? |
|------|-------------|--------------|---------------|------------------|---------------|------------------------|
| | | | | | | |

**Tool routing rules:**
1. When must it query RAG?
2. When must it search the web?
3. When can it answer directly?
4. When must it call a code tool?
5. When does it need user confirmation?
6. When is it forbidden to call any tool?

---

## 5. RAG / Retrieval

**Hybrid search:**
- Vector search config: _
- BM25 config: _
- Merge / dedupe / normalize strategy: _

**Chunking strategy:**
_Type (fixed / recursive / semantic / code / parent-child): _
_Parent-child link field: _

**Metadata fields per chunk:**
`source`, `title`, `section`, `tags`, `language`, `created_at`, `updated_at`, `doc_type`, `project`, `is_archived`, _

**Reranker:**
_Model / method: _ | _Recall top_k: _ | _Final top_k: _

**Query rewrite:**
_Number of rewritten queries: _ | _Rewrite strategy: _

**No-answer threshold:**
_Score below which the Agent returns "no evidence": _

---

## 6. Memory

**Layers:**

| Layer | Scope | Read rule | Write rule | TTL |
|-------|-------|-----------|------------|-----|
| Session | | | | |
| User preference | | | | |
| Task state | | | | |
| Long-term knowledge | | | | |

**Forbidden from long-term memory:**
_Secrets, PII, transient state, ..._

---

## 7. Context Engineering

**Assembly order:**
1. Current task goal
2. State
3. Relevant memory
4. Retrieval evidence
5. Tool results
6. Output format

**Source boundary rules:**
- system / developer / user = instructions
- retrieved documents = untrusted data
- tool results = constrained evidence
- document-embedded "instructions" = never escalated to policy

**Compression strategy:**
_How is context compressed near the token limit?_

---

## 8. Guardrails

- Input validation: _
- Output validation (JSON Schema per step): _
- Permission control: _
- High-risk confirmation gate: _
- Cost limits (max steps / max tool calls / max tokens): _
- Rate limits: _

---

## 9. Observability

**Trace fields recorded:**
`trace_id`, `session_id`, `user_input`, `current_state`, `retrieved_chunks`, `tool_calls`, `tool_results`, `model_input`, `model_output`, `token_usage`, `latency`, `error`, `retry_count`, `final_answer`

**Storage:** _Where are traces persisted?_
**Replay support:** _How is a run replayed for debug?_

---

## 10. Evaluation

**Test set coverage:**

| Category | Example case | Expected answer | Expected sources | Pass/fail rule |
|----------|--------------|-----------------|------------------|----------------|
| Normal | | | | |
| Ambiguous | | | | |
| Multi-language | | | | |
| Proper noun | | | | |
| Error message | | | | |
| No answer | | | | |
| Prompt injection | | | | |
| Tool failure | | | | |
| Long context | | | | |
| Cost pressure | | | | |

---

## 11. Retry / Recovery

| Step | Max retries | Retry changes params? | Degrade on failure? | Human intervention? | Persist state? | Resume allowed? |
|------|-------------|-----------------------|---------------------|---------------------|----------------|-----------------|
| | | | | | | |

**Idempotency:** _Which tools require `idempotency_key`?_
**High-risk failure default:** _Human confirmation (never auto-retry)._

---

## 12. Cost Control

- Max steps per task: _
- Max tool calls per task: _
- Max token budget per task: _
- Degradation strategy on budget overrun: _
- Model routing (simple → small model, complex → strong model): _
- Caching: embedding cache, query cache, tool-result cache, web cache, invalidation policy: _

---

## Architecture Diagram

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

## Open Questions / Decisions Pending

_Use this section to record unresolved design questions before implementation begins._
