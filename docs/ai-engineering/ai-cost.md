# AI Cost Engineering

## What is it?

The FinOps discipline for token-metered systems: knowing where every token goes, and engineering it downward — model selection, routing, caching, context discipline — without breaking quality. Cost is an NFR (NFRs page), and in AI it's a *metered resource* like bandwidth.

## The cost anatomy (where the money actually goes)

    cost per call = input tokens × input price  +  output tokens × output price
    (output runs 3–5× pricier — sequential generation is expensive work)

The levers, ranked by typical impact:

| Lever | Mechanism | Typical saving |
|---|---|---|
| **Model routing** | cheap model for easy tasks, frontier for hard | 5–20× on the easy volume |
| **Context discipline** | don't paste whole docs; retrieve top-k | 2–5× on RAG features |
| **Prompt/response caching** | identical prompts served from cache | huge on repetitive workloads |
| **Smaller/quantized/local models** | 8B local vs frontier API | per-feature unit economics |
| **Batching non-urgent work** | batch APIs at ~50% price | on offline pipelines |
| **Output discipline** | terse formats (JSON not essays), max-token caps | the output-token premium |
| **Agent loop caps** | iteration/budget ceilings | the runaway-loop insurance |

## The pattern: model routing / cascading (the production default)

```text
request → router
   ├─ classify: trivial (70% of traffic)  → small local model
   ├─ standard (25%)                      → mid-tier API model
   └─ hard (5%)                           → frontier — or cascade: try mid-tier,
                                           verify confidence, escalate on doubt
```

The cascade variant — cheap model answers, a verifier (or the model itself, calibrated) decides whether to escalate — is "tiered storage for tokens". Combined with the OpenAI-shape API everywhere (Running-models page), swapping tiers is config.

## The FinOps loop, AI edition (your existing discipline, new meter)

```text
Inform    — per-feature, per-team token attribution (the AI gateway's audit log!)
Optimize  — the lever table, applied where attribution points
Operate   — budgets + anomaly alerts (the retry loop that cost $400 → page someone)
Unit cost — cost per ticket handled / per extraction / per user session — the scalable metric
```

Showback changes behavior; chargeback changes architecture. The conversations are identical to cloud-FinOps: engineering owns what it can see.

## The tradeoffs to state honestly

- Cheap models fail differently — quality gates (Evals page) *before* cost cuts, never after
- Caching repeated prompts risks stale answers where freshness matters — TTL by feature
- Local models trade capex/ops for zero marginal tokens — the volume crossover math (Open/Closed page)
- Aggressive context trimming degrades grounding — token budget vs answer quality is a measured curve, not a guess

## The DevOps mapping

| AI cost | Your world |
|---|---|
| Tokens = metered unit | bandwidth/compute-hours |
| Routing/cascades | tiered storage, instance right-sizing |
| Prompt caching | CDN semantics |
| Anomaly cost alerts | billing alerts — the 2 AM spend page |
| Unit economics | cost per request/order — the only scalable conversation |

## Remember This

1. Cost = tokens in + tokens out (out at 3–5×); attribution per feature via the gateway
2. Levers ranked: routing > context discipline > caching > model size > batching > caps
3. Routing/cascading is the production default — right model per task tier
4. Quality gates before cost cuts; measure the quality/token curve per feature
5. Unit cost (per ticket/extraction) is the metric that scales the conversation

## One Sentence

AI cost engineering is FinOps with tokens as the metered unit — attributing spend per feature through the gateway and driving it down with routing, caching, and context discipline while evals hold the quality floor.

## Knowledge Check

1. Your RAG feature costs $3k/month; 80% is input tokens. Which two levers first?
2. Design the routing tiers (with escalation) for a support product; estimate the split.
3. When does local inference beat API pricing? Show the crossover arithmetic.

---

**← Previous:** [AI Security](ai-security.md)
**Next:** [AI Reliability](ai-reliability.md) →
