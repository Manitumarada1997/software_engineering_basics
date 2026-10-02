# AI Observability

## What is it?

Your SRE observability discipline, extended with the signals AI systems add: **traces of every model/tool step, token and cost telemetry, and quality metrics**. Traditional APM is insufficient because AI adds two new failure classes: *quality degradation* (outputs wrong without errors) and *cost explosion* (perfectly healthy latency, catastrophic bill).

## The telemetry, layer by layer

| Signal | What to capture | Why it's new |
|---|---|---|
| **Model calls** | prompt version, model version, tokens in/out, latency (TTFT + decode), finish reason | non-deterministic black box — version correlation is everything |
| **Cost** | tokens × price per call, per feature, per team | the metered-resource problem (FinOps) |
| **Tool calls** | tool, args (redacted), result status, duration | actions, not just reads — audit-grade |
| **Agent steps** | the loop trace: reasoning → action → observation per iteration | multi-step failures need step-level forensics |
| **Retrieval quality** | which chunks retrieved, clicked-through citations | RAG failures hide here |
| **Quality (online)** | judge scores, user signals (thumbs, edits, escalations) | the "correctness" signal classic APM lacks |

## The trace (the single most valuable artifact)

An AI trace: one user request → every model call, tool invocation, retrieval, and branch, with timings, tokens, and versions:

```text
trace: support-agent-8812
 ├─ model[claude-4] plan: 400 tok, 0.8s, prompt v12
 ├─ tool[search_runbooks] "payments OOM" → 3 chunks (0.2s)
 ├─ model reason: 250 tok, 0.6s
 ├─ tool[kubectl_get] pods payments → 1 result (0.9s)
 └─ model final: 600 tok, 2.1s — finish: stop
 total: 1,250 tok, $0.012, 4.6s
```

With this, "the agent behaved weird at 14:00" becomes a replayable forensic timeline — the incident-management page's scribe, automated. **OTel spans cover this today** (GenAI semantic conventions) — the observability substrate you already operate, extended.

## Why traditional monitoring is insufficient (the two new dashboards)

1. **Quality dashboard**: judge scores + user-signal trends per feature; alert on *drift* (a model or prompt upgrade silently degrading answers — regression detection, not anomaly detection)
2. **Cost dashboard**: tokens per feature/user cohort, anomalous spend (the retry loop that cost $400 — your anomaly-billing alert, AI edition)

Plus the SLO question: what's the AI feature's SLO? Latency (TTFT/total) *plus* a quality threshold — a feature can be fast, available, and wrong. Quality enters the SLO sheet (SLO page) as a first-class metric.

## The failure catalog this enables

| Symptom | Where the trace answers it |
|---|---|
| "Answers got worse" | prompt/model version diff + judge-score trend |
| "It did the wrong action" | agent-step trace: which observation misled the reasoning |
| "Bill tripled" | per-feature token trend; find the loop |
| "Users hate it" | escalation/edit-rate correlation with specific trace patterns |

## The DevOps mapping (you are the most qualified person in the room)

| AI observability | Your world |
|---|---|
| Model-call tracing | distributed tracing (OTel) — same substrate |
| Prompt/model version correlation | release correlation in dashboards |
| Quality drift alerts | SLO burn alerts — regression, not anomaly |
| Cost anomalies | billing anomaly detection |
| Step-level replay | incident timelines — the scribe, automated |

## Remember This

1. AI adds two signals classic APM lacks: quality (drift) and token-cost telemetry
2. The full trace — model + tool + retrieval + steps, versioned — is the forensic artifact
3. Prompt version and model version on every span; correlation is everything
4. Quality belongs in SLOs: fast + available + wrong is a failing service
5. OTel GenAI conventions exist — your existing stack extends, not replaces

## One Sentence

AI observability extends your SRE practice with model-call tracing, token-cost telemetry, and quality-drift monitoring — making every AI failure replayable, attributable, and priced, on the same OTel substrate you already run.

## Knowledge Check

1. Design the trace schema for your first AI feature; which fields earn their storage cost?
2. A prompt change ships; complaints rise Thursday. Which two dashboards answer it?
3. Write the quality SLO for a support agent (metric, threshold, window).

---

**← Previous:** [AI Architecture](ai-architecture.md)
**Next:** [AI Security](ai-security.md) →
