# Reference 02 — Reasoning Pattern Cheat Sheet

Quick-lookup companion to [Chapter 02's Decision Framework](../chapters/chapter-02-reasoning-planning-patterns.md#decision-framework-choosing-a-reasoning-pattern-by-task-shape). Use this when you're mid-design and need to pick a reasoning pattern fast — the chapter teaches *why*, this doc is for looking up the answer without re-reading it.

> **Currency Note:** Verified 2026-07-11. Pattern names and mechanics are comparatively stable (ReAct and Reflexion date to 2022–2023 papers); the frontier patterns (ToT/GoT) and framework-specific implementation details move faster — re-confirm those against current sources if you're reading this much later.

## The One-Question Router

```
Are the task's steps knowable BEFORE you start?
│
├─ NO  (each step's outcome changes what's next) ──────────► ReAct
│
└─ YES (the whole sequence is knowable up front) ──────────► Plan-and-Execute

Then, regardless of which you picked:

Does the output's correctness depend on more than
"did the loop terminate successfully"?
│
└─ YES ──► Layer Reflection on top of whichever pattern you picked above

Finally:

Does the task have genuinely branching or interdependent
reasoning paths that a single chain or single up-front plan
would force a premature commitment on?
│
└─ YES ──► You need Tree of Thoughts or Graph of Thoughts,
           not (only) the three patterns above
```

## Pattern Quick Reference

| | ReAct | Plan-and-Execute | Reflection |
|---|---|---|---|
| **Decides** | One step at a time | Whole sequence, up front (with replanning) | Nothing about control flow — a quality gate |
| **Best for** | Emergent/investigative tasks | Fully decomposable tasks | Quality-critical outputs |
| **Cost shape** | ~1 model call per step | ~1 planner call + cheap execution + 1 synthesis call | 2x–Nx base cost, proportional to passes |
| **Primary failure mode** | Termination failure — loop divergence, repeated reasoning | Reasoning failure concentrated at the Planner | Termination failure — oscillating critiques between conflicting criteria |
| **Bound to enforce** | `max_iterations` | `max_iterations` on the replan cycle | `max_reflections` |
| **This course's example** | Chapter 02 — ticket investigation | Chapter 02 — weekly engineering health report | Chapter 02 — customer-facing status-page draft |

## Failure-Mode Quick Diagnosis

| Symptom | Pattern | Likely Cause |
|---|---|---|
| Same tool called repeatedly with near-identical arguments | ReAct | Ambiguous tool result (see Chapter 01's Production Issue) |
| Reasoning trace repeats near-identical Thoughts across turns | ReAct | Termination failure — check exit-criteria clarity in the system prompt |
| Plan references data/tools that don't exist | Plan-and-Execute | Planner worked from bad or poisoned input — check what the Planner node actually received |
| One report section silently missing or wrong | Plan-and-Execute | Tool-use failure inside a single executor step, not surfaced | 
| Critique text nearly repeats an earlier round's critique | Reflection | Conflicting Critic criteria with no tie-breaking rule (see Chapter 02's Production Issue) |
| Reflection approves an output with a real flaw | Reflection | Critic prompt too permissive, or draft is adversarially persuasive — see Chapter 13 |

## Composition Rules

- **Patterns nest.** A Plan-and-Execute step that turns out to need step-by-step improvisation can run a ReAct loop internally. Either pattern's output can be passed through Reflection before it ships.
- **Reflection is never a top-level alternative to the other two** — it has no opinion about how the draft it's critiquing was produced. Don't reach for "should I use ReAct or Reflection" as a question; the real question is "which of ReAct/Plan-and-Execute produces the draft, and does the output also need a Reflection pass on top?"
- **Every pattern needs a hard bound.** A "smarter" pattern is not automatically a safer one — Plan-and-Execute's replan cycle and Reflection's revision cycle both need the same enforced iteration cap Chapter 01 established for the generic agentic loop.

## Beyond the Big Three

| Pattern | When | Note |
|---|---|---|
| **Tree of Thoughts (ToT)** | Branching problems needing exploration of multiple candidate paths (math, strategic planning, constrained multi-step planning) | Proposer + evaluator + search policy (beam/BFS) + pruning + stopping condition — cost multiplies by branching factor × depth, bound it |
| **Graph of Thoughts (GoT)** | Problems where combining partial solutions beats picking one best path | Generalizes ToT to an arbitrary graph — branches can merge, not just split — highest cost of the group |

---

*Verified: 2026-07-11*
