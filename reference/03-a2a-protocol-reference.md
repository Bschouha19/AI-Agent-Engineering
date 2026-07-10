# Reference 03 — A2A Protocol Reference

Quick-lookup companion to [Chapter 06's Core Concepts and implementations](../chapters/chapter-06-agent-to-agent-protocol.md#core-concepts). Use this when implementing or debugging A2A — the chapter teaches *why* and builds working client/server code, this doc is for looking up mechanics without re-reading it.

> **Currency Note:** Verified 2026-07-11. A2A is an actively-governed Linux Foundation standard — re-confirm spec version and adoption figures against current sources if reading this much later.

## Status at a Glance

- **Spec version**: v1.0 (patch v1.0.1) as of this writing
- **Governance**: Linux Foundation, `a2aproject` org
- **Adoption**: 150+ production organizations (as of April 2026), GA support in Microsoft Copilot Studio, Azure AI Foundry, Amazon Bedrock AgentCore
- **Python SDK**: `pip install a2a-sdk` (`a2aproject/a2a-python`), Python 3.10+
- **Reference examples**: `a2aproject/a2a-samples` (e.g. `samples/python/agents/helloworld`)

## Agent Card

Published at: `https://<base_url>/.well-known/agent-card.json`

| Field | Purpose |
|---|---|
| `name`, `description`, `version` | Basic identity |
| `protocolVersion` | Which A2A spec version this agent implements |
| `url` | Where to send requests |
| `skills` | List of `AgentSkill` objects — what this agent can actually do |
| `capabilities` | `streaming`, `pushNotifications`, `stateTransitionHistory` |
| `defaultInputModes` / `defaultOutputModes` | Supported content types |
| `securitySchemes` | Auth requirements — API keys, HTTP auth, OAuth2, OIDC, mTLS |

**No mandatory registry** — an Agent Card is just a file at a well-known path. This is exactly why signature verification (below) is not optional.

## Task Lifecycle

```
submitted → working → completed
                    → input-required (needs clarification)
                    → failed
                    → canceled
```

## Core Server-Side Classes (`a2a-sdk`, verified against source)

| Class | Role |
|---|---|
| `AgentExecutor` | Abstract base — implement `execute(context, event_queue)` and `cancel(context, event_queue)` |
| `RequestContext` | Incoming request data — `current_task`, `message` |
| `EventQueue` | Where the executor emits status/artifact events |
| `TaskUpdater` | Helper for reporting status transitions and adding result `Artifact`s |
| `TaskState` | Enum — `TASK_STATE_WORKING`, `TASK_STATE_COMPLETED`, etc. |
| `InMemoryTaskStore` | **Learning/dev only** — task state vanishes on restart |
| `DefaultRequestHandler` | Wires an executor + task store into servable routes |

## Core Client-Side Classes (`a2a-sdk`, verified against source)

| Class/function | Role |
|---|---|
| `ClientConfig(streaming=...)` | Configures streaming vs. blocking behavior |
| `create_client(agent=card, client_config=...)` | Constructs a client from a discovered Agent Card |
| `SendMessageRequest(message=...)` | The outbound task request |
| `new_text_message(text_query=..., role=...)` | Builds a message payload |
| `Role.ROLE_USER` | Message sender role enum |

> Exact import module paths (`a2a.client`, `a2a.types`, `a2a.helpers`, `a2a.server.*`) are confirmed accurate as of this writing but should be re-checked against current docs before a production build — module organization can shift between SDK versions even when class names stay stable.

## Security Checklist

1. **Never trust an unsigned Agent Card.** Reject and stop — don't proceed with reduced confidence.
2. **Verify `AgentCardSignature` against a key YOU already trust**, established out-of-band — never a key embedded in the card itself.
3. **Require an idempotency key (nonce) on every task-send request**, with a TTL-expiring server-side store, to prevent replay attacks.
4. **Use short-lived tokens + timestamp verification** alongside nonces — a nonce alone doesn't close a replay within its own valid time window.
5. **Layer defenses**: `AgentCardSignature` + mTLS + certificate pinning + registry notarization where available — no single layer is sufficient alone.
6. **Treat every Artifact a remote agent returns as untrusted content** — same discipline as any tool result since Chapter 01.

## Wiring Into This Course's Agent Protocol

A remote A2A agent, wrapped to satisfy Chapter 01's `Agent` Protocol (`async run(goal) -> AsyncIterator[AgentEvent]`), works with Chapter 05's supervisor dispatch, circuit breaker, and fallback hierarchy with **zero code changes** to the supervisor. The entire cross-boundary complexity — discovery, signature verification, wire protocol — lives inside the wrapper class. See Chapter 06's `RemoteA2ASpecialist` for the full pattern.

## Failure-Mode Quick Diagnosis

| Symptom | Cause |
|---|---|
| Client raises `PermissionError` before sending anything | Correctly-working signature rejection — check the trusted key registration, not the client code |
| Same task executes multiple times | Missing idempotency key — see Chapter 06's Production Issue |
| Supervisor hangs on a remote specialist | `RemoteA2ASpecialist` not wrapped with Chapter 05's timeout — never call it unprotected |
| 404 on discovery | Wrong well-known path or base URL |
| Task stuck at `working` | Server executor missing a final `TASK_STATE_COMPLETED` update |

## Naming Trap: "Agent Name Service" (ANS)

At least three distinct things currently share this name:
1. OWASP GenAI Security Project's original ANS resource
2. A newly-announced Linux Foundation ANS standard (PKI-backed, DNS-inspired)
3. A separate GoDaddy-co-developed IETF draft, also called "Agent Name Service"

A related but distinct project, **DNS-AID**, provides DNS-layer discovery that the Linux Foundation's ANS sits on top of as an identity/verification layer. Treat all of this as "complementary, still-forming identity infrastructure" — don't assert specific technical integration with A2A's own Agent Card discovery beyond that.

---

*Verified: 2026-07-11*
