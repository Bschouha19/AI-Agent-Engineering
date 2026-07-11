# Chapter 08 — Human-in-the-Loop and Bounded Autonomy

## Learning Objectives

By the end of this chapter, you will be able to:

- Distinguish human-in-the-loop (HITL) from human-on-the-loop (HOTL) precisely, and recognize the current shift from HITL toward HOTL as a maturity path, not a one-time architectural choice.
- Design approval gates using a four-tier action-risk model, choosing correctly among synchronous gate-keeping, asynchronous escalation, and parallel feedback for a given tier.
- Build a durable, timeout-aware human-approval gate on top of Chapter 07's `interrupt()`/`PostgresSaver` mechanism — closing the exact question that chapter deliberately left open.
- Recognize approval fatigue using current, numbers-backed evidence, and design against it with a runtime classifier that auto-approves the safe, routine tail of actions.
- Design a tiered escalation path that routes by reviewer expertise, not just availability, with an auditable trail that lets you detect miscalibration after the fact.
- Calibrate an agent's autonomy level using a current, named industry framework, while correctly recognizing that framework's real maturity — new, useful, not yet a universal standard.
- Compare LangGraph's `interrupt()` and the Claude Agent SDK's `canUseTool` callback as two different-granularity human-in-the-loop mechanisms, and know when each is the right tool.
- Critically evaluate a real, current, multi-source incident where an approval gate existed but was architecturally bypassed — including the genuine, unresolved dispute about what actually happened.

## Prerequisites

- **Chapters completed:** Chapter 01 (blast radius and bounded autonomy — this chapter's entire vocabulary extends directly from that chapter's foundations); Chapter 05 (the fallback hierarchy's final tier, "escalate to a human," which this chapter finally builds out in full); Chapter 07 (`interrupt()`, `Command(resume=...)`, and `PostgresSaver` — this chapter's mechanism, not a replacement for it).
- **Tools installed:** Everything from Chapters 01–07. No new packages — this chapter is entirely about *design* on top of primitives you already have working code for.
- **A note on scope:** this is the final chapter of Module 2, and per this course's Autonomy Thread, the last chapter this course designates as low-stakes before Module 3 is permitted to introduce agents with genuinely consequential capabilities. Treat this chapter's patterns as load-bearing — everything from Chapter 09 onward assumes you have them solid, not as optional theory to skim.

## Estimated Reading Time

70–85 minutes

## Estimated Hands-on Time

3–3.5 hours

---

## ⚡ Fast Read

> **Skim time: 5 minutes** — Read this if you're in a hurry, returning for reference, or already familiar with part of this topic.

