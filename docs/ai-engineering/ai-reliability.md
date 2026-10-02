# AI Reliability

## What is it?

Your SRE discipline (SRE phase) adapted to a stack whose dependencies are *non-deterministic services*: model APIs, retrieval systems, tools, and the model's own variable behavior. The failure modes are new; the patterns are yours.

## The failure catalog

| Class | Examples | First response |
|---|---|---|
| **Provider outages** | API down, rate-limited, region-degraded | fallback provider/model |
| **Degradation** | latency spikes, truncated outputs | timeout, retry, escalate |
| **Quality failures** | hallucination, wrong format — *no error raised* | validation + retry-with-feedback |
| **Tool failures** | downstream 5xx, timeouts | the Fundamentals page's pattern set |
| **Retrieval failures** | empty/wrong chunks | graceful "insufficient context" over confident nonsense |
| **Runaway agents** | loops, cost spirals | caps, watchdogs (Loops page) |

The distinctive engineering: **quality failures don't throw** — a wrong answer arrives, 200 OK. Reliability = making wrongness *detectable* (schemas, validators, judges) and then *retryable* (feed the error back — often self-corrects) and finally *degradable* (safe fallback, human escalation).

## The pattern stack (all four from your existing playbook)

**1. Timeouts and budget hierarchies.** Every model call: connect + total timeout; every agent run: token/dollar/iteration ceilings. The Fundamentals page's rule — each hop's budget inside its caller's — now includes the *user's patience budget*.

**2. Retries with a twist — retry-with-feedback.** Blanket retries amplify (and re-pay for) the same wrong output. The AI-specific pattern: validate output → on failure, retry *with the validation error in context* ("your JSON had severity=null; severity is required") — frequently self-corrects in one pass. Retry storms still apply: backoff + jitter + caps, idempotent tools.

**3. Fallbacks and circuit breakers.** Provider A degraded → circuit trips → route to provider B or a local model (the base-URL-swap architecture paying off). Degrade *capability*, not availability: "only simple queries today" beats a 500.

**4. Graceful degradation ladder** (AI edition):

```text
frontier model → mid-tier → small local → deterministic template
   ("I can't process this now; here's the runbook link")
```

A product that degrades to templates keeps the SLO that matters: *never dead, sometimes dumber*.

## SLOs for AI features (completing the SRE page's promise)

| SLO | Target example |
|---|---|
| Availability | API success rate (classic) |
| Latency | TTFT p95 < 1.5s; total p95 < 8s (Serving page) |
| **Quality** | eval score ≥ threshold on sampled production traffic (Observability page) |
| Cost | unit cost within budget — a reliability SLO for finance |

Error budgets apply beautifully: quality budget spent → freeze model/prompt changes, hardening sprint — the exact release-freeze policy, new domain.

## The DevOps mapping

| AI reliability | Your world |
|---|---|
| Model = flaky dependency | circuit breakers, fallbacks (Fundamentals page) |
| Retry-with-feedback | self-healing scripts / reconcilers |
| Capability degradation | the degradation ladder (Capacity page) |
| Quality SLO + freeze policy | error budgets, verbatim |
| Watchdogs on agent loops | resource limits + job deadlines |

## Remember This

1. Quality failures return 200 OK — detect (validators/judges), retry-with-feedback, degrade
2. Provider fallbacks via base-URL abstraction; degrade capability, not availability
3. Budgets: timeouts, tokens, dollars, iterations — every level
4. AI SLOs: availability, latency, quality, unit cost — with error-budget policy on quality
5. "Never dead, sometimes dumber" is the reliability goal for AI features

## One Sentence

AI reliability applies your distributed-systems pattern set — timeouts, retry discipline, circuit breakers, graceful degradation — to model, tool, and retrieval dependencies, extended with validators that catch the failures which never raise errors and quality SLOs whose budgets gate change.

## Knowledge Check

1. Why do blanket retries fail on quality errors but retry-with-feedback often works?
2. Design the degradation ladder for a summarization product.
3. Write the error-budget policy: quality at 89% vs 92% SLO — what freezes, for how long?

---

**← Previous:** [AI Cost](ai-cost.md)
**Next:** [Fine-tuning](fine-tuning.md) →
