# Tool System: Schema, Permissions & Approval Gates

Tools are not function names — they are contracts. Every tool the Agent can call must have a clear description, parameter constraints, output format, error format, permission level, side-effect flag, retry policy, and approval requirement.

## Bad vs. Good Tool Design

### Bad

```
search(query)
```

### Good

```json
{
  "name": "search_documents",
  "description": "Search indexed documents using hybrid retrieval",
  "input": {
    "query": "string",
    "filters": {
      "project": "string",
      "doc_type": "string",
      "date_range": "string"
    },
    "top_k": "number"
  },
  "output": {
    "chunks": [
      {
        "text": "string",
        "source": "string",
        "score": "number",
        "metadata": "object"
      }
    ]
  }
}
```

## Required Fields per Tool

Every tool definition must specify:

1. **What the tool does** — clear description.
2. **What the tool does NOT do** — scope boundary.
3. **Input parameters** — typed schema, with validation rules.
4. **Output format** — structured schema, not free text.
5. **Error format** — error codes + structured error body.
6. **Permission level** — low / medium / high risk.
7. **Whether it is retryable.**
8. **Whether it has side effects.**
9. **Whether it requires user confirmation.**

## Parameter Validation

Validate every input against the schema before execution. Reject early on type mismatch, missing required fields, or out-of-range values. Never pass unchecked arguments to a tool that touches external systems.

## Side-Effect Flagging

Classify each tool:

- **Read-only / side-effect-free**: search, read file, summarize. Safe to retry.
- **Side-effectful**: send email, write file, commit git, call production API. Must check `idempotency_key` before executing (see `guardrails-recovery.md`). Retries must not duplicate the effect.
- **Destructive**: delete file, drop table, force-push. High-risk — requires explicit user confirmation every time.

## Approval Gates (Human-in-the-Loop)

Risk tiers:

| Risk | Examples | Policy |
|------|----------|--------|
| Low | read, search, summarize | Auto-execute |
| Medium | generate draft, create file | Show diff for review |
| High | delete, send, publish, git commit, payment, production API, DB write | MUST require explicit user confirmation |

The Agent must never execute a high-risk action directly. It produces the intended action payload, presents it to the user, and waits for confirmation.

### Prompt phrasing

> Design each tool with schema, description, parameter validation, error codes, permission level, and side-effect description. High-risk tools must enter a human-approval gate — the model must not execute them directly.

## Tool Registry

Maintain a tool registry that the Router and Executor consult. Each entry includes the schema, permission level, side-effect flag, retry policy, and a reference to its implementation. The registry is the single source of truth — if a tool is not in the registry, the Agent cannot call it.

## Output Contract for Tools

Tool results must be structured JSON, not free text. Downstream logic must never parse tool output with regex. Example:

```json
{
  "tool": "search_documents",
  "status": "success",
  "result": { "chunks": [] },
  "error": null,
  "metadata": { "latency_ms": 120, "retrieved_count": 8 }
}
```

If the LLM that produced tool arguments emitted malformed JSON, attempt an auto-repair or retry — do not propagate free text into the workflow.