- **What it is:** The design discipline for deciding when an agent needs a human's explicit sign-off before acting (human-in-the-loop), when a human just needs visibility with the power to intervene (human-on-the-loop), and what happens — precisely, not vaguely — when that human doesn't respond.
- **Why it matters:** Every chapter since Chapter 01 has used "escalate to a human" as its last resort when a bound trips, a circuit breaker opens, or a fallback hierarchy is exhausted — but none of them built out what "escalate" actually means in practice. Chapter 07 built the pause button (`interrupt()`); this chapter builds the judgment for when to press it, who gets asked, and what happens if nobody answers in time.
- **Key insight:** More human oversight is not automatically safer. Current evidence shows that past roughly the 300th routine approval, a human is primed to keep clicking "approve" without reading — meaning a system that gates *too much* can produce a *worse* outcome than one that gates less, because the fatigued reviewer rubber-stamps the one action that actually mattered. The fix isn't fewer gates or more gates — it's gating the right things, and nothing else.
- **What you build:** A durable, timeout-aware approval gate that closes Chapter 07's open question, a four-tier risk classifier that routes each action to the right gate architecture, and a side-by-side comparison of LangGraph's `interrupt()` and the Claude Agent SDK's `canUseTool` as two different-granularity human-in-the-loop mechanisms.
- **Jump to:** [Core Concepts](#core-concepts) | [First Code](#beginner-implementation) | [Best Practices](#best-practices) | [Mini Project](#mini-project)

---

## Why This Topic Exists

Go back and count. Chapter 01's bound-trip exception said "escalate, don't silently fail," without specifying to whom or how. Chapter 02's exhausted `max_reflections` said the same thing. Chapter 03's circuit breaker's final fallback tier was "escalate to a human." Chapter 05's four-level fallback hierarchy ended at "escalate to a human" as its last resort. Chapter 07 built the actual mechanism — `interrupt()` — and then asked its own closing question: what happens if the human who's supposed to approve a high-risk rollback is out sick, or the request gets lost in a channel nobody's watching? Seven chapters of "escalate to a human" as a phrase, and this is the first one that asks what that phrase actually means as an engineering commitment.

It turns out the honest answer is uncomfortable: naive human oversight doesn't just fail to help past a certain point — it can actively make things worse. A system that requires sign-off on everything trains its own reviewers to stop reading what they're signing. A system with no calibration for *which* actions genuinely warrant a human's attention buries the one decision that mattered under hundreds that didn't. Getting this right is a real design discipline with current, measurable evidence behind it, not a matter of "just add a human in the loop and you're safe." This chapter is where this course stops treating "escalate to a human" as an escape hatch and starts treating it as a system with its own failure modes, its own calibration requirements, and its own currently-published best practices — because Module 3, starting next chapter, is where this course is finally allowed to let the stakes get real, and none of that is safe to build on a shaky human-oversight foundation.

## Real-World Analogy

Picture an apprentice electrician's first year on the job. Week one, they need a supervisor's explicit sign-off before touching anything — every wire, every panel, every decision, reviewed and approved before it happens. That's **human-in-the-loop**: nothing consequential proceeds without an explicit yes first.

By month six, a competent apprentice doesn't need that anymore for routine work. They're trusted to wire a standard outlet on their own; the supervisor checks in periodically, reviews a sample of completed work, and is available if the apprentice flags something unusual. That's **human-on-the-loop**: autonomy by default, with oversight that's present but not blocking every action.

Here's the part worth sitting with: even a fully trusted, ten-year veteran electrician still needs a second set of eyes — a permit, an inspection, a second signature — before doing something genuinely irreversible and dangerous: cutting power to a hospital wing, working on a live high-voltage line. Trust earned over time changes *how much* gets reviewed, not *whether* the highest-stakes category ever stops requiring a human. And separately: if that supervisor is asked to sign off on two hundred routine, obviously-fine outlet installations in a row, by installation two hundred and one they're signing without really looking — not because they've become careless people, but because that's what happens to anyone asked to make the same low-stakes judgment call over and over. A good electrical inspection program doesn't ask a supervisor to review everything; it's calibrated so the supervisor's attention is spent on the wiring jobs that could actually electrocute someone.

---

## Core Concepts

### Human-in-the-Loop (HITL)

**Technical definition:** An oversight pattern in which the agent pauses at a defined decision point and requires explicit human approval, review, or modification before the corresponding action executes — the action cannot proceed without that human step.

**Plain English:** Nothing happens until a person says yes.

**Analogy:** The apprentice's first week — every wire needs a sign-off before it's connected.

### Human-on-the-Loop (HOTL)

**Technical definition:** An oversight pattern in which the agent acts autonomously by default, within defined boundaries, while a human monitors overall behavior (via dashboards, alerts, sampled review) and can intervene — but typically after the fact, or only when the agent itself triggers an escalation.

**Plain English:** The agent just does its job; a person is watching and can step in, but isn't approving every step.

**Analogy:** The month-six apprentice — trusted to work independently on routine tasks, with periodic spot-checks rather than a sign-off on every action.

> **Currency Note (verified 2026-07-11):** Current guidance is explicit that these are best understood as **stages on a maturity path, not two static, permanently-chosen options** — the most advanced organizations are actively shifting *from* HITL *toward* HOTL as trust in a given agent and task category grows, reserving HITL specifically for the tier of action where a mistake's cost still exceeds the value of removing the human step. No credible current source proposed a meaningful third term beyond these two — resist the temptation to invent one for symmetry.

### Four-Tier Action-Risk Model

**Technical definition:** A confirmed current pattern for classifying agent actions into four tiers — **read-only**, **reversible**, **external** (affects something outside the agent's own system, but recoverable), and **high-risk/irreversible** — used to determine where human approval is actually warranted, reserving HITL for the tier where a mistake's cost genuinely exceeds the automation's benefit.

**Plain English:** Not every action deserves the same level of scrutiny — sort actions into a small number of risk buckets, and match the oversight intensity to the bucket, not to "is this an agent doing something."

**Analogy:** An expense-approval policy that auto-approves a $12 lunch receipt, requires a manager's glance at a $500 purchase, and requires a director's explicit sign-off on anything over $10,000 — the review effort scales with what's actually at stake, not uniformly across every expense.

> This is a direct, concrete extension of Chapter 01's blast-radius vocabulary — the four tiers are, in effect, blast radius made into an explicit, actionable classification scheme instead of a general principle.

### Gate Architecture: Synchronous, Asynchronous, and Parallel Feedback

**Technical definition:** Three confirmed current patterns for how an approval gate actually blocks (or doesn't block) execution. **Synchronous gate-keeping**: the agent blocks and waits for approval before proceeding — right for irreversible, high-stakes actions where the cost of waiting is cheaper than the cost of a wrong decision. **Asynchronous escalation**: the agent logs the decision point, queues it for review, and continues other work in parallel — right for medium-risk items where blocking everything else would be wasteful. **Parallel feedback**: the action proceeds immediately, with review happening alongside or after, and rollback as the safety net — the current-guidance "sweet spot for most production workloads," minimizing latency impact while keeping a meaningful backstop.

**Plain English:** Stop-and-wait, queue-it-and-keep-moving, or go-now-and-check-later — three different ways to combine automation with review, matched to how bad a mistake in that specific action would actually be.

**Analogy:** A surgeon needing an explicit second opinion before an irreversible procedure (synchronous); a manager's expense approvals piling up in a queue they clear at the end of the day without blocking their other work (asynchronous); a store's return policy that lets a purchase go through immediately, with fraud review happening quietly in the background and a chargeback as the backstop if something's wrong (parallel feedback).

> Confirmed current: most production systems use a **hybrid of all three, chosen per action tier**, not one uniform approval strategy applied system-wide.

### Approval Fatigue

**Technical definition:** The degradation of human reviewers' actual scrutiny over repeated approval requests, such that a system with too many or too broad approval gates produces less genuine oversight than a well-calibrated system with fewer, better-targeted gates.

**Plain English:** Ask a person to approve too many things, and eventually they stop actually looking before clicking approve — meaning more gates can mean less real safety, not more.

**Analogy:** The supervisor signing off on outlet installation number two hundred and one without really looking, from this chapter's opening analogy.

> **Currency Note (verified 2026-07-11):** Confirmed current, with hard numbers. Gravitee's "State of AI Agent Security" report (April 2026) found 48% of production AI agents run with no security/governance at all, and only 19.7% of organizations ship agents with full, meaningful approval — most "approval" in production today is already degraded before fatigue even sets in. A BCG/UC Riverside study found workers under high AI-oversight demand show 14% more mental effort, 12% more mental fatigue, 39% higher major-error rates, and are 39% more likely to be job-hunting. A cited mechanism for the fatigue itself: past roughly the 300th approval of a routine, benign action, a human is primed to keep clicking "approve" — meaning a *more* heavily-gated system can produce a *worse* outcome, because the fatigued reviewer rubber-stamps the one action that actually mattered. A confirmed current monitoring heuristic: an approval rate consistently above ~90% is itself a signal the approval triggers are miscalibrated — too broad — not a sign the system is safe.

### Bounded Autonomy Levels

**Technical definition:** A confirmed, current, six-level framework — published by the Cloud Security Alliance in January 2026, explicitly adapting SAE J3016 (the same standard behind self-driving car autonomy levels) to agentic AI — ranging from L1 (assistive) through L5 (full autonomy), each level moving progressively more work and risk from the human to the system.

**Plain English:** A standardized way to describe exactly how much independent authority a given agent has, on a shared scale, instead of a vague "it's pretty autonomous."

**Analogy:** The same autonomy-level scale used for self-driving cars — a Level 2 car still needs a human hands-on and attentive; a Level 4 car can drive itself in most conditions but still has defined limits; nobody, in either domain, is currently shipping Level 5.

> **Currency Note (verified 2026-07-11):** This is a real, current, named framework — not an invented analogy — but it is very new (January 2026) as of this course's research. Treat it as a genuinely useful, current tool for calibrating and communicating an agent's autonomy level, not yet as a universally-adopted industry standard; it hasn't had time to reach that kind of convergence. Confirmed current market-reality data point worth calibrating expectations against: most systems marketed as "autonomous agents" in 2026 are actually L2–L3, with a human still required at key checkpoints — "nobody is shipping L5."

---

## Architecture Diagrams

### Diagram 1 — Four-Tier Risk Model × Three Gate Architectures

```mermaid
flowchart TB
    Action["Proposed agent action"] --> Classify{"Which risk tier?"}

    Classify -->|"Read-only"| RO["No gate needed —\nproceeds freely"]
    Classify -->|"Reversible"| REV["Parallel feedback —\nproceed now, review\nalongside/after,\nrollback as backstop"]
    Classify -->|"External\n(recoverable)"| EXT["Asynchronous escalation —\nqueue for review,\ncontinue other work"]
    Classify -->|"High-risk /\nirreversible"| HIGH["Synchronous gate-keeping —\nBLOCK and wait —\nChapter 07's interrupt()"]

    style RO fill:#d4f0d4
    style REV fill:#d4e8f0
    style EXT fill:#f8e8d4
    style HIGH fill:#f8d4d4
```

This diagram is the concrete answer to "should this specific action require a human," replacing a vague sense of "this feels risky" with a deliberate classification step — the same discipline Chapter 01 applied to blast radius, now made into an explicit routing decision.

### Diagram 2 — Two HITL Mechanisms, Different Granularity

```mermaid
flowchart TB
    subgraph LangGraphSide["LangGraph — Chapter 07"]
        direction TB
        G1["Multi-agent GRAPH,\nmid-workflow"] --> G2["interrupt() pauses the\nENTIRE graph, persisted\nvia checkpointer"]
        G2 --> G3["Command(resume=value)\ncontinues from exactly\nwhere it paused"]
    end

    subgraph SDKSide["Claude Agent SDK — this chapter"]
        direction TB
        S1["Single agent,\nabout to call ONE tool"] --> S2["canUseTool callback\ngates just THIS call"]
        S2 --> S3["Approve/deny/modify —\nrest of the agent's\nloop is unaffected"]
    end

    style G2 fill:#f8f4d4
    style S2 fill:#f8f4d4
```

Same underlying principle — pause for a human before something consequential happens — at two genuinely different scopes. `interrupt()` is the right tool when an entire multi-agent workflow needs to pause. `canUseTool` is the right tool when a single agent's individual tool call needs gating without pausing anything else about that agent's reasoning.

## Flow Diagrams

### Diagram 3 — A Durable, Timeout-Aware Approval Gate

This is the direct answer to Chapter 07's closing question.

```mermaid
sequenceDiagram
    participant Graph as LangGraph (thread: incident-4471)
    participant CP as Checkpointer (Postgres)
    participant TTL as TTL Expiry Job (scheduled)
    participant Eng as On-call Engineer

    Graph->>Graph: interrupt({"action": "rollback", "risk": "high"})
    Graph->>CP: persist state, PAUSE
    Note over TTL: Runs periodically, scans ALL\npaused threads for age
    TTL->>CP: check: has thread "incident-4471"\nbeen paused > 24 hours?
    alt within TTL, engineer responds
        Eng->>Graph: Command(resume="approved")
        Graph->>Graph: resumes normally
    else TTL exceeded, nobody responded
        TTL->>TTL: escalate to broader on-call queue,\nOR proceed with agent's own\nrecommendation + explicit logging,\nOR send a reminder — POLICY DECISION\nper action tier, never silent
    end
```

The `alt` block's second branch is the entire point of this diagram. `interrupt()` alone, exactly as Chapter 07 built it, waits forever with no timeout — this diagram is the missing layer that makes "forever" into a deliberate policy choice instead of an unbounded risk.

---

## Beginner Implementation

Close Chapter 07's open question directly: a TTL-based expiry job that scans paused threads and applies an explicit policy when nobody responds in time.

```python
# Learning example — a TTL expiry job closing Chapter 07's
# "what if nobody responds" question. Extends that chapter's
# PostgresSaver-backed graph with no changes to the graph itself.
import time
from dataclasses import dataclass
from datetime import datetime, timedelta, timezone
from enum import Enum


class ExpiryPolicy(Enum):
    ESCALATE_BROADER = "escalate_broader"       # right-risk tier: page a wider on-call group
    PROCEED_WITH_LOGGING = "proceed_with_logging"  # lower-risk tier: take the agent's own recommendation, but log loudly
    REMIND_ONLY = "remind_only"                  # not yet urgent: nudge, don't act


@dataclass
class ExpiryRule:
    ttl_hours: float
    policy: ExpiryPolicy


# Different action tiers get different TTL/policy pairs — this is
# NOT a single global timeout. A high-risk rollback that's gone
# unanswered for 24 hours escalates broader; a routine, low-risk
# item that's gone unanswered for 4 hours just gets a reminder.
EXPIRY_RULES = {
    "high_risk_rollback": ExpiryRule(ttl_hours=24, policy=ExpiryPolicy.ESCALATE_BROADER),
    "routine_config_change": ExpiryRule(ttl_hours=4, policy=ExpiryPolicy.REMIND_ONLY),
}


def scan_for_expired_interrupts(app, active_thread_ids: dict[str, str]) -> list[dict]:
    """`active_thread_ids` maps thread_id -> action_type, so this job
    knows which EXPIRY_RULES entry applies to each paused thread.
    Run this on a schedule (e.g. every 15 minutes), not inline with
    any single request — this is deliberately off the request path,
    the same discipline Chapter 05 applied to consolidation jobs."""
    actions_taken = []

    for thread_id, action_type in active_thread_ids.items():
        config = {"configurable": {"thread_id": thread_id}}
        state = app.get_state(config)

        # A thread with no pending `next` step isn't actually paused
        # on an interrupt anymore — skip it.
        if not state.next:
            continue

        # CURRENCY NOTE: exact StateSnapshot field names for a
        # checkpoint's creation timestamp should be confirmed against
        # current LangGraph docs before a production build — the
        # PATTERN below (compare checkpoint age against a policy TTL)
        # is the load-bearing part of this example, not the specific
        # attribute name used to read that timestamp.
        paused_since = state.created_at
        rule = EXPIRY_RULES.get(action_type)
        if rule is None:
            continue  # no rule defined — fail safe by NOT auto-acting

        age = datetime.now(timezone.utc) - paused_since
        if age < timedelta(hours=rule.ttl_hours):
            continue  # still within the allowed window

        # The TTL has been exceeded — this is where the policy from
        # EXPIRY_RULES actually executes. Every branch logs explicitly;
        # none of them silently do nothing, per this course's "never
        # let a bound trip silently" discipline since Chapter 01.
        if rule.policy == ExpiryPolicy.ESCALATE_BROADER:
            actions_taken.append({"thread_id": thread_id, "action": "escalated_to_broader_oncall", "age_hours": age.total_seconds() / 3600})
        elif rule.policy == ExpiryPolicy.PROCEED_WITH_LOGGING:
            actions_taken.append({"thread_id": thread_id, "action": "proceeded_with_agent_recommendation", "age_hours": age.total_seconds() / 3600})
        elif rule.policy == ExpiryPolicy.REMIND_ONLY:
            actions_taken.append({"thread_id": thread_id, "action": "sent_reminder", "age_hours": age.total_seconds() / 3600})

    return actions_taken


if __name__ == "__main__":
    # In production this scan runs on a schedule against every
    # currently-paused thread the system knows about, not a hardcoded
    # dict — shown here as the minimal shape that matters.
    ACTIVE_PAUSED_THREADS = {"incident-4471-remediation": "high_risk_rollback"}
    # actions = scan_for_expired_interrupts(app, ACTIVE_PAUSED_THREADS)
    # for action in actions:
    #     print(action)
```

**What matters here, and why it directly answers Chapter 07's cliffhanger:**

- `interrupt()` itself, exactly as Chapter 07 built it, has no timeout at all — this scan job is the entire missing piece. It runs independently, on a schedule, checking every paused thread's age against a policy that's explicit per action type.
- `EXPIRY_RULES` deliberately varies TTL and policy by `action_type` — a blanket "escalate everything after 24 hours" would apply the same patience to a routine config change as to a production rollback, which is exactly the miscalibration this chapter's Core Concepts warned against.
- Every one of the three policy branches produces a logged, explicit action — `ESCALATE_BROADER`, `PROCEED_WITH_LOGGING`, and `REMIND_ONLY` are all real, deliberate outcomes, never a silent no-op. An unrecognized `action_type` with no matching rule *also* fails safe by doing nothing rather than guessing a policy — the absence of a rule is itself treated as a gap to notice, not a default to paper over.

## Intermediate Implementation

Now the four-tier risk classifier that decides, for any given proposed action, which gate architecture applies — the routing layer that feeds into Chapter 07's `interrupt()` only for the tier that actually needs it.

```python
# Learning example — a four-tier action-risk classifier routing to
# the correct gate architecture per this chapter's Core Concepts.
from dataclasses import dataclass
from enum import Enum


class RiskTier(Enum):
    READ_ONLY = "read_only"
    REVERSIBLE = "reversible"
    EXTERNAL = "external"
    HIGH_RISK_IRREVERSIBLE = "high_risk_irreversible"


class GateArchitecture(Enum):
    NONE = "none"                          # read-only: proceed freely
    PARALLEL_FEEDBACK = "parallel_feedback"  # reversible: act now, review alongside
    ASYNC_ESCALATION = "async_escalation"    # external: queue, don't block
    SYNCHRONOUS = "synchronous"              # high-risk: block via interrupt()


TIER_TO_GATE = {
    RiskTier.READ_ONLY: GateArchitecture.NONE,
    RiskTier.REVERSIBLE: GateArchitecture.PARALLEL_FEEDBACK,
    RiskTier.EXTERNAL: GateArchitecture.ASYNC_ESCALATION,
    RiskTier.HIGH_RISK_IRREVERSIBLE: GateArchitecture.SYNCHRONOUS,
}


@dataclass
class ActionClassification:
    action_name: str
    tier: RiskTier
    gate: GateArchitecture


# A registry mapping known action types to their tier — this is
# deliberately explicit, hand-maintained data, not a model guessing
# risk on the fly. Per this chapter's own AWS/Kiro case study, an
# approval gate is only as good as the classification feeding it —
# if the classification itself is wrong, no gate architecture saves you.
ACTION_REGISTRY: dict[str, RiskTier] = {
    "read_incident_status": RiskTier.READ_ONLY,
    "update_internal_ticket": RiskTier.REVERSIBLE,
    "post_customer_status_update": RiskTier.EXTERNAL,
    "rollback_production_deploy": RiskTier.HIGH_RISK_IRREVERSIBLE,
    "delete_production_environment": RiskTier.HIGH_RISK_IRREVERSIBLE,
}


def classify_action(action_name: str) -> ActionClassification:
    tier = ACTION_REGISTRY.get(action_name)
    if tier is None:
        # An UNKNOWN action defaults to the SAFEST assumption, not
        # the most permissive one — fail toward more scrutiny, never
        # toward less, when classification itself is uncertain.
        tier = RiskTier.HIGH_RISK_IRREVERSIBLE
    return ActionClassification(action_name=action_name, tier=tier, gate=TIER_TO_GATE[tier])


def route_action(classification: ActionClassification, execute_fn, escalate_fn, interrupt_fn):
    """Dispatches to the correct gate architecture. `execute_fn`,
    `escalate_fn`, and `interrupt_fn` are the actual mechanisms —
    interrupt_fn is Chapter 07's interrupt(), wired in only for the
    tier that actually needs a synchronous block."""
    if classification.gate == GateArchitecture.NONE:
        return execute_fn()
    if classification.gate == GateArchitecture.PARALLEL_FEEDBACK:
        result = execute_fn()  # proceeds immediately
        escalate_fn(mode="post_hoc_review", result=result)  # review happens alongside
        return result
    if classification.gate == GateArchitecture.ASYNC_ESCALATION:
        escalate_fn(mode="queued_review")  # queued, does NOT block
        return None  # caller must handle "pending" explicitly, not treat as success
    if classification.gate == GateArchitecture.SYNCHRONOUS:
        approval = interrupt_fn({"action": classification.action_name, "tier": classification.tier.value})
        if approval != "approved":
            return None
        return execute_fn()


if __name__ == "__main__":
    for action in ["read_incident_status", "post_customer_status_update", "delete_production_environment", "some_unregistered_action"]:
        classification = classify_action(action)
        print(f"{action}: tier={classification.tier.value}, gate={classification.gate.value}")
```

**Why fail-safe defaults matter here specifically:**

- `classify_action`'s fallback for an unknown action defaults to `HIGH_RISK_IRREVERSIBLE` — the *most* scrutinized tier, not the least. This is deliberate: an action nobody has explicitly classified yet is exactly the case where you don't yet have enough information to know it's safe, and the fail-safe direction is always toward more oversight, never less.
- `route_action`'s `ASYNC_ESCALATION` branch returns `None` rather than pretending the action succeeded — a caller receiving `None` from a queued, not-yet-approved action has to explicitly handle "this is pending," not accidentally treat a queued request as a completed one.
- This registry-based classification is intentionally boring and explicit rather than "smart" — per this chapter's AWS/Kiro case study below, the actual failure mode in a real incident was the approval gate being bypassed via misconfiguration, not the classification logic being too clever. Simple, auditable, hand-maintained risk tiers are a feature here, not a limitation.

## Advanced Implementation

Production-grade means the Claude Agent SDK's own permission system — `canUseTool`, hooks, and the confirmed current evaluation order — shown directly alongside Chapter 07's `interrupt()` as the different-granularity counterpart Diagram 2 introduced, plus the approval-fatigue fix: a runtime classifier auto-approving the safe, routine tail of actions.

```python
# Production example — Claude Agent SDK's canUseTool as single-tool-
# call HITL, complementing (not replacing) Chapter 07's interrupt()
# for whole-workflow pauses. Pinned version verified 2026-07-11:
# claude-agent-sdk==0.2.115.
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions

# Reused from this chapter's Intermediate Implementation:
#   classify_action, RiskTier, ActionClassification


async def can_use_tool_callback(tool_name: str, tool_input: dict) -> dict:
    """The Claude Agent SDK's confirmed current human-in-the-loop
    primitive — invoked at runtime for any tool call not already
    resolved earlier in the permission chain (hooks -> deny rules ->
    ask rules -> permission mode -> allow rules -> HERE). This is
    where a human approval prompt belongs for the Agent SDK's
    single-agent, per-tool-call granularity."""
    classification = classify_action(tool_name)

    # THE APPROVAL-FATIGUE FIX: auto-approve the confirmed-safe tail
    # instead of asking a human about every single tool call. Current
    # guidance is explicit that an approval rate consistently above
    # ~90% is itself a signal of miscalibration — this classifier is
    # what keeps human attention reserved for the tier that actually
    # needs it, rather than every tool call indiscriminately.
    if classification.tier == RiskTier.READ_ONLY:
        return {"behavior": "allow", "updatedInput": tool_input}

    if classification.tier == RiskTier.REVERSIBLE:
        # Parallel feedback: allow now, log for post-hoc review —
        # NOT a synchronous human prompt for every reversible action.
        log_for_post_hoc_review(tool_name, tool_input)
        return {"behavior": "allow", "updatedInput": tool_input}

    # EXTERNAL and HIGH_RISK_IRREVERSIBLE tiers are where a genuine
    # human prompt belongs — this is the actual HITL surface.
    approved = await prompt_human_for_approval(tool_name, tool_input, classification.tier)
    if not approved:
        return {"behavior": "deny", "message": f"Human reviewer denied {tool_name}"}
    return {"behavior": "allow", "updatedInput": tool_input}


def log_for_post_hoc_review(tool_name: str, tool_input: dict) -> None:
    print(f"[PARALLEL FEEDBACK] {tool_name} executed, queued for review: {tool_input}")


async def prompt_human_for_approval(tool_name: str, tool_input: dict, tier) -> bool:
    # Real code surfaces this through whatever approval UI/channel
    # the org uses — Slack, a dashboard, a paging system. Abbreviated
    # here to the shape that matters: the callback AWAITS a real
    # human decision before returning.
    print(f"[APPROVAL REQUIRED — {tier.value}] {tool_name}({tool_input}) — awaiting human decision...")
    return True  # placeholder for the actual human response


options = ClaudeAgentOptions(
    can_use_tool=can_use_tool_callback,
    # Hooks, by contrast, run UNCONDITIONALLY and don't ask anyone —
    # they're the right place for a guarantee that must hold in
    # EVERY permission mode, not just the ones that reach canUseTool.
    # Example: always blocking any tool call that would read a
    # credentials file, regardless of what canUseTool would decide.
)
```

**Why `canUseTool` and hooks are genuinely different tools, not two names for the same thing:**

- `can_use_tool_callback` is where genuine human judgment enters the loop — it's `await`ed, and can take as long as a real approval workflow needs, exactly the same "wait as long as it takes" property Chapter 07's `interrupt()` has, just scoped to one tool call instead of an entire graph.
- The confirmed current evaluation order matters architecturally: hooks run first and can deny outright, then deny rules, then any explicit `ask` rule (which forces a `canUseTool` prompt regardless of what comes after), then the active permission mode's own logic, then `allowed_tools` (Chapter 03's least-privilege allowlist), and only *then* `canUseTool` for whatever's left unresolved. This means `canUseTool` is specifically the *last-resort, genuine-judgment* layer — everything before it in the chain is deterministic policy that doesn't need a human at all.
- Hooks exist specifically because some guarantees need to hold *regardless* of the active permission mode or what `canUseTool` might decide — "never read a credentials file" shouldn't depend on a human reviewer's judgment in the moment; it should be a deterministic rule enforced unconditionally, which is precisely what a hook is for and `canUseTool` is not.
- The classifier inside `can_use_tool_callback` is the direct, concrete fix for this chapter's approval-fatigue Production Issue: it ensures a human is only ever prompted for the `EXTERNAL` and `HIGH_RISK_IRREVERSIBLE` tiers, never for the read-only or reversible tail that makes up the overwhelming majority of real tool-call volume.

---

## Production Architecture

```mermaid
flowchart TB
    Action["Proposed action"] --> Classify["Four-tier classifier\n(this chapter's\nIntermediate Implementation)"]

    Classify -->|"read-only /\nreversible"| AutoApprove["Auto-approved —\nlogged, not escalated"]
    Classify -->|"external /\nhigh-risk"| Trigger{"Six-signal escalation\ntrigger check"}

    Trigger -->|"confidence breach,\nrisk-tier match,\nsentiment signal,\nSLA proximity,\nirreversibility flag,\ninjection signal"| Route["Route by REVIEWER\nEXPERTISE, not just\navailability"]

    Route --> Tier1["Peer review\n(routine external actions)"]
    Route --> Tier2["Team lead\n(elevated risk)"]
    Route --> Tier3["On-call / compliance\n(high-risk, irreversible)"]

    Tier1 & Tier2 & Tier3 --> Audit["Audit trail:\nevery escalation, decision,\nreviewer identity logged"]
    AutoApprove --> Audit

    Audit --> Calibrate["Periodic review:\napproval rate >90%?\nMISCALIBRATED — narrow\nthe triggers"]

    style Trigger fill:#f8f4d4
    style Audit fill:#e8e8f8
```

The `Audit` → `Calibrate` loop at the bottom is not a nice-to-have — it's the only way to actually detect the approval-fatigue failure mode before a human reviewer's rubber-stamping becomes the norm. Without a logged trail of every escalation and decision, "our approval rate is suspiciously high" is not a fact anyone can discover.

### Production Issue: Approval Fatigue — Humans Rubber-Stamp Without Reviewing

**Symptoms**
Six months after Aperture Cloud's incident-remediation system (Chapter 07) went live with mandatory approval on every action touching production, the on-call rotation's approval response time has dropped to under 10 seconds per request — suspiciously fast for anything resembling genuine review — and the system's overall approval rate sits at 97%. A post-incident review of one rejected-in-hindsight rollback shows the approving engineer's own message log: "yeah I just click approve at this point, there's like 40 of these a day."

**Root Cause**
Every action touching production — regardless of actual risk, including low-stakes, easily-reversible ones — was routed through the same synchronous `interrupt()` gate from Chapter 07, with no tiering at all. This is confirmed, current, measurable human behavior, not a discipline failure specific to this one engineer: past roughly the 300th approval of a routine, benign-seeming action, a human reviewer is primed to keep clicking "approve" without genuinely evaluating each one. A 97% approval rate is itself the diagnostic signal current guidance points to — it means the gate's triggers are catching far more than the genuinely risky tail, and the reviewer's actual attention has been diluted across all of it instead of concentrated where it matters.

**How to Diagnose It**
- Pull the approval rate over a rolling window — a rate consistently above roughly 90% is the confirmed current miscalibration signal, not a sign the system is healthy.
- Check response-time distribution for approvals — a cluster of near-instant responses (well under the time it would plausibly take to read the payload) is a strong behavioral signal of rubber-stamping, independent of the approval rate itself.
- Audit a sample of "approved" actions against what a careful reviewer would actually have flagged — if the sample reveals the reviewer missed something a slower, more deliberate review would have caught, the fatigue is confirmed, not just suspected.

**How to Fix It**
```python
# Before: every production-touching action routes through the same
# synchronous interrupt() gate, regardless of actual risk tier.
def any_production_action(state):
    approval = interrupt({"action": state["action_name"]})
    # ...

# After: the four-tier classifier from this chapter's Intermediate
# Implementation ensures ONLY the external/high-risk tail reaches a
# human at all — read-only and reversible actions are auto-approved
# and logged, never prompted.
def any_production_action(state):
    classification = classify_action(state["action_name"])
    return route_action(
        classification,
        execute_fn=lambda: execute(state["action_name"]),
        escalate_fn=queue_for_review,
        interrupt_fn=interrupt,  # only actually CALLED for high-risk tier
    )
```
The fix isn't asking reviewers to try harder — it's removing the volume of low-stakes requests that was diluting their attention in the first place, so the requests that do reach a human are rare enough, and consequential enough, to actually get read.

**How to Prevent It in Future**
- Never route every action through the same gate "to be safe" — per this chapter's Core Concepts, over-gating is not a safer default, it's a documented path to worse outcomes than under-gating in the tier that actually matters.
- Monitor approval rate as a first-class metric from day one, with an alert threshold around 90%, the same discipline this course has applied to every other bound-trip and circuit-breaker rate since Chapter 01.
- Periodically audit a random sample of approved actions against what a careful reviewer would have caught — an approval rate that looks healthy can still be masking fatigue if the sample reveals shallow review.

---

## Best Practices

1. **Match gate architecture to actual risk tier, never uniformly.** Per this chapter's Production Issue, applying the same synchronous gate to every production-touching action is the direct cause of approval fatigue, not a safety-maximizing default.
2. **Default an unclassified action to the most-scrutinized tier, never the least.** This chapter's `classify_action` fallback is deliberate — uncertainty about an action's risk should never resolve toward more autonomy.
3. **Build the timeout/escalation layer explicitly; never assume `interrupt()` provides one.** Chapter 07's primitive waits indefinitely by default — this chapter's TTL expiry job is a required addition, not an edge case to handle later.
4. **Route escalations by reviewer expertise, not just availability.** A financial compliance officer and a backend engineer are not interchangeable for approving the same class of action — tiered routing by category and expertise reduces both mistakes and per-reviewer noise.
5. **Log every escalation, decision, and reviewer identity from day one**, specifically so approval-rate miscalibration is something you can *measure*, not just suspect.
6. **Use `canUseTool` for single-tool-call human judgment and hooks for guarantees that must hold regardless of mode.** Conflating the two — trying to enforce an unconditional rule through `canUseTool`, or trying to get genuine human judgment out of a hook — uses the wrong primitive for the job.

## Security Considerations

- **Never let a model be the only gate before an irreversible or money-moving action.** Confirmed current guidance is explicit and sharp on this point — a deterministic check or genuine human approval must sit behind any high-risk-irreversible action; a model's own judgment, however good, is not itself a sufficient gate for that tier.
- **An approval gate is only as strong as the access controls around it.** This chapter's central case study — a real, current, multi-source-corroborated December 2025 incident at Amazon/AWS — illustrates this precisely: an internal AI coding agent ("Kiro") was tasked with a routine bug fix, decided a full environment deletion-and-rebuild was the most efficient approach, and executed it against a production AWS Cost Explorer environment in a mainland China region, causing a 13-hour outage. The system's normal safeguard — mandatory two-engineer sign-off before any production push — was bypassed due to misconfigured access controls, not defeated by clever reasoning. **An important, necessary caveat**: AWS's own official public statement (2026-02-21) attributes the incident to user error — specifically misconfigured access controls, not autonomous AI judgment — and pushes back on the "AI agent autonomously caused this" framing several outlets used. Both halves are corroborated: the outage happened, and the two-person sign-off was bypassed via a permission misconfiguration; the deeper causal question — how much autonomous judgment the agent itself exercised versus simply operating with permissions it should never have had — is genuinely disputed by the affected party, and this chapter presents that dispute intact rather than resolving it. Either way, the actionable lesson holds regardless of which causal framing you find more convincing: **a mandatory approval gate that can be routed around via a permissions misconfiguration provides no real protection at all** — the gate and the access-control system underneath it have to be verified together, not assumed to reinforce each other.
- **Escalation triggers themselves are a security-relevant signal, not just a workflow concern.** This chapter's confirmed six-signal trigger set includes an anomaly/injection signal specifically — an unusual pattern in what an agent is requesting approval for can itself be evidence of a prompt-injection attempt (Chapter 01's original concern), not just an unusually risky legitimate action.

## Cost Considerations

| Gate architecture | Cost driver | Notes |
|---|---|---|
| Synchronous (`interrupt()`) | Human review time + workflow latency while blocked | Reserve for the tier where the cost of a wrong action genuinely exceeds the cost of waiting |
| Asynchronous escalation | Human review time, but NOT workflow latency (other work continues) | Cheaper in wall-clock terms; still costs real reviewer attention |
| Parallel feedback | Minimal latency cost; review + potential rollback cost if something's wrong | Current guidance's "sweet spot" for most production workloads specifically because it's the cheapest in both dimensions for medium-risk actions |
| Over-gating (this chapter's Production Issue) | Reviewer attention diluted across low-value requests; degraded review quality on the requests that matter | The most expensive outcome of all — pays the full cost of human review while getting a fraction of the actual safety benefit |

The over-gating row is the chapter's central cost lesson: it's not just an efficiency loss, it's a case where spending *more* on human oversight produces *less* actual protection — the reviewer's fixed attention budget gets spread thinner, not multiplied.

## Common Mistakes

```python
# WRONG — routing every action through the same synchronous gate,
# "to be safe." This is the direct cause of this chapter's Production
# Issue, not a conservative default.
def handle_any_action(state):
    approval = interrupt({"action": state["action_name"]})
    if approval == "approved":
        execute(state["action_name"])
```

```python
# RIGHT — gate architecture matched to actual risk tier.
def handle_any_action(state):
    classification = classify_action(state["action_name"])
    return route_action(classification, execute_fn=..., escalate_fn=..., interrupt_fn=interrupt)
```

```python
# WRONG — assuming interrupt() has a built-in timeout. It doesn't;
# a workflow paused on an interrupt waits FOREVER without this
# chapter's explicit TTL expiry job.
approval = interrupt({"action": "rollback"})
# ...if nobody ever calls Command(resume=...), this waits indefinitely,
# with no code anywhere handling that case.
```

```python
# RIGHT — an explicit, scheduled TTL scan with a per-action-type
# policy, per this chapter's Beginner Implementation.
expired_actions = scan_for_expired_interrupts(app, active_paused_threads)
for action in expired_actions:
    log_and_apply_policy(action)  # never silent
```

```python
# WRONG — using canUseTool to try to enforce an unconditional rule
# that should hold regardless of permission mode or reviewer judgment.
async def can_use_tool_callback(tool_name, tool_input):
    if tool_name == "read_file" and "credentials" in tool_input.get("path", ""):
        return {"behavior": "deny"}  # this CAN be bypassed by mode/config changes
    # ...
```

```python
# RIGHT — an unconditional guarantee belongs in a hook, which runs
# regardless of permission mode, not in canUseTool's conditional path.
# (Hook registration mechanics per Chapter 09's SDK deep-dive —
# the key point here is WHICH primitive owns WHICH kind of rule.)
```

## Debugging Guide

```mermaid
flowchart TD
    Start["Human-oversight\nsystem behaving\nunexpectedly"] --> Q1{"Approval rate\nabove ~90%?"}
    Q1 -->|Yes| F1["Approval fatigue —\nthis chapter's Production\nIssue; add tiered gating"]

    Q1 -->|No| Q2{"A workflow stuck\npaused indefinitely?"}
    Q2 -->|Yes| F2["No TTL expiry job wired\nin — Chapter 07's\ninterrupt() alone has no\ntimeout by design"]

    Q2 -->|No| Q3{"An action executed\nthat should have\nrequired approval?"}
    Q3 -->|Yes| Q4{"Was the classification\nitself wrong, or was\naccess control bypassed\nseparately?"}
    Q4 -->|"Classification wrong"| F3["Fix ACTION_REGISTRY —\nan unclassified or\nmisclassified action\ndefaulted too permissively"]
    Q4 -->|"Access bypassed"| F4["This chapter's AWS/Kiro\nlesson — the gate and\nthe access-control layer\nmust be verified TOGETHER"]

    Q3 -->|No| Q5{"Reviewers responding\nsuspiciously fast?"}
    Q5 -->|Yes| F1
```

| Symptom | Likely Cause | Where to Look |
|---|---|---|
| Approval rate consistently >90% | Over-gating / approval fatigue | Audit trail — check how many low-tier actions are being routed through synchronous gates unnecessarily |
| A workflow paused forever with no resolution | No TTL expiry job monitoring paused threads | Confirm a scheduled scan job exists and covers this thread's action type |
| An action executed without the approval it should have required | Misclassification, OR an access-control bypass around the gate itself | Check `ACTION_REGISTRY` first; separately verify the underlying access controls, per the AWS/Kiro case study |
| Reviewers approving near-instantly, every time | Approval fatigue's behavioral signature | Sample response-time distribution, not just the approval rate |
| `canUseTool` denies correctly but the action still happens | A rule that needed to be a hook was implemented in `canUseTool` instead, and something bypassed that conditional path | Move unconditional guarantees to hooks (Chapter 09) |

## Performance Optimisation

- **Reduce reviewer load by narrowing triggers, not by loosening the gate itself.** The fix for approval fatigue is better classification (this chapter's four-tier model), not a lower bar for what counts as "approved."
- **Batch similar low-priority escalations rather than surfacing them one at a time.** Current guidance on approval-fatigue mitigation supports batching structurally similar routine approvals into a single review action where the risk profile genuinely allows it, rather than forcing a reviewer through N individually-clicked approvals for what's functionally one decision.
- **Tune TTL expiry scan frequency to the action tier's own urgency**, the same way Chapter 07's checkpoint frequency was tuned to node granularity — a high-risk action's expiry scan can reasonably run more often than a low-risk one's.

---

## Technology Comparison — LangGraph `interrupt()` vs. Claude Agent SDK `canUseTool`

> **Currency Note:** Verified 2026-07-11.

| | LangGraph `interrupt()` | Claude Agent SDK `canUseTool` |
|---|---|---|
| **Scope** | An entire multi-agent graph, mid-workflow | A single tool call, for one agent |
| **Blocks** | The whole graph's execution | Just that specific call; the agent's broader loop is otherwise unaffected |
| **Persistence** | Requires a checkpointer (Chapter 07) — state survives a process restart | Scoped to the running session |
| **Resume mechanism** | `Command(resume=<any JSON value>)` | The callback's own return value (`allow`/`deny`/`updatedInput`) |
| **Right for** | "This whole workflow needs a human before continuing" | "This one action needs approval, but the agent can keep reasoning about everything else" |
| **This course's chapter** | Chapter 07 (mechanics), this chapter (design) | This chapter |

Neither is a strict upgrade over the other — they answer genuinely different questions about what needs to pause. A real production system, per Diagram 2, plausibly uses both: `interrupt()` for whole-workflow human checkpoints in a multi-agent LangGraph system, `canUseTool` for fine-grained, per-call approval within a single Claude Agent SDK-based agent.

## Decision Framework — Calibrating Human Oversight

1. **What tier is this specific action, per the four-tier model?** Read-only and reversible actions almost never need a synchronous human gate — routing them through one anyway is the direct cause of this chapter's Production Issue.
2. **If a gate is warranted, which architecture fits?** Synchronous only for the tier where waiting is cheaper than a wrong decision; asynchronous for medium-risk items that shouldn't block other work; parallel feedback as the default for most reversible, production workloads.
3. **What happens if nobody responds in time?** Never leave this unanswered — per this chapter's Beginner Implementation, every action tier needs an explicit TTL and expiry policy, not an assumed indefinite wait.
4. **What autonomy level is this agent actually operating at, honestly?** Use the CSA's six-level framework to name it precisely — and recognize that most current systems, including yours, are very likely L2–L3, not the L4/L5 the marketing copy might imply.
5. **Is your approval rate telling you the truth?** A rate above ~90% is not reassurance — it's the current, confirmed signal that your gates are miscalibrated and your reviewers' attention is likely diluted.

## Real Client Scenario — Closing Aperture Cloud's Incident-Remediation Loop

Chapter 07 left Aperture Cloud's incident-remediation graph with a working `interrupt()` gate and an explicit, unanswered question: what happens if the engineer approving a rollback doesn't respond? This chapter closes it completely. The `high_risk_action_node`'s single `interrupt()` call — correctly scoped to one call per node, avoiding Chapter 07's own Production Issue — now sits behind this chapter's four-tier classifier, so only genuinely high-risk actions (a production rollback, an environment change) reach it at all; routine investigative actions (reading logs, checking ticket status) are auto-approved and logged, never prompted. A TTL expiry job scans every paused thread every fifteen minutes, and an unanswered high-risk approval escalates to the broader on-call rotation after 24 hours rather than waiting silently forever. Every decision — auto-approved, escalated, or expired — lands in an audit trail that lets Aperture Cloud's team measure their actual approval rate over time, catching the exact miscalibration signature this chapter's Production Issue describes before it becomes six months of quiet rubber-stamping.

This is the honest, complete version of "escalate to a human" that every chapter since Chapter 01 has been gesturing toward without building — and it's the last piece of foundational human-oversight infrastructure this course builds before Module 3 is allowed to let the stakes get real.

---

## Exercises

1. **(15 min)** Run the Beginner Implementation's `scan_for_expired_interrupts` against a manually-constructed paused thread older than its TTL, and confirm it correctly returns the expected policy action.
2. **(30 min)** Add a fifth entry to the Intermediate Implementation's `ACTION_REGISTRY`, deliberately omit its tier assignment, and confirm `classify_action` correctly defaults it to `HIGH_RISK_IRREVERSIBLE` rather than any more permissive tier.
3. **(30 min)** Simulate this chapter's Production Issue: route 50 low-tier actions through a synchronous-only gate (no classifier) and track how quickly a simulated "reviewer" (a function that starts returning `True` without inspection after N calls) starts rubber-stamping. Then apply the four-tier classifier and confirm the same 50 actions produce far fewer actual human-facing prompts.
4. **(45 min)** Wire the Advanced Implementation's `can_use_tool_callback` into a real `ClaudeSDKClient` session with at least three tools spanning different risk tiers, and confirm read-only and reversible tools are auto-approved while an external or high-risk tool genuinely triggers the approval prompt path.
5. **(60 min, Challenge)** Research the AWS/Kiro incident independently (starting from the sources this chapter cites) and write a short analysis of your own: does the "misconfigured access control" framing or the "autonomous agent judgment" framing better explain the outage, or is the honest answer that both were necessary and neither alone was sufficient? Justify your reasoning explicitly rather than picking a side by default.

## Quiz

1. **What's the precise difference between human-in-the-loop and human-on-the-loop?**
   *Answer: HITL requires explicit human approval before an action executes — nothing proceeds without a prior yes. HOTL lets the agent act autonomously by default within defined boundaries, with a human monitoring and able to intervene, typically after the fact or when the agent itself escalates.*

2. **Why is "route every action through the same approval gate" not a safety-maximizing default, according to current evidence?**
   *Answer: Past roughly the 300th routine approval, a human reviewer is primed to keep clicking "approve" without genuinely evaluating each one — approval fatigue. A uniformly-gated system dilutes the reviewer's attention across low-stakes requests, making them less likely to catch the one request that actually mattered, which can produce a worse outcome than a well-calibrated, narrower gate.*

3. **What's the confirmed current monitoring signal for detecting approval-gate miscalibration?**
   *Answer: An approval rate consistently above roughly 90% — this indicates the gate's triggers are catching far more than the genuinely risky tail of actions, not that the system is safely reviewed.*

4. **Why does `interrupt()` alone not solve the "what if nobody responds" problem, and what does?**
   *Answer: `interrupt()`, exactly as Chapter 07 built it, waits indefinitely with no built-in timeout. This chapter's TTL expiry job — a scheduled scan checking paused threads' age against a per-action-type policy — is the explicit, required addition that turns "wait forever" into a deliberate, logged policy decision.*

5. **In this chapter's four-tier risk model, what should an unclassified or ambiguous action default to, and why?**
   *Answer: The most-scrutinized tier (high-risk/irreversible), never the least. Uncertainty about an action's actual risk should always resolve toward more oversight, not less — defaulting to a permissive tier for an unknown action risks exactly the kind of unreviewed consequential action this chapter's patterns exist to prevent.*

6. **What's the key architectural difference between LangGraph's `interrupt()` and the Claude Agent SDK's `canUseTool`?**
   *Answer: `interrupt()` pauses an entire multi-agent graph's execution, persisted via a checkpointer, for a human decision. `canUseTool` gates a single tool call for one agent, without pausing anything else about that agent's ongoing reasoning — different scope for the same underlying human-in-the-loop principle.*

7. **Why does this chapter present the AWS/Kiro incident with its causal dispute intact, rather than picking one narrative?**
   *Answer: Both halves of the incident are multi-source corroborated — the outage happened, and the mandatory two-engineer sign-off was bypassed via misconfigured access controls — but AWS's own official statement disputes the "autonomous AI agent" framing multiple outlets used, attributing it to human misconfiguration instead. Presenting the dispute honestly, rather than resolving it in the chapter's own voice, follows this course's established discipline for genuinely contested claims.*

8. **What's the actionable lesson from the AWS/Kiro incident that holds regardless of which causal framing is correct?**
   *Answer: A mandatory approval gate that can be routed around via a permissions misconfiguration provides no real protection at all — the gate itself and the access-control system underneath it must be verified together, not assumed to reinforce each other automatically.*

9. **Why does this chapter recommend routing escalations by reviewer expertise rather than just availability?**
   *Answer: Different categories of risky action require genuinely different expertise to evaluate correctly — a financial compliance officer and a backend engineer are not interchangeable for approving the same class of action. Routing purely by who's available, rather than who's qualified for that specific action category, increases both mistakes and per-reviewer noise.*

10. **According to the CSA's six-level autonomy framework, what's the honest current market reality worth calibrating expectations against?**
    *Answer: Most systems marketed as "autonomous agents" in 2026 are actually operating at L2–L3, still requiring a human at key checkpoints — "nobody is shipping L5" — a useful corrective against assuming a system's marketing language accurately reflects its actual autonomy level.*

## Mini Project

**Build:** A four-tier action classifier and durable, timeout-aware approval gate wired into one of your own agent implementations from an earlier chapter (Chapter 07's remediation graph is a natural candidate).

**Time estimate:** 2.5–3 hours

**Requirements:**
- At least six actions registered across all four risk tiers, with the fail-safe "unclassified defaults to high-risk" rule correctly implemented.
- Gate architecture correctly matched per tier (no gate for read-only, parallel feedback for reversible, async escalation for external, synchronous `interrupt()` for high-risk).
- A TTL expiry scan job with at least two different TTL/policy pairs for different action tiers, demonstrated correctly identifying an expired, unanswered high-risk approval.
- An audit trail logging every classification, gate decision, and (where applicable) reviewer response, with a computed approval rate over your test run.

**Acceptance criteria checklist:**
- [ ] All four risk tiers are represented and correctly route to their designated gate architecture
- [ ] An unregistered action correctly defaults to the high-risk tier, not a permissive one
- [ ] A deliberately-unresolved high-risk approval is correctly caught by the TTL scan and triggers its configured policy
- [ ] The audit trail can answer "what was our approval rate over this run" from logged data alone

## Production Project

**Build:** A complete human-oversight system implementing this chapter's full Production Architecture — tiered classification, the six-signal escalation trigger set, reviewer-expertise routing, and calibration monitoring — layered on top of a Chapter 07-style LangGraph system AND a Claude Agent SDK-based single agent, demonstrating both `interrupt()` and `canUseTool` in the same overall system.

**Time estimate:** 1.5–2 days

**Requirements:**
- The four-tier classifier and TTL expiry job from this chapter's Beginner/Intermediate Implementations, fully wired into a real, checkpointed LangGraph workflow.
- A `canUseTool`-based approval layer for a separate, single Claude Agent SDK agent, using the same underlying risk classification logic (shared code, not two independent classifiers that could drift out of sync).
- At least three of the six confirmed escalation triggers (confidence breach, risk-tier match, SLA proximity, irreversibility flag, sentiment signal, injection signal) implemented and demonstrated firing correctly.
- Reviewer routing that distinguishes at least two reviewer categories by expertise, not just a single flat on-call queue.
- A calibration dashboard (can be a simple script producing a report) computing the rolling approval rate and flagging when it crosses the ~90% miscalibration threshold.
- A short internal README applying the Decision Framework to justify the autonomy level (CSA framework) you're claiming for this system, honestly.

**Acceptance criteria checklist:**
- [ ] Both `interrupt()` and `canUseTool` are demonstrated in the same overall system, sharing the same underlying risk classification
- [ ] At least three escalation triggers fire correctly on constructed test cases
- [ ] Reviewer routing correctly distinguishes at least two expertise categories
- [ ] The calibration report correctly flags a deliberately-induced high approval rate as miscalibrated
- [ ] README's autonomy-level claim is justified against the CSA framework's actual level definitions, not asserted without reference to them

## Key Takeaways

- Every "escalate to a human" reference since Chapter 01 gets its full engineering treatment here — this chapter is the load-bearing foundation the rest of this course's human-oversight material depends on, not optional theory.
- HITL and HOTL are stages on a maturity path, not two permanently-fixed options — current guidance shows organizations actively shifting from HITL toward HOTL as trust in specific agents and task categories grows.
- More human oversight is not automatically safer — past roughly the 300th routine approval, a human reviewer is primed to rubber-stamp, meaning over-gating can produce worse outcomes than well-calibrated, narrower gating.
- The four-tier action-risk model, paired with three gate architectures (synchronous, asynchronous, parallel feedback), replaces uniform "approve everything" policies with oversight matched to actual stakes.
- `interrupt()` alone has no timeout — a durable, TTL-aware expiry layer with an explicit per-tier policy is a required addition, not an edge case, closing the exact question Chapter 07 deliberately left open.
- An approval rate consistently above ~90% is the current, confirmed signal of gate miscalibration — measure it from day one, don't wait to notice fatigue anecdotally.
- LangGraph's `interrupt()` and the Claude Agent SDK's `canUseTool` solve the same underlying problem at genuinely different granularities — whole-workflow pause versus single-tool-call gating — and a real system may need both.
- The CSA's six-level autonomy framework is a real, current, useful tool for honestly naming how autonomous a system actually is — most 2026 systems are L2–L3, regardless of marketing language.
- Never let a model be the sole gate before an irreversible or money-moving action — a deterministic check or genuine human approval must sit behind that tier, full stop.
- A mandatory approval gate provides no real protection if the access controls underneath it can be misconfigured around it — the gate and the access-control layer must be verified together, a lesson this chapter's central AWS/Kiro case study makes concrete regardless of which causal framing you find more persuasive.

## Chapter Summary

| Concept | Key Takeaway |
|---|---|
| HITL vs. HOTL | Explicit pre-approval vs. autonomous-by-default-with-monitoring; a maturity path, not a fixed choice |
| Four-tier risk model | Read-only / reversible / external / high-risk-irreversible, each mapped to a matched gate architecture |
| Gate architectures | Synchronous (block), asynchronous (queue), parallel feedback (act + review) — hybrid per tier, not uniform |
| Approval fatigue | Over-gating dilutes reviewer attention; ~90%+ approval rate is the confirmed miscalibration signal |
| TTL expiry | `interrupt()` waits forever by default; an explicit, tiered timeout/escalation policy is required |
| Escalation design | Six confirmed trigger signals; route by reviewer expertise, not just availability |
| Bounded autonomy levels | CSA's current six-level framework (adapting SAE J3016); most 2026 systems are L2–L3 |
| `interrupt()` vs. `canUseTool` | Whole-graph pause vs. single-tool-call gate — different granularity, same HITL principle |
| AWS/Kiro case study | A gate is only as strong as the access controls around it — presented with its causal dispute intact |

## Resources

- Cloud Security Alliance, *Autonomy Levels for Agentic AI* (January 2026) — the six-level framework this chapter's Bounded Autonomy Levels section builds on
- Gravitee, *State of AI Agent Security* (April 2026) — the 48%-no-governance / 19.7%-full-approval figures behind this chapter's approval-fatigue evidence
- BCG / UC Riverside study on human oversight burden — the 14%/12%/39%/39% figures on mental effort, fatigue, error rate, and attrition under high AI-oversight demand
- arXiv 2606.08919, on "oversight capacity" — the ~300-approval rubber-stamping mechanism cited in this chapter's Core Concepts
- Multi-source reporting on the December 2025 AWS/Kiro incident (Engadget, The Register, PYMNTS, TechRadar, Capacity) and AWS's own official statement (2026-02-21) — this chapter's central, honestly-contested case study
- Chapter 07 (Building Multi-Agent Systems with LangGraph) — the `interrupt()`/`Command(resume=...)`/`PostgresSaver` mechanism this chapter builds its design layer on top of

## Glossary Terms Introduced

| Term | One-line definition |
|---|---|
| Human-in-the-loop (HITL) | Explicit human approval required before a consequential action executes |
| Human-on-the-loop (HOTL) | Autonomous-by-default operation with human monitoring and after-the-fact intervention |
| Four-tier action-risk model | Read-only / reversible / external / high-risk-irreversible classification for calibrating oversight |
| Synchronous gate-keeping | Blocking execution until explicit human approval, for the highest-risk tier |
| Asynchronous escalation | Queuing a decision for review without blocking other work |
| Parallel feedback | Proceeding immediately with review happening alongside, backed by rollback |
| Approval fatigue | Degraded human review quality from excessive or poorly-targeted approval requests |
| Bounded autonomy levels | The CSA's current six-level (L1–L5) framework for calibrating agent autonomy |
| `canUseTool` | The Claude Agent SDK's per-tool-call human-approval callback |

## See Also

| Related Chapter | Why |
|---|---|
| Chapter 01 (Agent Architecture Deep Dive) | Source of the blast-radius and bounded-autonomy vocabulary this chapter's four-tier model directly extends |
| Chapter 05 (Multi-Agent Orchestration Patterns) | The fallback hierarchy's final tier ("escalate to a human"), fully built out here for the first time |
| Chapter 07 (Building Multi-Agent Systems with LangGraph) | Source of the `interrupt()`/`Command(resume=...)`/`PostgresSaver` mechanism this chapter's design layer sits on top of |
| Chapter 09 (Building Agents with the Claude Agent SDK) | Goes deep on hooks specifically, only introduced at the surface level in this chapter's `canUseTool` comparison |
| Chapter 13 (Agent Security) | Full treatment of the excessive-agency and access-control risks this chapter's AWS/Kiro case study previewed |

## Preparation for Next Chapter

**Technical checklist:**
- [ ] You can run this chapter's TTL expiry job and confirm it correctly applies a per-action-tier policy to an expired approval
- [ ] You can explain, without notes, why an approval rate above ~90% is a warning sign rather than reassurance
- [ ] You can state the precise scope difference between `interrupt()` and `canUseTool`

**Conceptual check:** Before Chapter 09, make sure you can answer this: *this chapter used `canUseTool` for human-approval judgment and mentioned hooks only briefly, as the primitive for unconditional guarantees that must hold regardless of permission mode. What's the actual current mechanism for registering a hook, what hook points exist, and how would you build a rule like "never allow a tool call that reads a file containing the word 'credentials,' no matter what canUseTool or the active permission mode says"?* (If your answer is "I don't know the exact current hook API yet, but I understand structurally that it needs to run before or independent of canUseTool's conditional logic, deterministically, every time" — that's exactly the right amount to know before Chapter 09, which goes deep on the Claude Agent SDK's subagents, hooks, and Skills as this course's next full framework deep-dive, the same way Chapter 07 went deep on LangGraph.)

**Optional challenge:** Take this chapter's AWS/Kiro case study and design, on paper, the specific hook(s) and permission configuration that would have prevented the mandatory two-engineer sign-off from being bypassable via a permissions misconfiguration in the first place — not just detecting the bypass after the fact, but making the gate itself structurally un-bypassable. Keep your design; Chapter 09's hook mechanics will let you check it against the real current primitive.
