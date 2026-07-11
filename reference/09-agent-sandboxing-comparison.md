# Reference 09 — Agent Sandboxing Comparison

Quick-lookup companion to [Chapter 10's sandboxing content](../chapters/chapter-10-computer-use-browser-agents.md#core-concepts) and [Chapter 13's security mapping](../chapters/chapter-13-agent-security.md#core-concepts). Use this when choosing an execution sandbox for untrusted or high-capability agent actions — code execution, browser automation, or any tool with real-world side effects.

> **Currency Note:** Verified 2026-07-11, including a fresh re-check of session limits and pricing beyond this repository's original kickoff research. Sandboxing vendor pricing and limits move fast — re-confirm before a production commitment.

## Is Sandboxing Optional? No.

Current guidance treats sandboxing as **"no longer optional... a non-negotiable requirement"** for any agent with browser automation, untrusted code execution, or genuinely consequential real-world capability — not a hardening step to add later. This is one layer within a defense-in-depth stack: process isolation, resource controls, continuous monitoring, and IP-level isolation (a session's IP address is a primary cross-session linking vector worth isolating specifically).

## Isolation Technology Comparison

| Provider | Isolation | Notes |
|---|---|---|
| **E2B** | Firecracker microVM | Kernel-level isolation, each sandbox gets its own kernel |
| **Vercel Sandbox** | Firecracker microVM | Same underlying primitive as E2B, tightly integrated with the Vercel platform |
| **Daytona** | Hardened OCI/Docker containers (host-kernel sharing) by default; optional Kata/Sysbox for stronger, closer-to-microVM isolation | Fastest provisioning (27–90ms) of the group; weaker default isolation than a true microVM unless Kata/Sysbox is explicitly configured |
| **Modal** | gVisor (user-space kernel isolation, not a microVM) | No microVM option for higher-security needs; optimized for Python ML workloads with native GPU support |

**Firecracker microVMs remain the current dominant isolation primitive for untrusted agent code execution** specifically because each sandbox gets its own kernel — a meaningfully stronger boundary than container-based isolation alone.

## Session Limits and Concurrency (verified 2026-07-11)

| Provider | Session limit | Concurrency |
|---|---|---|
| E2B | 1 hour (Hobby) / 24 hours (Pro) | 20 (Hobby) → up to 1,100 (Pro + add-on) |
| Vercel Sandbox | 45 min (Hobby) / 5 hours (Pro/Enterprise) | 10 (Hobby) → up to 2,000 (Pro/Enterprise) |
| Daytona | Effectively indefinite | Custom, enterprise-scoped |
| Modal | Configurable per workload | GPU-capable; 3x CPU cost multiplier for that capability |

## Pricing Snapshot (verified 2026-07-11 — re-confirm before committing)

| Provider | Compute | Notes |
|---|---|---|
| E2B | ~$0.0504/vCPU-hour | Hobby: $100 one-time credit, no card required. Pro: $150/mo, 24-hour sessions |
| Vercel Sandbox | $0.128/active CPU-hour, $0.0106/GB-hr memory, $0.15/GB network | Plus $0.60 per million sandbox creations |
| Daytona | ~$0.0504/vCPU-hour, ~$0.0162/GiB-hour memory | $200 free compute for new users; enterprise-only pricing for larger scale (GPU, volume discounts require sales contact) |
| Modal | GPU billed per second (H100, RTX PRO 6000, etc.) | Volume discounts available |

## Decision Guide

**Choose E2B or Vercel Sandbox when:** you need the strongest kernel-level isolation (Firecracker microVM) for genuinely untrusted code, and can work within their shorter session caps.

**Choose Daytona when:** you need long-running or indefinite sessions and can accept container-level isolation by default — explicitly configure Kata/Sysbox if you need isolation closer to a microVM.

**Choose Modal when:** GPU access on the sandbox side is required — no comparable substitute exists among the other three for this specifically. Accept the 3x CPU cost multiplier as the price of that capability.

**Choose Vercel Sandbox specifically when:** already deployed on Vercel and want tight platform integration — note the current region limitation (`iad1`/US East only as of this course's original research; re-verify before a multi-region requirement).

## Browser Agent Sandboxing — The Specific Case (Chapter 10)

For browser automation specifically, current guidance calls for **disposable, containerized Chromium sessions**: every cookie, credential, cache entry, and locally-stored artifact fully destroyed (not just marked for deletion) on exit, with a fresh session starting from zero every time. This is the browser-specific application of the same Firecracker/microVM discipline above, not a separate standard.

## Common Mistakes

| Mistake | Why it matters |
|---|---|
| Treating container isolation as equivalent to microVM isolation | Container-based sandboxes (Daytona's default) share the host kernel — a real, current, weaker boundary than Firecracker unless explicitly hardened |
| Reusing a browser session across multiple agent tasks | Defeats the entire point of disposable sandboxing — any successful injection persists into the next task |
| Sandboxing compute but not isolating IP address | Confirmed current guidance names session IP as a primary cross-session linking vector — isolate it specifically, not just the compute environment |
| Choosing a provider by price alone without checking isolation technology | A cheaper container-based option is not a like-for-like substitute for a microVM-isolated one when running genuinely untrusted code |

---

*Verified: 2026-07-11*
