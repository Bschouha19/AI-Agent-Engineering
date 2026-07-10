# Reference 06 — Multi-Agent Topology Comparison

Quick-lookup companion to [Chapter 05's Technology Comparison and Decision Framework](../chapters/chapter-05-multi-agent-orchestration.md#technology-comparison-multi-agent-topologies-at-a-glance). Use this when choosing (or justifying) a multi-agent topology — the chapter teaches *why* and builds each pattern by hand, this doc is for looking up the answer without re-reading it.

> **Currency Note:** Verified 2026-07-11. Multi-agent framework support and star counts move fast — re-confirm against current sources if reading this much later.

## Is Multi-Agent Even the Right Call?

```
Does the task decompose into genuinely INDEPENDENT,
parallelizable sub-tasks?
│
├─ NO (sequential, each step depends on the last) ──► Single agent +
│                                                       Plan-and-Execute (Ch02)
│
└─ YES ──► Continue

Do different pieces need different tools, different scoped
access, or different underlying models?
│
└─ NO (same agent could handle all pieces equally well) ──► Still
                                                              consider a
                                                              single agent

Is there a specific safety/audit/compliance reason to
enforce a hard boundary between pieces of work?
│
Have you already scaled up a single agent (better prompt,
better model, more tools) and confirmed it genuinely can't
do the job?
│
└─ YES to enough of the above ──► Multi-agent is justified.
                                    Pick a topology below.
```

**Default: single agent.** Current evidence confirms multi-agent typically costs 2–5x more per task and can *underperform* a well-designed single agent on tightly interdependent work. Add agents only for a specific, evidence-backed reason — not because a task "feels" complex.

## Topology Quick Reference

| | Supervisor/Worker | Hierarchical | Swarm | Mesh | Pipeline |
|---|---|---|---|---|---|
| **Production maturity** | Most production-proven | Real, minority pattern | Largely experimental at scale | Least production-proven | Production-proven (closer to Plan-and-Execute) |
| **Structure** | One supervisor, direct-reporting workers | Supervisor of supervisors | No fixed supervisor; direct agent-to-agent handoff | Direct peer-to-peer, no hierarchy at all | Sequential, one agent per stage |
| **Cost multiplier vs. single agent** | ~2–3x | Higher, scales with depth | Varies; academic work only at scale | Highest coordination overhead of the group | Comparable to Plan-and-Execute |
| **Current named implementation** | LangGraph tool-calling pattern (recommended) or `langgraph-supervisor` | Hand-rolled `Command`-based routing | `langgraph-swarm` (NOT the deprecated OpenAI Swarm framework) | No dominant current implementation | Chapter 02's `StateGraph`, one agent per stage |
| **Best for** | Most multi-agent needs | Genuinely large specialist counts needing sub-grouping | Exploratory, dynamic hand-off scenarios where production maturity gap is acceptable | Rarely the right default | Sequential workflows already well-served by Plan-and-Execute |

## Hard Numbers Worth Remembering

- **Supervisor/worker (lean, few specialists)**: ~1.5–2x latency, ~2–3x cost vs. single agent.
- **Hierarchical (3-tier)**: adds **≥6 seconds of pure coordination latency** before any worker even starts (2-second LLM call per level) — "depth that looks right on a whiteboard often performs worse than a flat dispatch with a clearer schema."
- **Anthropic's own parallel-fan-out research system**: ~**15x tokens** vs. single-agent chat, **>90% quality improvement** — but explicitly scoped to independent, parallelizable research tasks; explicitly *worse* for tightly-coupled work like coding.
- **Circuit breaker threshold for a whole agent**: ~3–5 consecutive failures before tripping (higher/longer cooldown than a single tool call, per Chapter 03's per-tool version).

## Fallback Hierarchy (When a Worker Is Unavailable)

1. Alternative specialist agent
2. Rule-based (non-LLM) handler
3. Cheaper/smaller model
4. Escalate to a human

Never skip straight from "specialist unavailable" to "task fails" — and never silently proceed as if nothing were missing. Always surface the gap explicitly in the final synthesis.

## Common Bugs

| Symptom | Cause |
|---|---|
| Request never completes, no error | No timeout on a specialist dispatch |
| `Command` handoff doesn't take effect, no error raised | State-schema key mismatch between the routing graph and the target graph |
| Multi-agent result worse than a single agent | Task was actually sequential, not parallelizable — wrong topology choice |
| One specialist keeps getting retried despite repeated failure | Circuit breaker not actually wired into the dispatch path |

## Naming Trap

**Swarm (the topology)** ≠ **OpenAI Swarm (the framework)**. OpenAI Swarm is archived, superseded by the OpenAI Agents SDK — which itself implements supervisor/worker, not a peer swarm. The framework's deprecation says nothing about the topology's viability. LangGraph's actively-maintained `langgraph-swarm` is current, real proof the topology itself is alive.

---

*Verified: 2026-07-11*
