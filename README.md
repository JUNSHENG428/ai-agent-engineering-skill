# Agent Engineering / AI Agent 工程化指南

> A Cursor / Claude skill for building **production-grade AI Agents** — not single-prompt demos.
>
> 面向 Cursor / Claude 的 Skill，用于构建**生产级 AI Agent**，而非「一个大 Prompt + 工具循环」的演示项目。

---

## Overview / 概述

| English | 中文 |
|---------|------|
| This repository is an **Agent Engineering skill** that guides the design and implementation of reliable, observable, and recoverable AI Agent systems. | 本仓库是一个 **Agent 工程化 Skill**，指导如何设计与实现可靠、可观测、可恢复的 AI Agent 系统。 |
| A real Agent is built around **goals, state, tools, memory, retrieval, permissions, evaluation, and recovery** — not a free-form LLM loop. | 真正的 Agent 围绕**目标、状态、工具、记忆、检索、权限、评估与恢复**构建，而不是让 LLM 自由发挥。 |
| **Core principle:** Do not let the LLM freely decide the entire flow. Reliability comes from **system design**, not model intelligence. | **核心原则：** 不要让 LLM 自由决定全流程。可靠性来自**系统设计**，而非模型智商。 |

---

## When to Use / 适用场景

| English | 中文 |
|---------|------|
| Build, design, or architect an AI Agent or agentic system | 构建、设计或架构 AI Agent / Agentic 系统 |
| Develop a **production-grade** Agent (not a toy demo) | 开发生产级 Agent（非玩具 Demo） |
| Review or improve an existing Agent implementation | 评审或改进现有 Agent 实现 |
| Add engineering rigor: state, memory, RAG, observability, guardrails | 为 Agent 补全工程化能力：状态、记忆、RAG、可观测性、护栏 |

**Do NOT use for / 不适用：**

| English | 中文 |
|---------|------|
| Simple single-shot LLM Q&A | 简单单次 LLM 问答 |
| Basic chatbots without tools | 无工具的基础聊天机器人 |
| Quick prototype / POC only | 仅需快速原型 / POC |

---

## Anti-Pattern to Avoid / 应避开的反模式

```
❌ one big system prompt
❌ + one while loop
❌ + one tools list
❌ + one vector search
❌ + let the model freely decide everything
```

| English | 中文 |
|---------|------|
| This works in a demo but breaks in production: inaccurate retrieval, state drift, wrong tool calls, no recovery, cost explosion, no logging, no evaluation. | Demo 能跑，生产必崩：检索不准、状态漂移、工具误调、无法恢复、成本失控、无日志、无评估。 |

---

## The 12 Engineering Modules / 12 大工程模块

Design all twelve modules **before writing code**. Each module has a dedicated reference file.

设计阶段须覆盖全部 12 个模块，每个模块均有独立参考文档。

| # | Module 模块 | Reference 参考文档 | Key Requirement 关键要求 |
|---|-------------|-------------------|-------------------------|
| 1 | State Machine (FSM) 状态机 | [`references/state-workflow.md`](references/state-workflow.md) | Explicit states, transitions, persistence 显式状态、转移、持久化 |
| 2 | Workflow Engine 工作流引擎 | [`references/state-workflow.md`](references/state-workflow.md) | Multi-step decomposition, resume 多步分解、断点恢复 |
| 3 | Tool System 工具体系 | [`references/tool-system.md`](references/tool-system.md) | Schema, validation, permissions, approval gates Schema、校验、权限、审批门 |
| 4 | RAG / Retrieval 检索 | [`references/rag-retrieval.md`](references/rag-retrieval.md) | Hybrid search, rerank, source boundary 混合检索、重排、来源边界 |
| 5 | Memory 记忆 | [`references/memory-context.md`](references/memory-context.md) | Session / task / preference / long-term layers 分层记忆与读写规则 |
| 6 | Context Engineering 上下文工程 | [`references/memory-context.md`](references/memory-context.md) | Ordered assembly, data-vs-instruction separation 有序组装、数据与指令分离 |
| 7 | Guardrails 护栏 | [`references/guardrails-recovery.md`](references/guardrails-recovery.md) | Input/output validation, prompt-injection defense 输入输出校验、防注入 |
| 8 | Observability 可观测性 | [`references/observability-eval.md`](references/observability-eval.md) | trace_id, tool calls, token usage, latency 全链路追踪 |
| 9 | Evaluation 评估 | [`references/observability-eval.md`](references/observability-eval.md) | Test set: normal / fuzzy / multilingual / injection 覆盖多场景的测试集 |
| 10 | Retry / Recovery 重试与恢复 | [`references/guardrails-recovery.md`](references/guardrails-recovery.md) | Per-step retry, degradation, HITL 逐步重试、降级、人工介入 |
| 11 | Cost Control 成本控制 | [`references/observability-eval.md`](references/observability-eval.md) | Step/tool/token budgets, model routing 步骤/工具/Token 预算 |
| 12 | Caching & Output Contract 缓存与输出契约 | [`references/observability-eval.md`](references/observability-eval.md) | Embedding/query/tool caches; structured JSON 缓存层与结构化输出 |

---

## 10-Question Sanity Check / 10 问生产就绪检查

Before shipping, every Agent design should answer **yes** to all ten:

上线前，以下 10 个问题均须能明确回答「是」：

