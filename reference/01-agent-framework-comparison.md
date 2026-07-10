# Reference 01 — Agent Framework Comparison

Quick-lookup companion to [Chapter 01's Technology Comparison section](../chapters/chapter-01-agent-architecture-deep-dive.md#technology-comparison-the-2026-agent-framework-landscape). Use this when choosing a framework for a real project — the chapter teaches *why* the landscape looks like this, this doc is for looking up specifics without re-reading the chapter.

> **Currency Note:** Verified 2026-07-11 via live GitHub/PyPI/official docs. Framework versions and star counts move fast — re-confirm against each project's current source before a production decision, especially if you're reading this more than a few months after the verification date below.

## At a Glance

| Framework | Version (verified 2026-07-11) | Governance | This course's role for it |
|---|---|---|---|
| Claude Agent SDK | `claude-agent-sdk` v0.2.115 (Python) | Anthropic | **Primary teaching vehicle** — single-agent and subagent patterns (Chapter 09) |
| LangGraph | v1.2.9, 37k★ | LangChain | **Primary teaching vehicle** — multi-agent orchestration (Chapter 07) |
| OpenAI Agents SDK | v0.18.1 | OpenAI | Technology Comparison only |
| CrewAI | 55.3k★ | CrewAI Inc. | Technology Comparison only |
| Microsoft Agent Framework (MAF) | 1.0 GA (2026-04-03) | Microsoft | Technology Comparison only |
| Google ADK | v2.4.0 | Google | Technology Comparison only |
| AutoGen / AG2 | `microsoft/autogen` dormant since ~April 2026; `ag2ai/ag2` active fork | Microsoft (dormant) / community (AG2) | Technology Comparison only — legacy migration context |

## Decision Guide

**Choose the Claude Agent SDK when:**
- You're building a single agent or a small tree of subagents, not a large stateful multi-agent graph
- You want first-class hooks, fine-grained permission modes, and native session resume/fork
- You're already standardized on Claude models and want the tightest integration
- You want Managed Agents (hosted REST API) as a deployment option instead of self-hosting the loop

**Choose LangGraph when:**
- Your system is genuinely graph-shaped — multiple agents, conditional routing, cycles, checkpointing
- You need a vendor-neutral orchestration layer that isn't tied to one model provider
- You need durable, resumable state across long-running or human-in-the-loop workflows (Chapter 08)

**Choose OpenAI Agents SDK when:**
- Your team and infra are already standardized on the OpenAI ecosystem
- Note: it is the confirmed production successor to OpenAI Swarm, which is now explicitly educational-only — don't start new work on Swarm

**Choose CrewAI when:**
- You want a lighter-weight, role-based "crew" mental model (specific named roles collaborating) rather than a full state-machine graph

**Choose Microsoft Agent Framework (MAF) when:**
- You're already on Microsoft Foundry / Azure OpenAI / GitHub Copilot SDK
- You specifically want the Magentic-One orchestration pattern — confirmed stable and GA inside MAF as of this writing, not deprecated

**Choose Google ADK when:**
- You're building on Google's ecosystem and need first-class A2A (Agent2Agent) protocol integration — ADK's docs are hosted under the same `a2aproject` GitHub org as the A2A protocol itself

**Avoid / migrate away from:**
- `microsoft/autogen` for new work — it's been relatively dormant since April 2026; the community fork `ag2ai/ag2` is the actively-maintained continuation of that lineage
- OpenAI Swarm for anything beyond learning — explicitly positioned as educational-only, with the Agents SDK as its production successor

## Related Protocols (not frameworks, but load-bearing for the decision)

| Protocol | Status (verified 2026-07-11) | Role |
|---|---|---|
| MCP (Model Context Protocol) | Actively evolving (Volume 2's subject) | Agent-to-tool/data communication |
| A2A (Agent2Agent) | v1.0.0 (March 2026), v1.0.1 (May 2026); donated to the Linux Foundation, governed by `a2aproject` | Agent-to-agent communication — complementary to MCP, not a replacement for it |

MCP has **not** absorbed agent-to-agent communication — confirmed via the live MCP spec changelog as of this writing. The two protocols solve different problems and are meant to be used together, not chosen between. See Chapter 06 and reference doc 04 (MCP vs. A2A Decision Guide) for the full decision tree.

## The Framework Thread — Why This Course Doesn't Pick One

As of this writing, no single framework in the table above has the kind of unambiguous "obvious default" status that MCP had for Volume 2 or pgvector had for Volume 3. The landscape is genuinely fragmented and still consolidating (note MAF's own 2026 consolidation of AutoGen + Semantic Kernel, and the AutoGen → AG2 fork, both within the last year). This course teaches every core architectural concept — the agentic loop, bounded autonomy, multi-agent orchestration, memory — framework-agnostically first, then illustrates it concretely in LangGraph and/or the Claude Agent SDK. Chapter 01's Protocol-based agent interface pattern (`typing.Protocol`) is the concrete engineering technique that keeps a real system decoupled from this choice, so it can absorb whichever framework consolidates as the field matures without a rewrite.

---

*Verified: 2026-07-11*
