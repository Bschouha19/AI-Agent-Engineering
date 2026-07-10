# Reference 05 — Memory Systems Comparison

Quick-lookup companion to [Chapter 04's Technology Comparison and Decision Framework](../chapters/chapter-04-agent-memory-systems.md#technology-comparison-agent-memory-layers). Use this when choosing a persistent memory approach for an agent — the chapter teaches *why* and builds the underlying mechanic by hand, this doc is for looking up the answer without re-reading it.

> **Currency Note:** Verified 2026-07-11. Memory-layer tooling is fast-moving — re-confirm star counts, pricing tiers, and benchmark figures against current sources if reading this much later.

## Which Memory Type Does This Task Need?

```
Does the task only need to remember things within its own run?
│
└─ YES ──► Working memory only (Chapter 01's messages list) — stop here

Does it benefit from recalling a SPECIFIC past event when a
similar one recurs?
│
└─ YES ──► Episodic memory (recency + relevance + importance retrieval)

Does it need durable FACTS that don't attach to any single event?
│
└─ YES ──► Semantic memory (usually customer/user-scoped)

Does it benefit from a RULE OF THUMB that generalizes across many
past instances, not any one of them?
│
└─ YES ──► Procedural memory (usually agent-scoped, NOT user-scoped)

Whichever you picked: did you build the PRUNING path alongside
the promotion path? This is not optional past a certain scale.
```

## Memory Layer Quick Reference

| | Mem0 | Letta (formerly MemGPT) | Zep |
|---|---|---|---|
| **Positioning** | De facto standard; automatic extraction/consolidation pipeline | OS-inspired virtual context management | Temporal reasoning over facts that change over time |
| **Key differentiator** | Conflict resolution between contradictory memories, built-in `user_id`/`agent_id`/session scoping | Agent manages its own context paging (RAM vs. disk framing) via memory tools | Graphiti knowledge-graph engine; reasons about *when* a fact was true |
| **Benchmark data point** | 49.0% on LongMemEval | Not directly compared in this pass | 63.8% on LongMemEval (vs. Mem0's 49.0%) |
| **Best fit** | General-purpose agent memory, fastest path to a working system | Long-running, stateful agents that should self-manage memory allocation | Facts with a validity window ("what was true as of X date") |
| **Status (2026-07-11)** | v2.0 (June 2026), 59,600+★, Apache 2.0 | Active; center of gravity around "Letta Code" | Active; free Community Edition **deprecated** — use Zep Cloud or self-run Graphiti |

## The Core Retrieval Formula

Every system above ultimately scores candidate memories on some combination of:

```
score = normalize(relevance) + normalize(recency) + normalize(importance)
```

- **Relevance** — cosine similarity between query and memory embeddings (Volume 3 Chapter 06's dense retrieval, unchanged)
- **Recency** — exponential decay since last access (Generative Agents paper's constant: γ=0.995/hour)
- **Importance** — an LLM-assigned 1–10 significance rating, captured at write time

**Always normalize each term to [0,1] before summing** — otherwise whichever term has the widest raw numeric range silently dominates every ranking.

## Memory Taxonomy Quick Reference

| Type | Scope (typical) | Lifecycle | Chapter 04 example |
|---|---|---|---|
| Working | Single run | Discarded at run end (default) | Chapter 01's `messages` list |
| Episodic | `user_id` + `agent_id` | Retrieved by recency+relevance+importance | "We had a database failover like this in March" |
| Semantic | `user_id` + `agent_id` | Durable until explicitly updated | "This customer is on the Enterprise tier" |
| Procedural | `agent_id` only | Refined across many instances, generalizes across customers | "Check retry-path logic first on billing-service reopens" |

**Common scoping bug**: scoping procedural memory per-customer prevents the agent from ever generalizing what it learns. Scoping episodic/semantic memory per-agent-only (no `user_id`) leaks one customer's data into another's context. Get this backwards in either direction and something real breaks — not just an inconvenience.

## Failure-Mode Quick Diagnosis

| Symptom | Likely Cause |
|---|---|
| Context-length error from a memory-backed agent | Unbounded memory growth — no pruning pipeline was ever built |
| Retrieved memory is old, low-value, crowds out better matches | Large unpruned store, or un-normalized retrieval scoring |
| One customer's data appears in another's session | Tenant-scoping bug — check both `user_id` and `agent_id` on every write and read |
| Reflexion-style agent isn't improving across repeated tasks | Memory not actually being written on success/failure, or `top_k` too low to surface it |
| A learned lesson doesn't generalize across customers | Procedural memory incorrectly scoped by `user_id` |

## Security Non-Negotiables

- Enforce tenant scoping **at the query layer** (built into every `add`/`search` call) — never as an application-level filter applied after the fact.
- **Memory poisoning ≠ prompt injection.** Injection ends with the session; poisoning persists into every future session that retrieves the corrupted memory. Remediation costs are categorically higher once something is written and persisted.
- Treat every memory write exactly as untrusted as a tool result (Chapter 01's discipline) — an agent's own past output is not automatically safe to retrieve and trust blindly.

---

*Verified: 2026-07-11*