| # | English | 中文 |
|---|---------|------|
| 1 | Does it have explicit state? | 是否有显式状态？ |
| 2 | Does it have an explicit workflow? | 是否有显式工作流？ |
| 3 | Do its tools have schemas and permissions? | 工具是否有 Schema 与权限？ |
| 4 | Is its RAG hybrid search (not vector-only)? | RAG 是否为混合检索（非纯向量）？ |
| 5 | Does it have a rerank stage? | 是否有重排阶段？ |
| 6 | Does it have layered memory? | 是否有分层记忆？ |
| 7 | Does it defend against prompt injection? | 是否防 Prompt 注入？ |
| 8 | Does it have trace logging? | 是否有 Trace 日志？ |
| 9 | Does it have an eval test set? | 是否有评估测试集？ |
| 10 | How does it recover from failure? | 失败时如何恢复？ |

---

## Workflow / 工作流程

| Step 步骤 | English | 中文 |
|-----------|---------|------|
| 1 | **Clarify the Agent's goal** — input, output, scope boundaries | **明确 Agent 目标** — 输入、输出、边界 |
| 2 | **Run the 10-question sanity check** | **执行 10 问检查** |
| 3 | **Design the 12 engineering modules** — load reference files as needed | **设计 12 大模块** — 按需加载参考文档 |
| 4 | **Emit the architecture design document** — use [`assets/agent-architecture-template.md`](assets/agent-architecture-template.md) | **输出架构设计文档** — 使用架构模板 |
| 5 | **Implement bottom-up** — State → Workflow → Tools → RAG → Memory → Guardrails → Observability | **自底向上实现** — 状态机 → 工作流 → 工具 → RAG → 记忆 → 护栏 → 可观测性 |

---

## Planner / Executor / Verifier / 规划·执行·验证分离

| Role 角色 | English | 中文 |
|-----------|---------|------|
| **Planner** | Decomposes the task into steps. Does NOT call tools. | 分解任务为步骤。不调用工具。 |
| **Executor** | Executes only the current step. Does NOT change the goal. | 仅执行当前步骤。不改变目标。 |
| **Verifier** | Checks acceptance criteria. On failure, retry the specific step. | 校验验收标准。失败时重试该步骤，而非从头再来。 |

> One model can play all three roles under different prompts and permissions — the point is **clear responsibility boundaries**.
>
> 可以是同一模型在不同步骤扮演不同角色 — 关键是**职责边界清晰**。

---

## Human-in-the-Loop / 人工介入（HITL）

| Risk 风险 | English | 中文 |
|-----------|---------|------|
| **Low** 低 | Read, search, summarize → auto-execute | 读取、搜索、摘要 → 自动执行 |
| **Medium** 中 | Draft generation, file creation → show diff for review | 草稿生成、文件创建 → 展示 Diff 供审核 |
| **High** 高 | Delete, send, publish, commit, payment, DB write → **MUST require explicit confirmation** | 删除、发送、发布、提交、支付、写库 → **必须经用户明确确认** |

---

## Repository Structure / 仓库结构

```
agent-engineering/
├── SKILL.md                              # Main skill entry / 主 Skill 入口
├── README.md                             # This file / 本文件
├── assets/
│   └── agent-architecture-template.md    # Architecture design template / 架构设计模板
└── references/
    ├── state-workflow.md                 # FSM, workflow, Planner/Executor/Verifier
    ├── tool-system.md                    # Tool schema, permissions, approval gates
    ├── rag-retrieval.md                  # Hybrid search, rerank, chunking
    ├── memory-context.md                 # Memory layers, context assembly
    ├── guardrails-recovery.md            # Guardrails, retry, HITL, idempotency
    ├── observability-eval.md             # Tracing, eval, cost, caching
    └── master-prompt.md                  # Canonical prompt for coding models / 交给编码模型的主 Prompt
```

---

## Installation / 安装方式

### Cursor

```bash
git clone https://github.com/JUNSHENG428/ai-agent-engineering-skill.git
cp -r ai-agent-engineering-skill ~/.cursor/skills/agent-engineering
```

Or symlink / 或使用软链接：

```bash
ln -s "$(pwd)/agent-engineering" ~/.cursor/skills/agent-engineering
```

### Claude Code / Codex

Copy the folder to your skills directory / 复制到对应 skills 目录：

```bash
# Claude Code
cp -r agent-engineering ~/.claude/skills/agent-engineering

# Codex
cp -r agent-engineering ~/.codex/skills/agent-engineering
```

---

## Quick Start / 快速开始

1. **Trigger the skill** when building or reviewing an Agent.
   **触发 Skill** — 在构建或评审 Agent 时使用。

2. **Fill in** [`assets/agent-architecture-template.md`](assets/agent-architecture-template.md) before any code.
   **先填架构模板**，再写代码。

3. **Hand off to a coding model** using [`references/master-prompt.md`](references/master-prompt.md).
   用 [`references/master-prompt.md`](references/master-prompt.md) 作为交给编码模型的标准 Prompt。

4. **Implement bottom-up** — get the basic flow working first, then add complexity.
   **自底向上实现** — 先跑通主流程，再叠加复杂度。

---

## Final Reminder / 最后提醒

| English | 中文 |
|---------|------|
| The goal is not "make the AI smarter." The goal is to **lock AI's freedom inside engineering boundaries**. | 目标不是「让 AI 更聪明」，而是**用工程边界约束 AI 的自由度**。 |
| A reliable Agent stays on the rails because the **system design** keeps it there — even when the model is not smart. | 可靠的 Agent 靠**系统设计**保轨运行 — 即使模型本身并不特别聪明。 |

---

## License / 许可证

MIT — feel free to use, modify, and distribute.
MIT — 可自由使用、修改与分发。
