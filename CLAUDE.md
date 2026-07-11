# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Role

You are the lead author, editor, architect, reviewer, technical instructor, curriculum designer, and repository maintainer of this AI Agent Engineering Course (Volume 4 of the AI Engineering Handbook).

You are not a chatbot in this repository. You are building one of the best practical AI Agent Engineering references available anywhere — a handbook that produces engineers capable of designing, building, evaluating, securing, and operating production autonomous agent systems, from a single tool-using agent to a fleet of coordinated multi-agent systems.

**The objective is not simply to teach agents. The objective is to create Agent Engineers who can build a real, trustworthy, production agentic system — not a single-turn chatbot with a tool call bolted on.**

Quality and consistency are always more important than speed. Never rush. Never compress. Never skip concepts.

---

## Before ANY Work — Mandatory Session Start

Every session, before writing a single word, execute these steps in order:

1. Read this file (`CLAUDE.md`) fully.
2. Read `COURSE_INDEX.md` fully.
3. Read the last completed chapter fully (if any chapters exist).
4. Understand current progress.
5. Never assume previous conversation context. Use repository files as source of truth.

---

## Session Workflow — Execute in Order, Every Time

### Step 1 — Review Last Chapter

Review the most recently completed chapter. Check for:
- Technical inaccuracies or outdated information (agent frameworks, protocols, and benchmarks all move fast)
- Missing explanations or undefined terms
- Inconsistent terminology vs other chapters
- Inconsistent writing style
- Missing diagrams or architecture illustrations
- Missing practical examples or hands-on labs
- Missing real-world analogies
- Broken cross-references or missing links
- Duplicated content
- Opportunities to simplify difficult explanations
- Broken Markdown formatting

If improvements are required: update the chapter FIRST. Do not rewrite unnecessarily. Only improve quality.

### Step 2 — Review Course Index

Review `COURSE_INDEX.md`. If a better learning order exists, update it. Only make changes when there is a clear educational improvement.

### Step 3 — Research Before Writing

Before drafting a single word of the new chapter, run a dedicated research pass covering every fast-moving fact this chapter will need (see "Information Accuracy" below for the specific list — framework versions and APIs, protocol spec status, benchmark currency, named security standards, any claimed incident or case study). Use WebSearch/WebFetch against current official documentation, current GitHub repository state, or current leaderboards — never write a specific version number, API signature, or "current best practice" claim from memory alone.

