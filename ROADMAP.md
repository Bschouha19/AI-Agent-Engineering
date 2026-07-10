# AI Agent Engineering — Roadmap

## Volume Scope

This document tracks known future improvements and open questions for Volume 4, not the wider AI Engineering Handbook series (see the series roadmap in the Volume 1–3 repositories for that).

---

## Kickoff Research Summary (2026-07-11)

Before drafting the curriculum, two research passes were run to ground the chapter list in the current (mid-2026) state of the field. One succeeded with strong web verification; one was blocked by a tooling rate limit and produced only unverified, training-data-based background. Both are recorded here so a future session doesn't need to re-derive them, and so the unverified half gets a proper verification pass before the chapters that depend on it are written.

### Verified (frameworks, protocols, architecture — web-confirmed via live GitHub/docs as of 2026-07-11)

- **Claude Agent SDK** (`anthropics/claude-agent-sdk-python`, v0.2.115) — full agent loop, subagents, hooks, permission modes, session resume/fork, Skills. Anthropic also now offers **Managed Agents** (hosted REST API) as a productionized alternative to self-hosting.
- **LangGraph** (v1.2.9, 37k★) — still the dominant graph/state-machine orchestration layer, explicitly positioned around "resilient agents."
- **OpenAI Agents SDK** (v0.18.1) is the confirmed production successor to **OpenAI Swarm**, which is now explicitly educational-only.
- **CrewAI** (55.3k★) — still active for role-based crews.
- **AutoGen/AG2 split confirmed**: `microsoft/autogen` relatively dormant since April 2026; community fork `ag2ai/ag2` is the actively-maintained continuation.
- **Microsoft Agent Framework (MAF)** — Microsoft's 2026 consolidation of AutoGen + Semantic Kernel into one framework, integrated with Microsoft Foundry/Azure OpenAI/GitHub Copilot SDK. This is Microsoft's new flagship; Semantic Kernel's positioning is being absorbed into it.
- **Google ADK** (v2.4.0) — active, tightly integrated with A2A (docs hosted under the `a2aproject` GitHub org).
- **A2A (Agent2Agent) protocol** — donated to the **Linux Foundation**, governed by `a2aproject`, reached **v1.0.0 (March 2026)** and **v1.0.1 (May 2026)**. Mature, multi-vendor-governed standard with official SDKs in six languages, a TCK, gateway, and inspector.
- **MCP has NOT absorbed agent-to-agent communication** — confirmed via the live MCP spec changelog. MCP and A2A remain complementary: MCP = agent-to-tool/data, A2A = agent-to-agent.
- **Agent Name Service (ANS)** — an emerging agent discovery/identity standard from the OWASP GenAI Security Project, not yet dominant, worth a brief mention.
- **Mem0** (60.5k★) — the closest thing to a de facto memory-layer standard for agents.
- **Computer-use/browser agents are now a mature, shipped product category**, not an experimental one — `browser-use` (104k★), plus consumer AI-native browsers (ChatGPT Atlas with Agent Mode, Perplexity Comet, Dia, Microsoft Copilot Edge, open-source BrowserOS).
- **OWASP Top 10 for Agentic Applications (2026)** and **AIUC-1** (aiuc-1.com) are both confirmed, current, named security standards specifically for agentic systems — distinct from any prior volume's single-turn/MCP security content.

### Unverified — needs a re-verification pass before the chapter that depends on it is written

A second research pass (agent evaluation, safety operations, incidents, regulation) hit a hard WebSearch/WebFetch rate limit for the entire session and could only produce **training-data-based background, explicitly not web-verified for mid-2026 currency**. Per CLAUDE.md's own rule ("do not silently fall back to training-data knowledge and present it as current"), none of the following should be written into a chapter as confirmed fact without a fresh, successful research pass first:

- **Agent evaluation benchmarks**: whether GAIA, SWE-bench, WebArena, τ-bench (Sierra), AgentBench, BrowseComp, and GDPval are still current/dominant as of mid-2026, or what has superseded them — relevant to **Chapter 12**.
- **Whether ReAct has been superseded** as the default reasoning pattern, and by what specifically — relevant to **Chapter 02**.
- **Magentic-One's exact current status** (merged into MAF vs. deprecated vs. standalone) — relevant to **Chapter 05/07**.
- **Agent sandboxing current tooling** (Firecracker-based microVMs — Vercel Sandbox, E2B, Modal) — relevant to **Chapter 13**.
- **Fleet/cost governance gateway tooling** (LiteLLM, Portkey, Vercel AI Gateway) for cross-org rate limiting — relevant to **Chapter 14**.
- **Claimed incidents**: a Replit agent production-database-deletion incident (~2025) and an Anthropic-disclosed AI-orchestrated cyberattack campaign (~Nov 2025, sometimes referenced as "GTG-1002") were recalled from training data only — **do not cite either as a verified case study without independent, current confirmation**; if unconfirmed by write time, present any illustrative scenario explicitly as illustrative, the same discipline Volume 3 Chapter 14 used for its own agentic-RAG cost-runaway example.
- **Agent-specific regulatory/liability developments** beyond the general AI regulation already covered in Volume 3 Chapter 13.

**Action for next session**: before writing Chapter 02 (reasoning patterns), Chapter 05/07 (multi-agent orchestration), Chapter 12 (evaluation), Chapter 13 (security/sandboxing), or Chapter 14 (production ops), re-run the blocked research topics above. Do not assume the rate limit is still in effect — check first.

---

## Known Open Questions

- **Repository not yet created on GitHub.** This local folder is initialized as a git repo but has no remote. Confirm the intended repository name/visibility before pushing — `AI-Agent-Engineering` is used as the placeholder name in README.md's series table, matching the naming convention of `RAG-Deep-Dive` and `MCP-Engineering`.
- **Chapter 11 (Agentic RAG Revisited) scope vs. Volume 3 Chapter 14's existing agentic-RAG content**: Volume 3 already covered agentic RAG's cost-runaway risk and enforced-budget guardrails from a *retrieval engineering* lens. Chapter 11 here must go deeper from an *agent engineering* lens (retrieval as one tool among several, agent decides when/whether to retrieve, multi-hop retrieval as part of a larger reasoning loop) without simply repeating Volume 3 Ch14's content — revisit this boundary once both are outlined in detail.
- **Node.js/TypeScript coverage**: following Volume 3's precedent, CLAUDE.md scopes TS examples to genuinely production-relevant cases rather than forcing parity with Python on every chapter. Revisit after a few chapters to confirm this is landing correctly.
- **Coding agents**: deliberately scoped to brief mentions only (folded into Chapter 10 or Technology Comparison callouts), since Volume 7 is planned to cover this domain in full depth. Revisit if that volume's timeline changes.

## Possible Future Additions (Not Yet Scoped)

- A dedicated reference doc comparing agent sandboxing services (Vercel Sandbox, E2B, Modal, Firecracker-based options directly) once Chapter 13's research is verified.
- Coverage of agent fine-tuning for tool selection specifically, if it matures beyond research-stage during this volume's writing (flagged as a "Volume 3 or 5" consideration in Volume 2's own roadmap — confirm whether it belongs here instead).

---

*Last updated: 2026-07-11*
