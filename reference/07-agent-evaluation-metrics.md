# Reference 07 — Agent Evaluation Metrics

Quick-lookup companion to [Chapter 12's Core Concepts and Technology Comparison](../chapters/chapter-12-agent-evaluation.md#core-concepts). Use this when building or interpreting a trajectory-level agent evaluation harness — the chapter teaches *why* outcome-only evaluation isn't enough and builds the tooling by hand, this doc is for looking up the specifics without re-reading it.

> **Currency Note:** Verified 2026-07-11. Benchmark leaderboards, OTel spec stability, and SDK tracing configuration move fast — re-confirm against live sources before a production evaluation build, especially if reading this months later.

## The Three Evaluation Tiers

| Tier | Question it answers | This course's tooling |
|---|---|---|
| **Outcome** | Did the task succeed? | A single pass/fail or scored final answer |
| **Trajectory** | Was the *path* efficient and sound? | `TrajectoryRecorder`, Tool Correctness (coverage + efficiency, checked independently) |
| **Component** | Which specific piece broke? | Per-tool/per-subagent isolation testing |

**The rule to remember:** these are independent axes. A trajectory can pass outcome evaluation while failing efficiency — two agents can reach an identical correct answer via a "cheap and reliable" path or a "47-step disaster." Passing one tier says nothing about the others.

## Tool Correctness — Two Independent Checks

1. **Coverage**: were all the expected tools called?
2. **Efficiency**: was the trajectory free of unnecessary or redundant calls?

Check both, never combine into one boolean — a trajectory can call every needed tool while still calling the same tool three times with an identical query.

## LLM-as-Judge Bias — Five Named Categories

| Bias | What it is | Mitigation |
|---|---|---|
| Position bias | Favors whichever candidate appears first/second | Swap ordering, average scores across both |
| Verbosity bias | Favors longer responses regardless of quality | Explicit rubric scoring content, not length |
| **Self-preference bias** | Judge rates its own model family's outputs more favorably | Use a **different-family** judge, or explicitly flag/discount when judge and agent share a family |
| Format bias | Favors a specific output structure over substance | Rubric-driven scoring, not format matching |
| Calibration drift | Judge's scoring standard shifts over repeated use | Periodic re-anchoring against known-good/known-bad examples |

**Position bias and self-preference bias require separate fixes** — order-swap-and-average solves the first, not the second. Given this course's own consistent use of Claude throughout, using Claude to judge a Claude-built agent's trajectory carries a specific, real self-preference risk — don't assume order-swap mitigation covers it.

## Current Benchmark Landscape

| Benchmark | Status | Notes |
|---|---|---|
| GAIA (via Princeton's HAL leaderboard) | **Paused** for new-model updates | Explicitly refocused on agent reliability over leaderboard chasing (confirmed via the leaderboard's own page) |
| SWE-bench Verified | Current, increasingly saturated | Top scores cluster 85–95% |
| SWE-bench Pro | Current, harder successor | Top score is source-dependent — always cite a named source alongside any figure |
| WebArena / τ-bench / OSWorld | Current "frontline" benchmarks | Still actively cited in 2026 |
| AgentBench | Current but largely superseded | "More historically important than practically used" |
| MTEB / BEIR | Current standard for retrieval-*component* quality | Measures embeddings, not agent tool-use behavior |

**No single dominant benchmark exists for agentic/tool-using RAG or general agent trajectory quality specifically** — evaluation in 2026 is compositional: retrieval quality (MTEB/BEIR) + task success/trajectory (this doc's own tiers), not one consolidated score.

## SDK → LangSmith Tracing Quick Reference

`.claude/settings.local.json`:

```json
{
  "plugins": {
    "langsmith": {
      "TRACE_TO_LANGSMITH": "true",
      "CC_LANGSMITH_API_KEY": "${LANGSMITH_API_KEY}",
      "CC_LANGSMITH_PROJECT": "your-project-name"
    }
  }
}
```

Captures: user messages, tool calls, compaction events, **subagent runs**, assistant responses. Excludes: system prompts. Groups by `thread_id` in LangSmith's Threads tab. JS/TS also has a lighter-weight `wrapClaudeAgentSDK` helper as an alternative to the settings-file plugin.

## OTel GenAI Semantic Conventions — Stability Status

Current spec (v1.41 as of this course's research): defines `agent`, `workflow`, `tool`, and `model` spans plus required latency/token-usage metrics. **Nearly all `gen_ai.*` attributes still carry Development stability** — not Stable — meaning attribute names can change without a major version bump. No public stabilization timeline exists. Treat the pattern as production-usable today; treat exact attribute names as provisional. Major observability vendors (Datadog, Honeycomb, New Relic) and frameworks (LangChain, CrewAI, AutoGen) already emit/ingest OTel-compliant GenAI spans regardless of this formal status.

## Citation Verification Warning

This course's own research process caught an unfamiliar model name ("Claude Mythos Preview") on a benchmark leaderboard and initially flagged it as fabricated — a follow-up check found it's real (an invitation-only Anthropic preview model), but that access restriction means a public leaderboard score attributed to it is still unverifiable. **Two-layer check before citing any leaderboard entry:** (1) is the model name real, per the vendor's own current model list, and (2) could the citing party plausibly have obtained that score at all, given the model's actual access status?

---

*Verified: 2026-07-11*
