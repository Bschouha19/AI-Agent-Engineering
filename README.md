# AI Agent Engineering

### Master Autonomous Agent Systems — From Fundamentals to Production — Volume 4

A complete, practical Agent Engineering course for software developers who have completed Volumes 1–3. Not a survey of frameworks — a handbook that produces engineers capable of designing, building, evaluating, securing, and operating production autonomous agent systems, from a single tool-using agent to a coordinated multi-agent fleet.

---

## Prerequisites

This is Volume 4 of the AI Engineering Handbook series. Before starting this volume, complete:

**[Volume 1 — AI Engineering](https://github.com/Bschouha19/AI-Engineering-Handbook)** — especially Chapter 07 (Embeddings), Chapter 09 (RAG), and Chapters 10–11 (Agents and Multi-Agent Systems), which this volume goes far beyond.

**[Volume 2 — MCP Engineering](https://github.com/Bschouha19/MCP-Engineering)** — tool-serving is the other half of the tool-use problem this volume completes from the agent's own side.

**[Volume 3 — RAG Deep Dive](https://github.com/Bschouha19/RAG-Deep-Dive)** — especially Chapter 06 (Dense Retrieval), Chapter 12 (RAG Evaluation), Chapter 13 (Trustworthy RAG), and Chapter 14 (Production RAG Operations), all directly extended in this volume.

Volume 4 assumes familiarity with: Python, basic agent concepts (the ReAct loop, tool calling), MCP server/client basics, RAG fundamentals, and Docker.

---

## What This Is

An autonomous agent is more than an LLM call with a tool bolted on — it reasons across multiple steps, decides when to call which tool, remembers context across sessions, and sometimes coordinates with other agents to get a job done. Volume 1 taught the basic agent loop. This volume teaches everything between "a working agent demo" and "a production multi-agent system you'd trust with genuine autonomy" — reasoning and planning patterns, persistent memory, multi-agent orchestration, agent-to-agent protocols, human oversight calibrated to risk, trajectory-level evaluation, and the security discipline unique to systems that can actually *act*, not just respond.

**By the end of this volume you will be able to:**

- Design an agent's reasoning loop deliberately (ReAct, plan-and-execute, reflection) instead of defaulting to whichever pattern a tutorial happened to use
- Build tool use that recovers from errors and validates results, not just the happy path
- Give an agent persistent, useful memory across sessions without letting it grow unbounded
- Orchestrate multiple agents safely, using the protocols purpose-built for agent-to-agent communication
- Design human-in-the-loop and human-on-the-loop oversight calibrated to how consequential an agent's actions actually are
- Evaluate an agent by its trajectory, not just its final answer — catching a costly, roundabout, or unsafe path to a correct result
- Bound an agent's autonomy deliberately — least-privilege capability scoping, sandboxing, and defense against excessive agency
- Operate a fleet of agents in production — cost governance, rate limiting, and debugging failures that span multiple agents

---

## Who It's For

| Reader | Background | What They Get |
|--------|-----------|--------------|
| **AI Engineer** | Completed Volumes 1–3 | Full agent engineering depth — reasoning, orchestration, evaluation, and trustworthy production deployment |
| **Backend Developer** | Has built a single tool-calling agent before | The gap between a single agent demo and a production multi-agent system, closed systematically |
| **Engineer building genuinely autonomous systems** | Real write access, real financial/communication actions, real browser automation | A course that teaches the safeguards these capabilities specifically require, not just the happy-path demo |

---

## The Framework Thread

This course teaches every core concept framework-agnostically first, the same way Volume 3 taught retrieval theory before naming a vector database — the current agent framework landscape (LangGraph, the Claude Agent SDK, OpenAI's Agents SDK, CrewAI, Microsoft's Agent Framework, Google's ADK) is genuinely fragmented, and none has an obvious-default status yet. This course uses **LangGraph** for multi-agent orchestration and the **Claude Agent SDK** for single-agent/subagent patterns as its two hands-on production frameworks; every other framework is covered accurately in comparison tables.

## The Autonomy Thread

This course escalates stakes by **autonomy and blast radius**, not document domain. Modules 1–2 use low-stakes, reversible scenarios (reusing Volume 3's fictional "Aperture Cloud" company where it fits). Modules 3–4 introduce agents with genuinely consequential capabilities, paired explicitly with the safeguards this course teaches for that risk level. The Capstone is a general "Production Multi-Agent System with Bounded Autonomy," adaptable to any domain.

---

## Progress

**15 of 15 chapters complete — Volume 4 is finished.** Volume 4 kicked off 2026-07-11 and completed 2026-07-11. Chapter 01 completed 2026-07-11. Chapter 02 completed 2026-07-11. Chapter 03 completed 2026-07-11. Chapter 04 completed 2026-07-11. Chapter 05 completed 2026-07-11. Chapter 06 completed 2026-07-11. Chapter 07 completed 2026-07-11. Chapter 08 completed 2026-07-11. Chapter 09 completed 2026-07-11. Chapter 10 completed 2026-07-11. Chapter 11 completed 2026-07-11. Chapter 12 completed 2026-07-11. Chapter 13 completed 2026-07-11. Chapter 14 completed 2026-07-11. Chapter 15 completed 2026-07-11.

| Module | Chapters | Status |
|--------|----------|--------|
| 1 — Agent Foundations, Deepened | Ch 01–04 | ✅ Complete |
| 2 — Multi-Agent Engineering | Ch 05–08 | ✅ Complete |
| 3 — Specialized and Production Agent Capabilities | Ch 09–11 | ✅ Complete |
| 4 — Trustworthy, Evaluated, Production-Grade Agents | Ch 12–14 | ✅ Complete |
| 5 — Capstone | Ch 15 | ✅ Complete |

### Chapter-by-Chapter

| # | Chapter | Status |
|---|---------|--------|
| 01 | Agent Architecture Deep Dive — From Assistants to Autonomous Systems | ✅ Complete |
| 02 | Reasoning and Planning Patterns — ReAct, Plan-and-Execute, and Reflection | ✅ Complete |
| 03 | Tool Use and Function Calling at Scale | ✅ Complete |
| 04 | Agent Memory Systems — Working, Long-Term, and Episodic Memory | ✅ Complete |
| 05 | Multi-Agent Orchestration Patterns | ✅ Complete |
| 06 | Agent-to-Agent Communication and the A2A Protocol | ✅ Complete |
| 07 | Building Multi-Agent Systems with LangGraph | ✅ Complete |
| 08 | Human-in-the-Loop and Bounded Autonomy | ✅ Complete |
| 09 | Building Agents with the Claude Agent SDK — Subagents, Hooks, and Skills | ✅ Complete |
| 10 | Computer-Use and Browser Agents | ✅ Complete |
| 11 | Agentic RAG Revisited — Retrieval as a Tool for Autonomous Agents | ✅ Complete |
| 12 | Agent Evaluation — Trajectory Analysis and Task Success Metrics | ✅ Complete |
| 13 | Agent Security — Bounding Autonomy and Defending Against Excessive Agency | ✅ Complete |
| 14 | Production Agent Operations at Scale | ✅ Complete |
| 15 | Capstone — Production Multi-Agent System with Bounded Autonomy | ✅ Complete |

See [COURSE_INDEX.md](./COURSE_INDEX.md) for learning goals and the full chapter dependency map.

---

## Reference Materials

Every reference document is open-in-second-tab material — built for lookup, not for learning from scratch. See [reference/README.md](./reference/README.md) for the index (populated as chapters are written).

---

## How to Use This Handbook

1. **Complete Volumes 1–3 first** — this volume builds directly on agent fundamentals, MCP tool-serving, and RAG engineering
2. **Read chapters in order** — each chapter builds on previous ones
3. **Run every code example** — reading code is not learning code
4. **Complete the exercises** before moving to the next chapter
5. **Build the mini project** — this converts reading into doing
6. **Use the Fast Read box** — every chapter opens with a 5-minute skim for days when you're in a hurry or revisiting material
7. **Build the production project** at the end of each module

---

## The AI Engineering Handbook Series

| Volume | Title | Status | Repository |
|--------|-------|--------|-----------|
| 1 | AI Engineering — From Zero to Production | ✅ Complete (20 chapters) | [AI-Engineering-Handbook](https://github.com/Bschouha19/AI-Engineering-Handbook) |
| 2 | MCP Engineering | ✅ Complete (15 chapters) | [MCP-Engineering](https://github.com/Bschouha19/MCP-Engineering) |
| 3 | RAG Deep Dive | ✅ Complete (15 chapters) | [RAG-Deep-Dive](https://github.com/Bschouha19/RAG-Deep-Dive) |
| 4 | AI Agent Engineering | ✅ Complete (15 chapters) | [AI-Agent-Engineering](https://github.com/Bschouha19/AI-Agent-Engineering) |
| 5 | n8n Automation Engineering | 🔄 In Progress | [n8n-AI-Workflow-Automation](https://github.com/Bschouha19/n8n-AI-Workflow-Automation) |
| 6 | Vector Database Engineering | 🔜 Planned | — |
| 7 | Coding Agents | 🔜 Planned | — |
| 8 | DevOps AI | 🔜 Planned | — |
| 9 | Technical PM AI | 🔜 Planned | — |
| 10 | Enterprise AI Systems | 🔜 Planned | — |
| 11 | AI Architecture Patterns | 🔜 Planned | — |
| 12 | Real Production Case Studies | 🔜 Planned | — |

---

*Volume 4 started and completed July 2026. 15 of 15 chapters delivered.*
