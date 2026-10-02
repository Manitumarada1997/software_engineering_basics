# Harnesses — The System Around the Model

## What is it?

The **harness** is the complete engineering system wrapped around a model: context management, tool registry, loop/graph control, memory, permissions, guardrails, logging, evaluation, recovery. The term exists to make one boundary unmistakable:

```text
MODEL    — the artifact: frozen weights, provider-supplied, non-deterministic
HARNESS  — everything else: deterministic, yours, versioned, tested
```

**LLM ≠ application.** The model is one component. When a system succeeds or fails, the harness is where most of the difference lives — and it is 100% engineering you already know how to do.

## Why the distinction matters

Teams that blur the boundary experience AI as magic that sometimes disappoints ("the model is unreliable"). Teams that respect it engineer reliability: the model's non-determinism is *contained* the way a flaky network is contained — with retries, validation, fallbacks, and observability around it. Your distributed-systems instincts (fallacies, patterns) were preparation for exactly this: **the model is another eventually-flaky dependency.**

## The harness component checklist (each = a page you've read)

| Component | Responsibility | Page |
|---|---|---|
| Context manager | budget, selection, compression, eviction | Context engineering |
| Tool registry | schemas, least privilege, execution, audit | Tool calling |
| Controller | loop/graph execution, caps, checkpoints | Loops/Graphs |
| Memory | persistence beyond the window | Memory (next) |
| Guardrails | injection, PII, moderation, egress | Guardrails |
| Evaluator | per-step verification + end-to-end evals | Evals |
| Observability | traces of every call/tool/step | AI Observability |
| Recovery | retries, fallback models, human escalation | AI Reliability |
| Permissions | who/what may act — policy as code | Tool calling/Security |

Build this list once as a platform (the Platform Engineering phase's pattern — the harness *is* the golden path for AI features), and every AI application inherits correctness by default. Coding assistants, IDE agents, and "agent products" are all harnesses with different audiences — evaluating them means evaluating the harness, not just the model.

## The two failure modes of harness-less systems

1. **The thin wrapper**: model + prompt + hope — every failure mode (loops, injections, cost blowouts, unparseable outputs) arrives in production, unmitigated
2. **The tangled harness**: control logic smeared through application code — unreproducible, unauditable. Cure: the harness as an explicit, versioned component with its own tests

## The DevOps mapping (this page *is* the thesis of the whole track)

| Harness | Your world |
|---|---|
| Model as flaky dependency | circuit breakers, timeouts — the Fundamentals page |
| Harness as platform | the IDP — golden paths; applications inherit correctness |
| Thin wrapper | `kubectl expose` as architecture — works until it doesn't |
| Harness versioning | the artifact discipline — models AND harness versioned together |

## Remember This

1. Model = frozen artifact; harness = your deterministic engineering; LLM ≠ application
2. Treat the model as an eventually-flaky dependency — contain it with patterns you own
3. The component checklist is nine boxes — each a prior page, assembled into a platform
4. Build the harness once as the golden path; every AI feature inherits it
5. Evaluating any "AI product" = evaluating its harness

## One Sentence

The harness is the deterministic engineering system around the model — context, tools, control, memory, guardrails, evaluation, and observability — and treating it as an explicit, versioned platform is what converts a sometimes-brilliant model into a reliable product.

## Knowledge Check

1. Audit any AI tool you use daily against the nine-component checklist. What's missing?
2. Why is "the model is a flaky dependency" the single most valuable reframe for a DevOps engineer?
3. Which components belong in the platform (shared) vs the application (per-feature)?

---

**← Previous:** [Graphs](graphs.md)
**Next:** [Memory](memory.md) →
