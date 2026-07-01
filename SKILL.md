---
name: agent-engineering
description: "Guide for building production-grade AI Agents with engineering rigor. This skill should be used when the user wants to build, design, or architect an AI Agent (not a simple demo), develop an Agent system, create a LLM-based autonomous workflow, or review/improve an existing Agent implementation. Covers state machines, workflows, tool contracts, hybrid RAG, layered memory, context engineering, guardrails, observability, evaluation, retry/recovery, cost control, and caching. Trigger keywords: build agent, create agent, agent architecture, agent development, production agent, LLM agent, autonomous agent, multi-step agent, agentic workflow, agent system design."
agent_created: true
---

# Agent Engineering

## Overview

This skill guides the construction of engineering-grade AI Agents — not single-prompt demos with a tool-calling loop. A real Agent is an automated system built around goals, state, tools, memory, retrieval, permissions, evaluation, and recovery mechanisms. The objective is to constrain AI's freedom within engineering boundaries so that the Agent stays reliable even when the model is not especially clever.

Core principle: **do not let the LLM freely decide the entire flow.** Constrain it with explicit state machines, workflows, tool contracts, layered memory, guardrails, and recovery strategies. Reliability comes from system design, not model intelligence.

## When to Use

Trigger this skill when the user asks to:
- Build, design, or architect an AI Agent or agentic system
- Develop a production-grade Agent (not a toy demo)
- Review or improve an existing Agent implementation
- Create an autonomous LLM workflow with tool calling
- Add engineering rigor (state, memory, RAG, observability) to an Agent

Do NOT trigger for simple single-shot LLM Q&A, basic chatbots without tools, or when the user only wants a quick prototype/POC.

## The Anti-Pattern to Avoid

A demo Agent typically looks like this — avoid producing this:

```
one big system prompt
+ one while loop
+ one tools list
+ one vector search
+ let the model freely decide everything
```

This runs in a demo but breaks in production: retrieval is inaccurate, state drifts, tools get called wrongly, failures don't recover, costs explode, nothing is logged, nothing can be evaluated.

## Workflow: Design First, Code Second

Never jump straight into writing code. Follow this sequence:

### 1. Clarify the Agent's Goal

Before any design, pin down:
- What problem does this Agent solve?
- What is the input?
- What is the output?
- What does it explicitly NOT do? (scope boundaries matter)

### 2. Run the 10-Question Sanity Check

Evaluate any Agent scheme against these questions. If any answer is "no" or unclear, the scheme is not yet production-ready:

1. Does it have explicit state?
2. Does it have an explicit workflow?
3. Do its tools have schemas and permissions?
4. Is its RAG hybrid search (not vector-only)?
5. Does it have a rerank stage?
6. Does it have layered memory?
7. Does it defend against prompt injection?
8. Does it have trace logging?
9. Does it have an eval test set?
10. How does it recover from failure?

### 3. Design the 12 Engineering Modules

Produce a system design covering all twelve modules below BEFORE writing code. Each module has a dedicated reference file with detailed requirements, examples, and the exact prompt phrasing to hand to a coding model. Load the relevant reference when designing that module.

| # | Module | Reference | Key Requirement |
|---|--------|-----------|-----------------|
| 1 | State Machine (FSM) | `references/state-workflow.md` | Explicit states, transitions, illegal-state handling, persistence |
| 2 | Workflow Engine | `references/state-workflow.md` | Multi-step decomposition, per-step I/O, failure handling, resume |
| 3 | Tool System | `references/tool-system.md` | Schema + validation + permissions + side-effect flags + approval gates |
| 4 | RAG / Retrieval | `references/rag-retrieval.md` | Hybrid (BM25 + vector), metadata filter, query rewrite, rerank, source-boundary |
| 5 | Memory | `references/memory-context.md` | Session / task / preference / long-term layers, read-write rules |
| 6 | Context Engineering | `references/memory-context.md` | Ordered assembly, source tagging, data-vs-instruction separation |
| 7 | Guardrails | `references/guardrails-recovery.md` | Input/output validation, cost limits, prompt-injection defense |
| 8 | Observability | `references/observability-eval.md` | trace_id, state, tool calls, chunks, token usage, latency, errors |
| 9 | Evaluation | `references/observability-eval.md` | Test set covering normal/fuzzy/multilingual/no-answer/injection/long-context |
| 10 | Retry / Recovery | `references/guardrails-recovery.md` | Per-step retry budget, query rewrite, degradation, idempotency, HITL |
| 11 | Cost Control | `references/observability-eval.md` | Step/tool/token budgets, model routing, caching, degradation |
| 12 | Caching & Output Contract | `references/observability-eval.md` | Embedding/query/tool caches; structured JSON output schema |

### 4. Emit the Architecture Design Document

Output the architecture design and module boundaries FIRST. Do not begin coding until the design covers all twelve modules. Use the template in `assets/agent-architecture-template.md` as a starting structure.

### 5. Implement Bottom-Up

Build the minimum viable Agent architecture, then layer features on top:

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

Get the basic flow + memory working first. Do not start with multi-agent, complex autonomy, or long-horizon planning. Those only make sense once the foundation is solid.

## Planner / Executor / Verifier Separation

Separate planning from execution. This does NOT require three separate models — it can be one model acting under different prompts, permissions, and output contracts at different steps. The point is clear responsibility boundaries:

- **Planner**: decomposes the task into steps. Does NOT call tools.
- **Executor**: executes only the current step. Does NOT change the goal.
- **Verifier**: checks whether the result meets acceptance criteria. On failure, retry the specific step — never restart from scratch.

See `references/state-workflow.md` for the full Planner/Executor/Verifier contract.

## Human-in-the-Loop

Classify every operation by risk and enforce approval gates:

- **Low risk** (read, search, summarize): auto-execute.
- **Medium risk** (draft generation, file creation): show a diff for review.
- **High risk** (delete, send, publish, git commit, payment, production API, DB write): MUST require explicit user confirmation. The Agent never executes these directly.

See `references/guardrails-recovery.md`.

## The Master Prompt

When handing the design to a coding model (or to yourself as the implementer), use the master prompt in `references/master-prompt.md`. It is the single canonical instruction set that enforces all twelve modules. The master prompt opens with:

> "I am building an engineering-grade Agent, not a simple demo. Do not write one big prompt + tool-calling loop. Produce the system design first, then code."

Always start implementation from that master prompt rather than reinventing the requirements each time.

## Resources

### references/
Load these as needed while designing each module:

- `references/state-workflow.md` — FSM design, workflow decomposition, Planner/Executor/Verifier, tool routing rules.
- `references/rag-retrieval.md` — Hybrid search, chunking strategies, metadata, reranker, query rewrite, no-answer handling.
- `references/tool-system.md` — Tool schema contract, parameter validation, permissions, side-effects, approval gates.
- `references/memory-context.md` — Memory layers, read/write rules, context assembly order, prompt-injection boundaries.
- `references/guardrails-recovery.md` — Guardrails, HITL risk tiers, retry/recovery, idempotency, sandbox isolation.
- `references/observability-eval.md` — Trace logging schema, eval test-set design, cost budgets, caching layers, output contract.
- `references/master-prompt.md` — The canonical prompt to hand to a coding model for building an engineering-grade Agent.

### assets/
- `assets/agent-architecture-template.md` — Fill-in template for the Agent architecture design document. Copy it as the starting point for any new Agent design.

## Final Reminder

The goal of building an Agent is not "make the AI smarter." The goal is to lock AI's freedom inside engineering boundaries. A reliable Agent is reliable because the system design keeps it from going off the rails even when the model is not smart. Always design the constraints first.
