# Observability, Evaluation, Cost Control, Caching & Output Contract

## Observability: Tracing Is the Lifeline

Without trace logs, an Agent failure is un-debuggable — the chain is too long to reconstruct from the final answer. On every run, record at minimum:

### Required Trace Fields

| Field | Purpose |
|-------|---------|
| `trace_id` | Correlate all steps of one run |
| `session_id` | Correlate across turns in a session |
| `user_input` | The original request |
| `current_state` | FSM state at each step |
| `retrieved_chunks` | What RAG returned (with scores) |
| `tool_calls` | Which tools, with what arguments |
| `tool_results` | What each tool returned |
| `model_input` | The assembled prompt sent to the model |
| `model_output` | The raw model response |
| `token_usage` | Prompt + completion tokens per call |
| `latency` | Per-step and end-to-end timing |
| `error` | Any error encountered |
| `retry_count` | How many retries occurred |
| `final_answer` | The answer delivered to the user, with cited sources |

### Prompt phrasing

> Design a trace mechanism for the Agent. Every run must record: user input; state transitions; retrieval queries; hit chunks; tool-call arguments; tool results; LLM output; tokens and latency; errors and retries; and the final answer's cited sources. Support replay and debug.

## Evaluation

Only a stable eval test set keeps Agent iteration from being superstition. Prepare a test set covering:

1. Normal questions.
2. Ambiguous / fuzzy questions.
3. Multi-language questions.
4. Proper-noun / code-symbol questions.
5. Error-message questions.
6. No-answer questions.
7. Prompt-injection questions.
8. Tool-failure questions.
9. Long-context questions.
10. Cost-pressure questions.

Each test case must have `expected_answer`, `expected_sources`, and a pass/fail rule.

### Example RAG Eval Case

```json
{
  "question": "Android 17 的 PMGD 通过什么配置文件控制？",
  "expected_keywords": ["/vendor/etc/pmgd/config.json"],
  "expected_sources": ["android_17_memory.md"],
  "should_not_include": ["ActivityManager 传统 OOM"]
}
```

### Eval Dimensions

- Did it complete the task?
- Did it call the correct tool?
- Did it retrieve the correct material?
- Did it cite the correct source?
- Did it respect permissions?
- Did it hallucinate?
- Can it handle failure?
- Is cost controlled?
- Is latency acceptable?

### Prompt phrasing

> Do not implement features only. Design an eval dataset simultaneously: normal, fuzzy, multi-language, proper-noun, error-message, no-answer, prompt-injection, tool-failure, long-context, and cost-pressure cases. Each case has `expected_answer`, `expected_source`, and a pass/fail rule.

## Cost Control

Agents burn tokens at every step: planning, retrieval, tool returns, multi-turn observation, reflection, retry, final answer. Complex agents are expensive.

### Control Levers

- Max steps per task.
- Max tool calls per task.
- Max retrieval chunks per query.
- Compress historical context.
- Route simple tasks to a small model; reserve the strong model for complex reasoning.
- Cache retrieval results.
- Cache tool results.
- Do not retry infinitely on failure.

### Prompt phrasing

> Add a cost budget: max steps per task; max tool calls; max token budget; degradation on budget overrun; small model for simple tasks; strong model only for complex reasoning.

## Caching

Within a session, many things should not be recomputed. Caching cuts cost and latency.

### What to Cache

- Document parsing results.
- Embeddings.
- Retrieval results (per query).
- Web-scrape results.
- Tool-call results (keyed by arguments + idempotency).
- Rerank results.
- Model classification results (small-model routing decisions).

### Cache Design Requirements

1. Embedding cache.
2. Query retrieval cache.
3. Tool-result cache.
4. Web-content cache.
5. Cache invalidation policy (TTL, source-change detection).
6. Cache-key design (deterministic, argument-aware).
7. Never cache sensitive data.

### Prompt phrasing

> Design a cache layer: embedding cache; query retrieval cache; tool-result cache; web-content cache; invalidation policy; cache-key design; and a rule that sensitive data is never cached.

## Output Contract

Output must be structured. The Agent's internal step results must be JSON-conforming to a schema — not free text. Free-text parsing downstream is a maintainability disaster.

### Example Output Contract

```json
{
  "status": "success",
  "summary": "Analysis complete",
  "findings": [],
  "sources": [],
  "next_actions": [],
  "requires_user_confirmation": false
}
```

### Prompt phrasing

> Define an output contract for every Agent step. Model output must conform to a JSON Schema. If it does not, auto-repair or retry. Downstream logic must not depend on free-text parsing.
