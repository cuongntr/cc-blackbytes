---
name: routing-discipline
description: >-
  Cost-aware guidance for when to delegate to a subagent versus doing the work
  inline. Use when deciding whether to spawn explore/oracle/librarian/general
  agents, when a task could fan out across independent parts, or when you catch
  yourself about to delegate something finishable in 1–2 direct tool calls.
  Helps avoid both over-delegation (token/latency waste) and under-delegation
  (clogging the main context with work better isolated).
---

# Routing discipline

Delegation is a cost/benefit trade, not a reflex. Each subagent call spins up a fresh context and adds token + latency overhead. The discipline: **delegate to isolate or parallelize, not to avoid thinking.**

## The one rule

If a task is finishable in **1–2 direct tool calls**, do it yourself. Delegation only pays off when the work is large, independent, or benefits from an isolated context or a specialist persona.

## Routing matrix

| Want | Route to | Cost | Use when | Avoid when |
|---|---|---|---|---|
| Find code / trace a flow | **Explore** (native) | Medium | Broad or unfamiliar search, cross-file discovery, "where is X / how does Y work" | A single grep or known-file lookup suffices |
| Deep reasoning / hard call | **oracle** (this plugin) | High | Architecture decision before implementing, a fix you've failed twice, security/perf trade-off, skeptical second opinion | Answerable from local code in a few calls; needs writes/execution |
| External research | **librarian** (this plugin) | High | 3+ external sources to triangulate, current-year authoritative answer, official docs + changelog + real usage together | Single URL fetch, single docs lookup, local-codebase questions |
| Heavy implementation | **general-purpose** (native), or an agent with `isolation: worktree` | High | Well-scoped work across 3+ files after the plan is clear; parallelizable parts | Single-file edits; work needing mid-stream feedback |

## Proactive triggers — delegate without hesitation when

- You need to understand an unfamiliar codebase area → fire 1–3 **Explore** tasks in parallel.
- You've failed a fix twice → **oracle** for elevated debugging (don't keep patching incrementally).
- A complex architecture question stands between you and implementation → **oracle** first.
- The user asks about external library behavior or APIs you're unsure of → **librarian**.
- A task has independent parts that can run simultaneously → fan out multiple agents in one message.

## Cost awareness

- Prefer **fewer, broader** delegations over many narrow ones.
- When you fan out, send all independent agent calls in a **single message** so they run concurrently — don't serialize what has no dependency.
- An expensive agent (oracle, librarian) used for a trivial question is pure waste. Match the agent's cost to the problem's difficulty.
- Delegation is for the orchestrator. A subagent should not try to spawn further subagents — return to the caller instead.

## Self-check before delegating

Ask: *Could I answer this in 1–2 tool calls right now?* If yes, do it. If no, is the win **isolation** (keep my context clean), **parallelism** (independent parts), or **specialism** (a sharper persona)? If none of those three apply, the delegation probably isn't worth it.
