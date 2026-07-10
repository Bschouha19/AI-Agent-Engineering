# AI Agent Engineering Course Index
### Master Autonomous Agent Systems — From Fundamentals to Production, Volume 4

---

> **Prerequisites:** [Volume 1 — AI Engineering](https://github.com/Bschouha19/AI-Engineering-Handbook) (especially Ch 07 Embeddings, Ch 09 RAG, Ch 10–11 Agents and Multi-Agent Systems), [Volume 2 — MCP Engineering](https://github.com/Bschouha19/MCP-Engineering) (tool-serving, the other half of the tool-use problem this volume completes), and [Volume 3 — RAG Deep Dive](https://github.com/Bschouha19/RAG-Deep-Dive) (especially Ch 06 Dense Retrieval, Ch 12 RAG Evaluation, Ch 13 Trustworthy RAG, Ch 14 Production RAG Operations)
>
> **Assumed knowledge:** Python, basic agent concepts (the ReAct loop, tool calling), MCP server/client basics, RAG fundamentals, Docker

> **What you will build by the end:** A production-grade, multi-agent system with bounded autonomy — reasoning and planning, persistent memory, multi-agent orchestration, agent-to-agent communication, human oversight calibrated to risk, a trajectory-level evaluation harness, and defenses against the security risks unique to autonomous, tool-granted systems.

---

## Progress

**8 of 15 chapters complete** — Volume 4 kicked off 2026-07-11. Chapter 01 completed 2026-07-11. Chapter 02 completed 2026-07-11. Chapter 03 completed 2026-07-11. Chapter 04 completed 2026-07-11. Chapter 05 completed 2026-07-11. Chapter 06 completed 2026-07-11. Chapter 07 completed 2026-07-11. Chapter 08 completed 2026-07-11.

---

## The Framework Thread

This course teaches agent engineering concepts framework-agnostically first, the same way Volume 3 taught retrieval theory before naming a vector database. As of this writing, no single agent framework has the kind of obvious-default status MCP had for Volume 2 or pgvector had for Volume 3 — the landscape (LangGraph, the Claude Agent SDK, OpenAI's Agents SDK, CrewAI, Microsoft's Agent Framework, Google's ADK) is genuinely fragmented and still consolidating. This course uses **two** frameworks for hands-on production examples: **LangGraph** for multi-agent orchestration, and the **Claude Agent SDK** for single-agent and subagent patterns (hooks, permission modes, Skills). Every other framework is covered accurately in Technology Comparison tables without being the primary teaching vehicle for any chapter.

## The Autonomy Thread

This course escalates stakes by **autonomy and blast radius**, not document domain. Modules 1–2 (Chapters 01–08) use low-stakes, reversible scenarios — reusing "Aperture Cloud," the same fictional B2B SaaS company from Volume 3, where it fits naturally. Modules 3–4 (Chapters 09–14) introduce agents with genuinely consequential capabilities (write access to production systems, real financial or communication actions, real browser interaction, autonomous fleets) — paired explicitly with the human-oversight, evaluation, and security techniques this course teaches for exactly that risk level. The Capstone (Chapter 15) is a general "Production Multi-Agent System with Bounded Autonomy," adaptable to any domain.

---

## Course Structure

### MODULE 1 — AGENT FOUNDATIONS, DEEPENED
*Go past Volume 1 Chapter 10–11's introductory agent treatment into real engineering depth.*

| # | Chapter | File | Status | Key Skills |
|---|---------|------|--------|-----------|
| 01 | Agent Architecture Deep Dive — From Assistants to Autonomous Systems | chapters/chapter-01-agent-architecture-deep-dive.md | ✅ Complete | Agent failure taxonomy, when agents are (and aren't) the right architecture, Protocol-based agent interfaces |
| 02 | Reasoning and Planning Patterns — ReAct, Plan-and-Execute, and Reflection | chapters/chapter-02-reasoning-planning-patterns.md | ✅ Complete | Reasoning loop design, planning vs. reactive execution, self-critique/reflection passes |
| 03 | Tool Use and Function Calling at Scale | chapters/chapter-03-tool-use-at-scale.md | ✅ Complete | Agent-side tool selection, parallel tool calls, error recovery, result validation |
| 04 | Agent Memory Systems — Working, Long-Term, and Episodic Memory | chapters/chapter-04-agent-memory-systems.md | ✅ Complete | Short-term/working memory, long-term persistent memory, memory retrieval |

**Module 1 Learning Goal:** Understand exactly what separates a "chatbot with a tool" from a genuine autonomous agent, and build the reasoning, tool-use, and memory foundations everything else in this course depends on.

**Module 1 Project:** A single, well-architected agent with a bounded reasoning loop, real tool access with error recovery, and persistent memory across sessions.

---

### MODULE 2 — MULTI-AGENT ENGINEERING
*Master coordination between multiple agents, and the protocols that make it safe.*

| # | Chapter | File | Status | Key Skills |
|---|---------|------|--------|-----------|
| 05 | Multi-Agent Orchestration Patterns | chapters/chapter-05-multi-agent-orchestration.md | ✅ Complete | Supervisor/worker, hierarchical, and swarm topologies; when each is production-proven vs. experimental |
| 06 | Agent-to-Agent Communication and the A2A Protocol | chapters/chapter-06-agent-to-agent-protocol.md | ✅ Complete | A2A protocol fundamentals, MCP vs. A2A, agent discovery and identity |
| 07 | Building Multi-Agent Systems with LangGraph | chapters/chapter-07-langgraph-multi-agent.md | ✅ Complete | Graph/state-machine orchestration, production multi-agent workflows |
| 08 | Human-in-the-Loop and Bounded Autonomy | chapters/chapter-08-human-in-the-loop.md | ✅ Complete | Approval gates, human-in-the-loop vs. human-on-the-loop, escalation design |

**Module 2 Learning Goal:** Coordinate multiple agents safely and effectively, understand the protocols purpose-built for agent-to-agent communication, and design the human-oversight layer that makes autonomy tolerable.

**Module 2 Project:** A multi-agent system (orchestrator + at least two worker agents) with a defined communication protocol and at least one human approval gate.

---

### MODULE 3 — SPECIALIZED AND PRODUCTION AGENT CAPABILITIES
*Where general agent engineering meets real, high-capability tools — and the stakes start escalating.*

| # | Chapter | File | Status | Key Skills |
|---|---------|------|--------|-----------|
| 09 | Building Agents with the Claude Agent SDK — Subagents, Hooks, and Skills | chapters/chapter-09-claude-agent-sdk.md | 🔜 Next | Subagent delegation, hook-based control, permission modes, Skills |
| 10 | Computer-Use and Browser Agents | chapters/chapter-10-computer-use-browser-agents.md | 🔜 | Browser automation agents, visual UI interpretation, real-world action risk |
| 11 | Agentic RAG Revisited — Retrieval as a Tool for Autonomous Agents | chapters/chapter-11-agentic-rag-revisited.md | 🔜 | Retrieval as an agent tool, bounded retrieval loops, connecting Volume 3's retrieval stack to agent reasoning |

**Module 3 Learning Goal:** Apply this course's foundations to real, high-capability tool categories — a production-grade single-agent framework, computer-use, and retrieval-as-a-tool — where a mistake has genuine, not hypothetical, consequences.

**Module 3 Project:** An agent with at least one genuinely consequential capability (real write access, real browser action, or bounded retrieval-driven research), instrumented with the safeguards Module 2 introduced.

---

### MODULE 4 — TRUSTWORTHY, EVALUATED, PRODUCTION-GRADE AGENTS
*The difference between an agent demo and an agent system you can put your name behind.*

| # | Chapter | File | Status | Key Skills |
|---|---------|------|--------|-----------|
| 12 | Agent Evaluation — Trajectory Analysis and Task Success Metrics | chapters/chapter-12-agent-evaluation.md | 🔜 | Trajectory vs. outcome evaluation, agent benchmarks, catching costly-but-correct paths |
| 13 | Agent Security — Bounding Autonomy and Defending Against Excessive Agency | chapters/chapter-13-agent-security.md | 🔜 | Least-privilege agent design, sandboxing, agent identity, named agentic security frameworks |
| 14 | Production Agent Operations at Scale | chapters/chapter-14-production-agent-ops.md | 🔜 | Agent fleet management, cost governance, multi-agent failure debugging |

**Module 4 Learning Goal:** Build an agent system that knows whether it's actually succeeding (not just producing plausible output), stays within its intended capability boundary, and can be operated reliably as a fleet, not just a single instance.

**Module 4 Project:** An evaluated, security-reviewed agent system with a documented capability boundary and fleet-level cost/rate governance.

---

### MODULE 5 — CAPSTONE
*Assemble everything into one production system.*

| # | Chapter | File | Status | Key Skills |
|---|---------|------|--------|-----------|
| 15 | Capstone — Production Multi-Agent System with Bounded Autonomy | chapters/chapter-15-capstone.md | 🔜 | All of Volume 4: reasoning, memory, multi-agent orchestration, human oversight, evaluation, security, fleet operations |

**Capstone System:** A production multi-agent system — an orchestrator coordinating specialized worker agents, at least one agent with genuinely consequential real-world capability, agent-to-agent communication, persistent memory, a risk-tiered human approval layer, a trajectory-level evaluation harness, and fleet-level operational monitoring. Explicitly designed so a reader can substitute their own domain (customer support automation, DevOps automation, research automation) with no conceptual gap.

---

## Chapter Dependency Map

```
Ch 01 (Agent Architecture Deep Dive)
  └─► Ch 02 (Reasoning and Planning Patterns)
        └─► Ch 03 (Tool Use at Scale)
              └─► Ch 04 (Agent Memory Systems)
                    └─► Ch 05 (Multi-Agent Orchestration Patterns)
                          └─► Ch 06 (Agent-to-Agent Protocol)
                                └─► Ch 07 (LangGraph Multi-Agent)
                                      └─► Ch 08 (Human-in-the-Loop)
                                            ├─► Ch 09 (Claude Agent SDK)
                                            ├─► Ch 10 (Computer-Use/Browser Agents)
                                            └─► Ch 11 (Agentic RAG Revisited)
                                                  └─► Ch 12 (Agent Evaluation)
                                                        └─► Ch 13 (Agent Security)
                                                              └─► Ch 14 (Production Agent Ops)
                                                                    └─► Ch 15 (Capstone)
```

---

## Learning Path Options

### Standard Path (recommended)
Read all 15 chapters in order. Takes approximately 6–8 weeks part-time.

### Multi-Agent Fast Track
Ch 01 → Ch 02 → Ch 03 → Ch 05 → Ch 06 → Ch 07 → Ch 08 → Ch 12 → Ch 13 → Ch 15
Skips: memory depth (Ch 04 in full), computer-use (Ch 10), agentic RAG depth (Ch 11), production ops depth (Ch 14)
Takes approximately 3–4 weeks part-time — the fastest path to a working, safe multi-agent system.

### Single-Agent Production Path
Ch 01 → Ch 02 → Ch 03 → Ch 04 → Ch 09 → Ch 12 → Ch 13
Focus: one production-grade, evaluated, secured single agent, skipping multi-agent orchestration entirely.
Takes approximately 2–3 weeks part-time.

---

*Last updated: 2026-07-11 — 0 of 15 chapters complete. Kickoff session.*
