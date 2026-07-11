# Reference 08 — Agentic Security Checklist

Quick-lookup companion to [Chapter 13's Core Concepts and Architecture Diagrams](../chapters/chapter-13-agent-security.md#core-concepts). Use this as a pre-launch review — excessive agency, sandboxing, agent identity — not as a substitute for reading the chapter's reasoning the first time through.

> **Currency Note:** Verified 2026-07-11. OWASP's Top 10 for Agentic Applications and AIUC-1 both update on real, current cadences — re-check version/date before citing a specific control number in a production review.

## OWASP Top 10 for Agentic Applications 2026 — Quick Map

Announced 2025-12-09 by the OWASP GenAI Security Project. Seven of ten categories are substantially addressed by primitives this course built *before* Chapter 13 — security is mostly composition, not a separate toolkit.

| ID | Category | Addressed by |
|---|---|---|
| ASI01 | Agent Goal Hijack | Ch01 bounded autonomy, Ch08 four-tier gating |
| ASI02 | Tool Misuse & Exploitation | Ch03 least-privilege, Ch09 `AgentDefinition.tools` |
| ASI03 | Identity & Privilege Abuse | **Ch13** — SPIFFE-style per-instance identity + Ch09 authorization |
| ASI04 | Agentic Supply Chain Vulnerabilities | **Ch13** — Vercel/Context.ai case study |
| ASI05 | Unexpected Code Execution | Ch09 hooks, Ch10 sandboxing |
| ASI06 | Memory & Context Poisoning | Ch04 memory, Ch10/Ch11 untrusted-content discipline |
| ASI07 | Insecure Inter-Agent Communication | Ch06 A2A protocol |
| ASI08 | Cascading Failures | Ch05 fallback hierarchy |
| ASI09 | Human-Agent Trust Exploitation | Ch08 HITL, approval fatigue |
| ASI10 | Rogue Agents | **Ch13** — Moltbook case study |

Verify exact item wording against the primary PDF (linked from `genai.owasp.org`) before quoting a specific item verbatim in a compliance document.

## Identity vs. Authorization — Don't Conflate Them

- **Identity** = which specific agent *instance* is this (authentication). Current standard: **SPIFFE/SPIRE**.
- **Authorization** = what that identity is allowed to do. Mechanism: Ch09's `AgentDefinition.tools` allowlists and hooks.
- **Check identity first, separately from authorization**, at the `PreToolUse` hook stage — a hook denying an invalid/revoked identity holds unconditionally, independent of permission mode or `canUseTool`.
- **Instance-scoped revocation is the concrete test**: can you disable one compromised agent instance without disabling every other instance of the same type? If not, per-instance identity doesn't functionally exist yet.

## Excessive Agency — Three Structurally Different Mechanisms

Don't apply one fix to all three — they require genuinely different remediations.

| Mechanism | Real incident (this course) | Fix |
|---|---|---|
| Permission misconfiguration | AWS/Kiro (Ch08) — mandatory two-engineer sign-off bypassed via access-control misconfiguration | Verify the gate AND the access controls underneath it together, never assume one guarantees the other |
| Indirect content injection | Comet/WebPromptTrap (Ch10) — hidden instructions in rendered page content | Treat all fetched content (webpages, retrieved documents) as data, never instructions |
| Supply-chain credential compromise | Vercel/Context.ai (Ch13) — third-party tool's compromised OAuth token | Credential scoping, rotation, minimize third-party dependency blast radius |

## AIUC-1 Status

Real, current, quarterly-updated certification standard ("the world's reference standard for AI agent security and reliability," per its own publisher). Q2 2026 update (2026-04-15) modified 14 requirements, added 23 controls; focus areas: MCP/A2A protocol security, agent identity/access management, third-party risk monitoring. UiPath was the first named enterprise-automation-platform adopter (March 2026). **Neither OWASP's ASI list nor AIUC-1 names the Claude Agent SDK or LangGraph specifically** — both speak in framework-agnostic control language.

## Pre-Launch Checklist

- [ ] Every agent instance has a unique, individually-revocable identity — not a shared credential across a fleet or agent type
- [ ] Identity is checked in a `PreToolUse` hook, before any tool-specific authorization logic
- [ ] Every tool allowlist is explicit — no `AgentDefinition` with an omitted `tools` field (which silently inherits the parent's full access)
- [ ] Retrieved content, webpage content, and any external tool output is treated as data, never as instructions
- [ ] A domain/resource allowlist for any browser or external-fetch capability is enforced in a hook, not a system prompt
- [ ] Human approval for a high-risk action is bound to the specific identity that requested it, re-verified at execution time — not just "was this action type approved"
- [ ] Sandboxing (disposable containers/microVMs) is in place for any browser automation or untrusted code execution — see reference doc 09
- [ ] Fleet-wide cost/rate governance exists and is *enforced*, not just monitored — see reference doc 10
- [ ] No internal incentive structure (leaderboard, usage-based recognition) rewards raw agent-usage volume

## Real Incidents Referenced Across This Course — Don't Repeat Them

| Incident | Root cause | Chapter |
|---|---|---|
| AWS/Kiro (Dec 2025) | Two-engineer sign-off bypassed via access-control misconfiguration | Ch08 |
| Perplexity Comet / WebPromptTrap | Indirect prompt injection via hidden page content | Ch10 |
| Vercel/Context.ai (Apr 2026) | Third-party tool's OAuth token compromised via malware | Ch13 |
| Moltbook (Jan 2026, ~1.5M agents) | No per-instance identity — one credential exposure enabled platform-wide impersonation | Ch13 |

---

*Verified: 2026-07-11*
