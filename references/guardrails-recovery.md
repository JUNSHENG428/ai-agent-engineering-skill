# Guardrails, Human-in-the-Loop, Retry/Recovery, Idempotency & Sandbox

## Guardrails

Guardrails are the constraint system that stops the Agent from going off the rails. They include:

- Input validation.
- Output validation.
- Tool permissions.
- Sensitive-operation confirmation.
- Content safety.
- Cost limits.
- Rate limits.
- Privilege escalation defense.
- Prompt-injection defense (see `memory-context.md`).

### Input Validation

Reject malformed, oversized, or policy-violating input before it enters the workflow. Examples: empty query, query exceeding max length, query containing embedded tool-call syntax from an untrusted source.

### Output Validation

Validate every step's output against its JSON Schema contract. If the model returns free text where structured JSON is required, attempt auto-repair; if repair fails, retry the step up to its retry budget.

### Cost & Rate Limits

- Max steps per task.
- Max tool calls per task.
- Max tokens per task.
- Max tokens per step.
- Requests-per-minute cap on external APIs.

When a budget is exceeded, degrade (switch to a smaller model, reduce retrieval top_k, compress context) rather than silently continuing.

## Human-in-the-Loop (HITL)

Not everything should auto-execute. Classify operations by risk:

| Risk | Examples | Policy |
|------|----------|--------|
| Low | read, search, summarize | Auto-execute |
| Medium | generate draft, create file | Show diff for review |
| High | delete, send, publish, git commit, payment, production API, DB write, notification | MUST require explicit user confirmation |

The Agent must never directly execute a high-risk action. It produces the action payload, presents it, and waits.

### Prompt phrasing

> Classify operations into low / medium / high risk:
> - Low: read, search, summarize — auto-execute.
> - Medium: generate draft, create file — show diff.
> - High: delete, send, publish, commit, payment — require user confirmation.
>
> The Agent must not execute high-risk actions directly.

## Retry / Recovery

"Failure recovery matters more than the success path." Agents fail constantly: tool timeout, no retrieval result, JSON parse error, model output format error, API rate limit, network error, context-too-long, result-not-as-expected. Without recovery, the Agent short-circuits.

### Common Failure → Recovery Map

| Failure | Recovery |
|---------|----------|
| No retrieval result | Query rewrite, then re-search |
| Tool timeout | Retry, reduce top_k |
| JSON format error | Ask the model to repair the JSON |
| Context too long | Compress context |
| Code execution failed | Read error log, then fix |
| Conflicting information | Annotate conflict, do not merge |
| Insufficient permission | Request user authorization |

### Per-Step Recovery Design

For every workflow step, define:
1. Max retry count.
2. Whether retry changes parameters (e.g., rewrites the query).
3. Whether to degrade on failure.
4. Whether human intervention is needed.
5. Whether intermediate state is persisted.
6. Whether checkpoint resume is allowed.

### Prompt phrasing

> Design a failure-recovery strategy for every workflow step: max retries; whether retry changes parameters; whether to degrade; whether human intervention is needed; whether intermediate state is persisted; whether checkpoint resume is allowed.

### Retry Safety Rules

- Read-only tools may auto-retry.
- Side-effectful tools must check `idempotency_key` and prior execution status before retrying.
- High-risk tools, on failure, default to human confirmation — never auto-retry.

## Idempotency

Same operation executed once or many times must yield the same result, with no duplicated side effects. A retry that sends two emails is a disaster.

### Design

```
send_email(action_id="email_20260626_001")
```

If `action_id` has already been executed, do not re-send.

### Prompt phrasing

> All side-effectful tools must support `idempotency_key`. Retries must not duplicate send/delete/submit. Before execution, check whether the `action_id` has already completed.

## Sandbox

Where possible, run the Agent inside a sandbox to limit its execution scope: run code, modify files, analyze projects, execute shell, install deps, run tests, process user-uploaded files — all isolated from the real environment.

### Rules

1. Sandbox may create, modify, delete temporary files freely.
2. Real-project modifications must be produced as a diff.
3. Diffs are applied only after user confirmation.
4. Sandbox execution has resource limits (CPU, memory, time, disk).
5. On task end, destroy or archive the sandbox.

### Prompt phrasing

> Code execution and file operations must happen in a sandbox. Separate real-world operations from sandbox operations:
> 1. Sandbox may create/modify/delete temp files.
> 2. Real-project modifications must produce a diff.
> 3. User confirmation required before applying.
> 4. Sandbox execution has resource limits.
> 5. Destroy or archive the sandbox on task end.