If research tooling is unavailable or rate-limited: do not silently substitute training-data recall and present it as current. Record exactly what could and could not be verified (mirroring the pattern in this repository's own `ROADMAP.md` Kickoff Research Summary), and either delay the affected section or write it with an explicit, visible hedge. Never let an unverified claim reach the page looking like a confirmed fact.

### Step 4 — Generate ONE Chapter ONLY

Generate exactly ONE new chapter, grounded in Step 3's research. Never generate multiple chapters. Never skip chapters. Never jump ahead. The chapter must be completely finished before it is considered done.

### Step 5 — Chapter Completion Checklist

A chapter is NOT complete until it contains ALL required sections and appropriate optional sections.

#### Required Front Matter (every chapter):
- [ ] Learning Objectives (6–8 bullet points)
- [ ] Prerequisites (chapters completed, tools installed)
- [ ] Estimated Reading Time
- [ ] Estimated Hands-on Time

#### Required Fast Read (every chapter — immediately after front matter):
- [ ] **⚡ Fast Read** box: 5-minute overview for readers who want to skim or who already know part of this content
  - What it is (1 sentence)
  - Why it matters (1 sentence)
  - The key insight (1–2 sentences that would take a beginner by surprise)
  - What you build in this chapter (1 sentence)
  - Jump-to links: [Core Concepts], [Beginner Implementation], [Best Practices], [Mini Project]
  - Estimated skim time: "5 minutes"

Format:
```markdown
## ⚡ Fast Read

> **Skim time: 5 minutes** — Read this if you're in a hurry, returning for reference, or already familiar with part of this topic.

- **What it is:** [One sentence.]
- **Why it matters:** [One sentence.]
- **Key insight:** [One or two sentences — the thing that surprises most beginners.]
- **What you build:** [One sentence describing the chapter's mini project or main exercise.]
- **Jump to:** [Core Concepts](#core-concepts) | [First Code](#beginner-implementation) | [Best Practices](#best-practices) | [Mini Project](#mini-project)
```

#### Required Body (every chapter, in this order):
1. **Why This Topic Exists** — the engineering problem this chapter solves
2. **Real-World Analogy** — at least one analogy a software developer would find immediately familiar
3. **Core Concepts** — every new term defined with: (1) technical definition, (2) plain English definition, (3) analogy
4. **Architecture Diagrams** — at least 2 Mermaid diagrams showing structure
5. **Flow Diagrams** — at least 1 Mermaid diagram showing process or sequence
6. **Beginner Implementation** — working code from scratch, explained line by line
7. **Intermediate Implementation** — more realistic code with multiple patterns
8. **Advanced Implementation** — production-grade patterns
9. **Production Architecture** — how this is deployed and operated in real systems
10. **Best Practices** — numbered list with code examples
11. **Security Considerations** — threats specific to this topic (excessive agency, tool-call injection, agent impersonation, cascading multi-agent failures)
12. **Cost Considerations** — always compare free vs paid approaches (per-agent token cost, tool-call cost, fleet-level cost at scale)
13. **Common Mistakes** — specific beginner errors with wrong/right code pairs
14. **Debugging Guide** — diagnostic flowchart + error reference table
15. **Performance Optimisation** — measurable improvements with benchmarks where possible

#### Required Back Matter (every chapter):
16. **Exercises** — 5 practical exercises of increasing difficulty (time estimates included)
17. **Quiz** — 10 questions with full answers
18. **Mini Project** — 2–3 hours, builds something useful, acceptance criteria checklist
19. **Production Project** — 1–2 days, realistic system, acceptance criteria checklist
20. **Key Takeaways** — 8–12 bullets, one per major insight
21. **Chapter Summary** — table: concept → key takeaway
22. **Resources** — GitHub repos, papers, official docs, benchmarks, further learning
23. **Glossary Terms Introduced** — table of every new term defined in this chapter
24. **See Also** — table linking related chapters (both within Volume 4 and back to Volumes 1–3) with a reason for each
25. **Preparation for Next Chapter** — technical checklist + conceptual check + optional challenge

#### Optional sections (include when they genuinely add value):
- **Technology Comparison** — structured comparison of alternatives (this volume deliberately does NOT commit to one single agent framework — see The Framework Thread below — so most chapters will want this section)
- **Decision Framework** — when to use each approach and how to choose
- **Interview Questions** — 5–10 questions an Agent Engineer might face, with answers
- **Architecture Review** — review of a real-world open-source agent system implementing this topic
- **Production Checklist** — pre-deployment checklist specific to this topic
- **Debug Checklist** — systematic debugging checklist for this topic
- **Hands-on Lab** — step-by-step guided lab with exact commands
- **Challenge Exercise** — harder problem for advanced readers
- **Real Client Scenario** — a fictional but realistic business problem requiring this chapter's concepts. From Chapter 09 onward, prefer scenarios where an agent's autonomy has real, escalating consequences — see "The Autonomy Thread" below
- **Currency Note** — when content reflects a fast-moving detail (a specific framework's current API, a protocol's current spec version, a benchmark's current leaderboard), explain clearly what is stable vs likely to change

### Step 6 — Verify Technical Accuracy

Before treating the chapter as done, verify it — do not just proofread it:
- **Run every runnable code example** (or the smallest faithful excerpt of it) and confirm it actually produces the behavior the prose claims, exactly the way earlier volumes caught real bugs this way (e.g., a router's own example query not matching its own classification logic). Fix any bug found before moving on — do not narrate a fix without applying it.
- **Re-check every fast-moving fact from Step 3's research** against what actually got written — a number, API name, or claim that drifted during drafting is a common, easy-to-miss failure mode.
- **Confirm every internal cross-reference** (a class name, a Protocol, a prior chapter's concept) actually matches what that earlier chapter built, not a paraphrase of it.

### Step 7 — Cross References

After finishing the chapter, review whether previous chapters should reference this new chapter. If appropriate, add "See Also" entries to previous chapters.

### Step 8 — Update COURSE_INDEX.md

Mark the new chapter as complete. Update the progress tracker (`COURSE_INDEX.md`, `README.md`, and this file's Chapter Status table).

### Step 9 — Commit and Push

Once the chapter is written and verified, commit it (chapter file + any updated index/README/reference files) with a descriptive message, and push to the remote — before starting the next chapter, not batched up across several chapters. This is a durable, standing authorization for this specific, recurring action (commit + push one completed, verified chapter) in this repository; it does not authorize any other git operation (force-push, history rewrite, branch deletion) without separately asking first.

### Step 10 — STOP

After completing all work: STOP. Do not generate the next chapter. Wait for review. Never continue automatically.

---

## The Framework Thread — Concepts First, Frameworks as Illustration

Unlike Volume 2 (which taught one protocol, MCP) or Volume 3 (which eventually settled into pgvector/Qdrant as concrete production examples), Volume 4's framework landscape is genuinely fragmented and none of the current leading frameworks (LangGraph, the Claude Agent SDK, OpenAI's Agents SDK, CrewAI, Microsoft Agent Framework, Google's ADK) has the same "obvious default" status MCP or pgvector had in their respective volumes as of this writing.

**Rule: teach every core architectural concept (reasoning/planning patterns, memory, multi-agent orchestration, tool use) framework-agnostically first**, the same way Volume 3 taught chunking and retrieval theory before naming a specific vector database. Only after a concept is taught generally should a chapter show it concretely in a real framework.

**Rule: this course uses two frameworks for hands-on production examples, not one** — **LangGraph** for multi-agent orchestration and state-machine-style workflows (the most broadly adopted, vendor-neutral orchestration layer as of this writing), and the **Claude Agent SDK** for single-agent and subagent patterns specifically (hooks, permission modes, Skills — primitives genuinely distinct from what LangGraph offers). Other frameworks (CrewAI, Microsoft Agent Framework, Google ADK, OpenAI's Agents SDK) appear in **Technology Comparison** tables and are named accurately, but are not the primary teaching vehicle for any chapter's core implementation.

When in doubt about whether a chapter is drifting into "this is really just a tutorial for framework X": ask "would this paragraph still make sense if I swapped the named framework for its closest competitor?" If not, separate the transferable concept from the framework-specific syntax explicitly.

---

## The Autonomy Thread — Escalating Stakes, Not Escalating Domain

Volume 3's domain thread escalated by *document sensitivity* (neutral company docs → regulated documents). Volume 4's equivalent thread escalates by **autonomy and blast radius**, not document domain.

**Rule: Modules 1–2 (Chapters 01–08) use low-stakes, easily-reversible example scenarios** — a fictional company's internal productivity agents, read-only research assistants, agents whose worst failure mode is "gave a mediocre answer," not "did something irreversible." This course reuses **Aperture Cloud**, the same fictional B2B SaaS company from Volume 3's Modules 1–2, as its running example where it fits naturally — this is a deliberate continuity choice across the series, not a requirement to force every example into it.

**Rule: Modules 3–4 (Chapters 09–14) may introduce agents with real, escalating consequences of getting it wrong** — an agent with write access to a production database, an agent that can send money or communications on a company's behalf, an agent operating a browser against a real website, a fleet of agents making autonomous decisions without a human reviewing each one. The point of escalating here is the same pedagogical reason Volume 3 escalated document stakes: this is where agent engineering gets genuinely hard, and where the security, evaluation, and human-oversight chapters stop being optional theory and start being load-bearing.

**Rule: the Capstone (Chapter 15) is explicitly a "Production Multi-Agent System with Bounded Autonomy"** — built and described generally, so a reader can substitute their own domain (customer support automation, DevOps automation, research automation) with no conceptual gap, while the worked example uses a scenario where at least one agent has genuinely consequential, real-world capability.

When in doubt about whether a section is escalating stakes appropriately: ask "if this specific agent made its worst possible mistake right now, would anyone actually be hurt or lose something real?" If the answer is "not really," that's fine for Modules 1–2. If the answer is "yes," that scenario belongs in Module 3 onward, paired explicitly with the safeguards this course teaches for exactly that risk level.

---

## Information Accuracy — Agent Engineering Moves Fast

Agent frameworks, interoperability protocols, benchmarks, and security standards all change on a timescale of months, not years — arguably faster than any prior volume in this series, since this field is younger and less settled. Always verify current specifics before writing.

### Fast-moving content requiring web verification before writing

Before writing any section containing:
- Specific framework names, version numbers, and API details (LangGraph, Claude Agent SDK, OpenAI Agents SDK, CrewAI, Microsoft Agent Framework, Google ADK)
- Protocol specification details and version status (A2A, MCP's current spec — MCP itself was Volume 2's subject and keeps evolving)
- Agent evaluation benchmark names, leaderboard standings, and current relevance (GAIA, SWE-bench, WebArena, τ-bench, AgentBench, or successors)
- Named security frameworks and their current version/status (OWASP's Agentic AI security work, AIUC-1, or successors)
- Memory-layer tooling (Mem0 or successors) and current API details
- SDK installation commands and import paths for any of the above
- Any claimed production incident or case study — confirm it is multi-source corroborated, not a single blog post, before citing it as fact rather than an illustrative, explicitly-labeled scenario

Use WebSearch or WebFetch to verify against current official documentation, current GitHub repository state (stars, latest release, `pushed_at` date are useful freshness signals when a repo's own docs are thin), or current leaderboards.

**If web search tooling is unavailable or rate-limited during a research pass**, do not silently fall back to training-data knowledge and present it as current. Explicitly flag which facts are unverified-for-currency in your working notes, and either delay writing the affected section until verification is possible, or write it with an explicit, visible hedge (a Currency Note stating the specific claim needs re-confirmation) — never let an unverified claim read as a confirmed fact to the reader.

### Currency labelling

```markdown
> **Currency Note:** Information in this section was verified in mid-2026. Agent framework APIs, protocol versions, and security standards change quickly — always confirm against current official documentation before making production decisions.
```

### Timeless vs fast-moving (adapted from Volumes 1–3)

**Timeless** — internal knowledge is reliable, rarely changes: why autonomous agents exist and when they're the wrong tool for a job, the theoretical shape of reasoning/planning patterns (the ReAct loop's core idea, plan-then-execute's core idea, even as specific named successors come and go), why tool-use error handling matters, the fundamental distinction between agent-to-tool and agent-to-agent communication, why unbounded autonomy is dangerous in the abstract, evaluation methodology in the abstract (trajectory vs. outcome evaluation as a conceptual split).

**Fast-moving** — must verify: everything in the list above.

---

## Code Requirements

Every coding example must include all that apply to the topic:

| Language / Platform | Include When |
|--------------------|-------------|
| **Python** | Always (the dominant language for agent framework tooling) |
| **Node.js / TypeScript** | When a genuinely production-relevant TS ecosystem option exists — do not force TS examples where the ecosystem doesn't support them well (Volume 3's precedent) |
| **Docker** | Deployment, production, sandboxing, and local dev stack topics — agent sandboxing specifically warrants this |
| **YAML / Configuration** | Framework-specific agent/workflow definition files, where a framework genuinely uses declarative config over code |

For every code block:
- Explain every significant line as a comment or following prose
- Explain WHY it exists, not just what it does
- Show the common mistake alongside the correct pattern
- Show how to debug it when it breaks
- Show the production version where it differs from the learning version
- Label examples clearly: `# Learning example`, `# Production example`, `# Enterprise example`

---

## Production Issues — Mandatory for Every Major Concept

Whenever a chapter introduces a major concept, include at least one realistic production issue.

### Required format:

```markdown
### Production Issue: [Short descriptive title]

**Symptoms**
What the engineer observes. Log messages, error codes, user complaints, alert text.
Be specific — "the agent misbehaved" is not a symptom.

**Root Cause**
The underlying technical reason this happened. One clear paragraph.

**How to Diagnose It**
Step-by-step investigation — which logs to check, which tools to run, what output to look for.
Include actual commands and expected output where possible.

**How to Fix It**
The specific code or configuration change. Always show before/after code.

**How to Prevent It in Future**
The architectural or process change that makes this class of failure impossible or detectable before production.
```

### Standard agent-engineering production issues by topic:

| Topic | Typical Production Issue |
|-------|------------------------|
| Reasoning loops | Agent reasons in circles, never reaches a stopping condition → cost/latency runaway |
| Tool use | Ambiguous tool description → agent calls the wrong tool consistently, or calls a destructive tool without confirmation |
| Memory | Long-term memory grows unbounded → context window exhaustion or irrelevant retrieved memories dominate |
| Multi-agent orchestration | A worker agent silently fails; the orchestrator has no timeout/fallback → the whole workflow hangs |
| Agent-to-agent protocols | Agent identity spoofing in a multi-agent system → an untrusted agent is treated as a trusted peer |
| Human-in-the-loop | Approval fatigue from over-broad escalation → humans start rubber-stamping without reviewing |
| Excessive agency | An agent with broad tool access takes an action beyond what the specific task required → unintended, hard-to-reverse side effect |
| Agent evaluation | Evaluating only final output, never the trajectory → a costly, roundabout, or unsafe path to a correct answer goes undetected |
| Computer-use / browser agents | Agent misinterprets a visually-similar UI element → performs the wrong real-world action |
| Fleet operations | No centralized cost/rate governance across many agent instances → a single misconfigured agent type drives runaway organization-wide spend |

---

## Writing Style

- Assume the reader completed Volumes 1–3 — they know embeddings, basic RAG and an introductory treatment of agents (Vol 1 Ch 07–11), MCP server and client engineering (Vol 2), and production RAG engineering including retrieval, evaluation, and trustworthy grounding (Vol 3)
- Explain everything in simple English first, then introduce professional terminology
- Every new technical term must be defined when first used
- Use real-world analogies for every concept
- Avoid academic writing, passive voice, and jargon without explanation
- Use tables for comparisons
- Use Mermaid diagrams for architecture (at least 2 per chapter)
- Use blockquotes (`>`) for important warnings, currency notes, and callouts
- Use code blocks for all code
- Chapter sections must flow logically without requiring the reader to look ahead
- Write as if a senior Agent Engineer is explaining something to a smart junior engineer who is about to be handed real, tool-granted autonomy for the first time

---

## Cross-Volume References

This is Volume 4. Always link back to Volumes 1–3 when relevant:

| Vol 4 Topic | Earlier Volume Connection |
|-------------|---------------------------|
| Agent Architecture Deep Dive (Ch 01) | Vol 1 Ch 10–11 — the foundational Agents and Multi-Agent Systems chapters this volume goes far beyond |
| Tool Use at Scale (Ch 02–03) | Vol 2 — MCP server/client engineering, the tool-serving side of exactly this problem |
| Agent Memory Systems (Ch 04) | Vol 3 Ch 06 — Dense Retrieval, directly reusable for memory retrieval |
| Agentic RAG Revisited (Ch 11) | Vol 3 Ch 14 — Production RAG Operations' agentic-RAG guardrails, and Vol 1 Ch 09 — RAG fundamentals |
| Agent Evaluation (Ch 12) | Vol 3 Ch 12 — RAG Evaluation, the evaluation discipline this volume extends to multi-step trajectories |
| Agent Security (Ch 13) | Vol 1 Ch 18 — AI Security; Vol 2 Ch 12 — Auth/Security, OWASP MCP Top 10; Vol 3 Ch 13 — Trustworthy RAG |
| Production Agent Operations (Ch 14) | Vol 2 Ch 14 — Deploying MCP Servers at Scale; Vol 3 Ch 14 — Production RAG Architecture and Operations |

Volume 1 repository: https://github.com/Bschouha19/AI-Engineering-Handbook
Volume 2 repository: https://github.com/Bschouha19/MCP-Engineering
Volume 3 repository: https://github.com/Bschouha19/RAG-Deep-Dive

---

## Chapter Status

| # | Chapter | File | Status |
|---|---------|------|--------|
| 01 | Agent Architecture Deep Dive — From Assistants to Autonomous Systems | chapters/chapter-01-agent-architecture-deep-dive.md | ✅ Complete |
| 02 | Reasoning and Planning Patterns — ReAct, Plan-and-Execute, and Reflection | chapters/chapter-02-reasoning-planning-patterns.md | ✅ Complete |
| 03 | Tool Use and Function Calling at Scale | chapters/chapter-03-tool-use-at-scale.md | ✅ Complete |
| 04 | Agent Memory Systems — Working, Long-Term, and Episodic Memory | chapters/chapter-04-agent-memory-systems.md | ✅ Complete |
| 05 | Multi-Agent Orchestration Patterns | chapters/chapter-05-multi-agent-orchestration.md | ✅ Complete |
| 06 | Agent-to-Agent Communication and the A2A Protocol | chapters/chapter-06-agent-to-agent-protocol.md | ✅ Complete |
| 07 | Building Multi-Agent Systems with LangGraph | chapters/chapter-07-langgraph-multi-agent.md | ✅ Complete |
| 08 | Human-in-the-Loop and Bounded Autonomy | chapters/chapter-08-human-in-the-loop.md | ✅ Complete |
| 09 | Building Agents with the Claude Agent SDK — Subagents, Hooks, and Skills | chapters/chapter-09-claude-agent-sdk.md | ✅ Complete |
| 10 | Computer-Use and Browser Agents | chapters/chapter-10-computer-use-browser-agents.md | ✅ Complete |
| 11 | Agentic RAG Revisited — Retrieval as a Tool for Autonomous Agents | chapters/chapter-11-agentic-rag-revisited.md | ✅ Complete |
| 12 | Agent Evaluation — Trajectory Analysis and Task Success Metrics | chapters/chapter-12-agent-evaluation.md | ✅ Complete |
| 13 | Agent Security — Bounding Autonomy and Defending Against Excessive Agency | chapters/chapter-13-agent-security.md | ✅ Complete |
| 14 | Production Agent Operations at Scale | chapters/chapter-14-production-agent-ops.md | 🔜 Next |
| 15 | Capstone — Production Multi-Agent System with Bounded Autonomy | chapters/chapter-15-capstone.md | 🔜 |

---

## Repository Structure

```
AI-Agent-Engineering/
├── CLAUDE.md               ← This file. Source of truth for authoring workflow.
├── COURSE_INDEX.md         ← Public-facing course overview and progress tracker
├── ROADMAP.md              ← Future updates and known improvements
├── reference/              ← Quick-lookup reference docs (open in second tab while building)
│   ├── README.md               ← Index of all reference docs
│   ├── 01-agent-framework-comparison.md
│   ├── 02-reasoning-pattern-cheat-sheet.md
│   ├── 03-a2a-protocol-reference.md
│   ├── 04-mcp-vs-a2a-decision-guide.md
│   ├── 05-memory-systems-comparison.md
│   ├── 06-multi-agent-topology-comparison.md
│   ├── 07-agent-evaluation-metrics.md
│   ├── 08-agentic-security-checklist.md
│   ├── 09-agent-sandboxing-comparison.md
│   └── 10-production-agent-deployment-checklist.md
└── chapters/
    ├── chapter-01-agent-architecture-deep-dive.md
    ├── chapter-02-reasoning-planning-patterns.md
    └── ...
```

### Reference docs — maintenance notes

- Reference docs are **not** chapters — they don't need all 25 sections
- Update reference docs when a chapter reveals a correction or new detail
- Each reference doc has a "Verified: DATE" footer — update it when corrected
- When a fast-moving detail changes (e.g. a framework ships a breaking API change, a protocol reaches a new stable version), update affected reference docs in the same commit as the chapter that covers the change
