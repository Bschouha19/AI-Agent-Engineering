# Reference 04 — MCP vs. A2A Decision Guide

Quick-lookup companion to [Chapter 06's Decision Framework](../chapters/chapter-06-agent-to-agent-protocol.md#decision-framework-mcp-a2a-or-neither). Use this when deciding whether a given integration needs tool-serving (MCP), agent-to-agent communication (A2A), both, or neither yet.

> **Currency Note:** Verified 2026-07-11.

## The One-Line Rule

**"MCP for tools, A2A for peers."**

## The Decision Tree

```
Is the capability a FIXED FUNCTION — input in, output out, no
autonomous judgment about HOW to accomplish the task?
│
└─ YES ──► MCP. This is a tool call, not a peer relationship.

Does it decide HOW to accomplish something based on context,
potentially need clarification, or coordinate with other
agents/services?
│
└─ YES ──► Continue to the next question — this is A2A shaped.

Is this actually crossing a REAL trust boundary — a different
team, a different company, a separately-operated deployment —
or is it still fundamentally YOUR OWN CODE?
│
├─ Still your own code ──► Don't reach for A2A yet. Chapter 05's
│                           in-process supervisor/worker patterns
│                           are almost certainly sufficient.
│
└─ Genuine external boundary ──► A2A is warranted. Confirm the
                                   Agent Card is signed and you
                                   have a pre-established trusted
                                   key before shipping (Chapter 06's
                                   Security Considerations).
```

**Default to MCP.** Current guidance confirms MCP still covers the majority of production agent use cases in 2026. Add A2A only once multi-agent coordination across a real boundary is a genuine, not hypothetical, requirement.

## Side-by-Side

| | MCP | A2A |
|---|---|---|
| **Models** | Agent-to-tool/data | Agent-to-agent (peer) |
| **Right for** | Fixed-function capabilities, no judgment involved | Autonomous peers that decide, clarify, coordinate |
| **Volume covering it in depth** | Volume 2 (MCP Engineering) | This volume, Chapter 06 |
| **Discovery** | Tool schema exposed by the MCP server | Agent Card at `/.well-known/agent-card.json` |
| **Identity/trust model** | Server-side auth (API keys, OAuth) | Agent Card signing (`AgentCardSignature`) + mTLS/OAuth2 |
| **Typical relationship shape** | Client calls a fixed capability | Two decision-makers exchanging a task with a lifecycle |

## The Realistic Production Shape

A single production agent is typically **all three roles at once**:
- An **MCP client** for its own tool access
- An **A2A server** when other agents call it
- An **A2A client** when it delegates to a specialist agent elsewhere

Don't think of MCP and A2A as a single either/or choice for an entire system — the choice is made **per integration point**, and most real systems need both protocols simultaneously for different edges of the same architecture.

## Common Miscategorizations

| Scenario | Naive instinct | Correct categorization |
|---|---|---|
| Calling a narrow, specialized sub-agent that only ever returns one fixed kind of answer with zero judgment | "It's an agent, so A2A" | Actually MCP-shaped — if there's no autonomous judgment, clarification, or coordination happening, it's a tool call regardless of what it's called internally |
| A vendor only exposes a capability via A2A, even though the capability itself is judgment-free | "It should be MCP since it's judgment-free" | Still has to be called via A2A — the *ideal* protocol for the capability's shape doesn't override what the other side actually offers |
| Two teams in the same company, same deployment, coordinating in-process | "Different teams built it, so A2A" | Usually still Chapter 05's in-process patterns — team ownership isn't the same thing as a real trust/process boundary |

## Historical Note: ACP

IBM's Agent Communication Protocol (ACP) **formally merged into A2A** under the Linux Foundation (September 2025). IBM's BeeAI platform now runs on A2A. **Don't present MCP/A2A/ACP as three current, live choices** — it's MCP and A2A; ACP is historical context only.

---

*Verified: 2026-07-11*
