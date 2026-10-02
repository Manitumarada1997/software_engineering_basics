# Agentic Systems

## What is it?

The vocabulary layer above agents: an **agentic system** is any production system where LLM-driven decisions coordinate work — from a fixed workflow with one LLM step, to a fully autonomous agent, to teams of them. The spectrum (and knowing where on it to *build*) is the engineering content of this page.

## The autonomy spectrum (pick deliberately, not aspirationally)

| Level | Shape | Control | Use when |
|---|---|---|---|
| LLM step in workflow | fixed pipeline, one model call | max | task is structured, known shape |
| **Agentic workflow** | model chooses paths among defined steps | high | branches depend on content |
| Agent (loop) | model owns steps until goal | medium | open-ended, multi-step |
| Multi-agent | several loops, coordinated | low | genuinely parallel specializations |

The industry's hard-won lesson: **use the lowest level that works**. Every step up trades predictability for flexibility — the same judgment as "modular monolith before microservices" (Monolith page). Most successful "agents" in production are level 2: agentic *workflows* — routing, extraction-with-verification, generate-then-check — with the model choosing among engineered steps, not improvising the whole journey.

## The components of a real agentic system

| Component | From page |
|---|---|
| Tools (validated, least-privilege) | Tool calling |
| Context management (budgets, eviction) | Context engineering |
| Memory (beyond the window) | Memory (ahead) |
| Verification/evaluation (per-step and end-to-end) | Evals |
| Guardrails (injection, PII, moderation) | Guardrails |
| Human-in-the-loop gates | this page |

**Reflection & verification** deserve their line: the pattern of a second pass — the model (or a second model) critiques the draft against criteria before release — buys large quality gains for one extra call. Self-review, output validation, critic models: the "four-eyes principle" for non-deterministic components.

**Human-in-the-loop (HITL)**: the placement question — approve every action (safe, slow), approve *classes* of action (destructive/spend = gate; read-only = free), approve *anomalies* (confidence-thresholded). The right answer is almost always class-based + anomaly-based — exactly how your deployment approvals work.

## The benefits (why agentic at all)

- handles tasks you cannot enumerate as workflows (the rule-writer's wall, again!)
- adapts to evidence mid-task
- converts "many manual steps + judgment" into "supervised automation"
- composes: same agent, new tools, new domain

## The risks (the full ledger — each mapped to its mitigation)

| Risk | Mitigation |
|---|---|
| Hallucinated action (invented tool facts) | verification passes; evals |
| Infinite loops / runaway cost | iteration caps, budget ceilings, watchdogs |
| Wrong action with real effects | least privilege, approval gates, dry-run modes |
| Injection through any external content | labeled channels, tool output validation |
| Data leakage via tools | egress control, per-tool scopes |
| Non-determinism → irreproducibility | full tracing (seeded configs where possible) |
| Debugging difficulty | the Observability page: every step logged, replayable |

## The DevOps mapping

| Agentic concept | Your world |
|---|---|
| Autonomy levels | automation levels — script vs workflow vs self-healing system |
| Approval gates | deployment approvals: class-based + anomaly-based |
| Verification pass | CI check + human review — the four-eyes principle |
| Risk ledger | the Chaos page's discipline: design for failure you expect |

## Remember This

1. Spectrum: LLM-step → agentic workflow → agent → multi-agent; build at the lowest sufficient level
2. Most production value is at "agentic workflow": model picks paths among engineered steps
3. Reflection/verification (second pass) is the cheapest quality multiplier
4. HITL by action class + anomaly — mirror your deployment approvals
5. Every risk has a named mitigation — agentic ≠ reckless when engineered

## One Sentence

Agentic systems put LLM judgment into coordinated, tool-using workflows — gaining adaptability the rules-era could never encode, at the price of control, which is bought back with verification loops, scoped tools, approval gates, and evaluation at every layer.

## Knowledge Check

1. Place on the spectrum: invoice extraction, incident triage assistant, autonomous PR reviewer — defend each.
2. Why is "agentic workflow" the production sweet spot?
3. Design the HITL policy for an agent with read, write, and delete tools.

---

**← Previous:** [Agents](agents.md)
**Next:** [Agent Loops](agent-loops.md) →
