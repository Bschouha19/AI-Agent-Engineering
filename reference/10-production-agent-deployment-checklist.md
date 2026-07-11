# Reference 10 — Production Agent Deployment Checklist

Quick-lookup companion to [Chapter 14's Production Architecture](../chapters/chapter-14-production-agent-ops.md#production-architecture) and [Chapter 15's Capstone](../chapters/chapter-15-capstone.md). Use this when taking a single agent or multi-agent system from dev to production — it assumes you've already built the individual primitives each chapter covers; this is the composed, pre-launch view.

> **Currency Note:** Verified 2026-07-11. Fleet platform quotas, pricing, and named products in this space move fast — re-confirm before a production commitment, especially anything more than a few months old.

## Dev-to-Production Checklist

- [ ] **Bounded loop**: every agent has an explicit `max_iterations`/`max_turns`, decided deliberately for the task's actual complexity — never a framework default left unexamined (Ch01)
- [ ] **Least-privilege tools**: every `AgentDefinition.tools` (or equivalent) is an explicit allowlist, never omitted — an omitted field silently inherits the parent's full access (Ch09)
- [ ] **Identity**: every agent instance has a unique, individually-revocable identity — not a shared credential across a fleet or type (Ch13)
- [ ] **Authorization**: unconditional guarantees live in hooks (run before permission mode/`canUseTool`, cannot be routed around by a misconfiguration); human-judgment gates live in `canUseTool`, reserved for the tier that actually needs it (Ch08, Ch09)
- [ ] **Sandboxing**: any browser automation or untrusted code execution runs in a disposable container/microVM — see reference doc 09
- [ ] **Retrieved/fetched content** is treated as data, never instructions — webpages, retrieved documents, tool outputs (Ch10, Ch11)
- [ ] **Trajectory evaluation** runs in the same CI pipeline as outcome evaluation, not as a separate manual process (Ch12)
- [ ] **Fleet-wide budget governance is enforced, not just monitored** — a check that runs before a call proceeds, not a dashboard reviewed after the fact (Ch14)
- [ ] **Multi-agent tracing is unified** under one thread/session view, not reconstructed from disconnected per-agent logs (Ch06, Ch09, Ch12, Ch14)
- [ ] **No internal incentive structure rewards raw agent-usage volume** (a leaderboard, a usage-based bonus) — a confirmed real governance anti-pattern (Ch14)
- [ ] **Integration seams are tested explicitly** — retries, respawns, async approval delays — not just each component in isolation (Ch15)

## Fleet Governance — LiteLLM Quick Reference

| Field | Scope | Purpose |
|---|---|---|
| `tpm_limit` / `rpm_limit` | Per-agent | Tokens/requests per minute |
| `session_tpm_limit` / `session_rpm_limit` | Per-session | Same, scoped to one session |
| `max_iterations` | Per-session | Same bound discipline as Ch01's `max_iterations` |
| `max_budget_per_session` | Per-session | USD ceiling |
| `team_settings.max_budget` + `budget_duration` | Per-team/org | Daily/weekly/monthly/yearly spend cap, enforced via 429 on breach |

Redis-backed enforcement is required for consistency across multiple proxy instances at real fleet scale.

## Fleet Platform Landscape

| Platform | Scope | Notes |
|---|---|---|
| Claude Managed Agents | Single-agent hosting | Per-token pricing + $0.08/session-hour active-runtime charge (beta — not a committed GA figure) |
| AWS Bedrock AgentCore | Fleet-scale, multi-framework | Confirmed current quotas: 5,000 (US East/West) / 2,500 (other regions) active sessions, 200 TPS per agent/account; "Managed Harness" bundles tools/environment/memory/identity/observability |
| Google Gemini Enterprise Agent Platform | Fleet-scale, Google-model-centric | Rebrand of Vertex AI's agent tooling (2026) — re-verify naming given rebrand pace in this space |
| Microsoft Agent Framework 1.0 | Fleet-scale, Microsoft-stack-centric | Merges AutoGen + Semantic Kernel; leans on Azure AI Foundry for hosting, Entra identity, observability |
| LiteLLM (self-hosted governance) | Governance layer, any provider | Not a hosting platform — sits in front of a self-hosted fleet on any infrastructure |

## Multi-Agent Tracing Setup

**In-process subagents (Ch09's Agent tool):** captured automatically within the same session/`thread_id` — no extra configuration needed beyond `TRACE_TO_LANGSMITH`.

**Genuine A2A (Ch06) crossing a real trust/process boundary:** LangSmith's Agent Server A2A endpoint automatically converts the A2A protocol's `contextId` into a LangSmith `thread_id`; `taskId` identifies individual requests within that thread. For non-LangGraph agents, extract `thread_id` from A2A request metadata and configure OTel tracing to group into the same thread manually.

**Framework-agnostic baseline (no LangSmith):** W3C `traceparent` header propagation across HTTP-based agent-to-agent hops.

**Don't confuse the two mechanisms** — in-process subagent trace correlation (via `parent_tool_use_id`) and A2A-to-`thread_id` unification solve the same "one trace, many agents" problem for two different trust-boundary situations. Using the wrong one for your system's actual architecture is a real, current mistake this course's own Capstone chapter had to correct.

## Pre-Launch Sign-Off

- [ ] Load-tested with deliberately adversarial timing (retries during pending approvals, concurrent identity issuance) — not just component-level unit tests
- [ ] Approval rate tracked as a first-class metric with an alert threshold around 90% (a rate above that is a miscalibration signal, not reassurance)
- [ ] Fleet-wide spend trend reviewed against team caps before go-live, not discovered after a budget cycle closes
- [ ] Security review completed against reference doc 08's checklist
- [ ] Sandboxing technology matches the actual isolation need — see reference doc 09's decision guide, don't default to whatever's cheapest

---

*Verified: 2026-07-11*
