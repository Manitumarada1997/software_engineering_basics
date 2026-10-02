# Serving at Scale

## What is it?

Serving models to a *team or product* rather than a user: throughput engines, latency SLOs, capacity math, and the routing layer — the page where your six years of DevOps become direct AI infrastructure leverage.

## The throughput engine (vLLM-class)

The Optimization trio (Inference page) productized:

| Feature | What it gives |
|---|---|
| **Continuous batching** | requests join/leave a live batch — 10–50× tokens/GPU/hour |
| **PagedAttention** | KV cache managed like virtual-memory pages — no fragmentation, long contexts affordable |
| **Tensor parallelism** | one model sharded across N GPUs (the 70B+ path) |
| **OpenAI-compatible API** | clients unchanged across engines |

Deployed as a container (vLLM/TGI images) behind your standard LB/K8s machinery — HPA on queue depth, PDBs, all the K8s-phase discipline. A model server is a stateful-ish, GPU-pinned workload class: special scheduling, general rules.

## The SLOs (adapted from SRE, not reinvented)

| Signal | AI-serving version |
|---|---|
| Latency | time-to-first-token (TTFT — the prefill feel) vs inter-token latency (the streaming feel) — *two different SLOs* |
| Throughput | tokens/GPU/hour — the unit-economics line |
| Saturation | KV-cache utilization, batch occupancy, request queue depth |
| Errors | refusals, context overflows, malformed outputs (the Guardrails page's territory) |

The canary/anomaly questions are yours already: "p95 TTFT spiked after the model swap" is a capacity-and-prefill question; "quality dropped" is an evals question — *different dashboards, different owners* (Observability page later).

## Capacity math (one worked example)

    Team need: 50 concurrent chatters, ~500 tok/s aggregate
    One A100 + 8B fp16 + continuous batching ≈ 2,000–4,000 tok/s
    → one GPU serves it with headroom; N+1 for HA (the Capacity page's law, GPU edition)

Overload behaves classically: queue grows → TTFT SLO burns → shed/load-limit at the gateway. GPUs are too expensive to absorb thundering herds politely — the degradation ladder applies: queue non-urgent, batch small models, route down (next).

## Model routing — the multi-model fleet

The production pattern (Cost page expands): **not one model, a router** —

    easy tasks (classify, extract)  → small local/cheap model
    hard tasks (reason, write)      → frontier model
    fallback                        → secondary provider on outage (Reliability page)

Routing by task classifier or by escalating on failure — "cascade" — is the model-fleet equivalent of tiered storage.

## The DevOps mapping (it *is* DevOps)

| Serving concern | Your existing discipline |
|---|---|
| GPU scheduling | node pools + taints/tolerations for a special hardware class |
| Batching/queueing | connection pooling and backpressure |
| TTFT/ITL SLOs | read/write latency — two metrics, two feels |
| Model upgrades | blue/green + eval gate — the deployment-strategies page, AI edition |
| Cost | tokens/GPU/hour as the unit-economics line (FinOps) |

## Remember This

1. Serving engines = continuous batching + paged KV cache + tensor parallelism, OpenAI-shaped
2. Two latency SLOs: TTFT (prefill) and inter-token (decode) — monitor separately
3. tokens/GPU/hour is the throughput/economics line; N+1 still applies
4. GPU fleets need deliberate shedding — too expensive for polite overload
5. Routing/cascades: right model per task tier, fallback on outage

## One Sentence

Serving at scale is classic infrastructure engineering with new units — continuous batching for throughput, two latency SLOs for the streaming feel, tokens per GPU-hour for economics, and routing across a model fleet.

## Knowledge Check

1. Why are TTFT and inter-token latency separate SLOs with different fixes?
2. Size: 200 concurrent users, 2,000 tok/s aggregate — how many A100s (order of magnitude), and what breaks first at 10×?
3. Design the router: which task classes go to which model tier, and what's the fallback?

---

**← Previous:** [Running Models Locally](running-models-locally.md)
**Next:** [Prompt Engineering](prompt-engineering.md) →
