# Monitoring vs Observability

## What Is It?

- **Monitoring**: collecting predefined metrics and alerting on known failure modes — *watching for what you predicted*
- **Observability** (control theory origin): a system's property — **how well you can infer internal state from external outputs** — operationalized as rich telemetry (metrics, logs, traces) that lets you investigate *questions you didn't know you'd have*

```text
Monitoring answers:  "Is the system broken?"        (known-unknowns)
Observability answers: "Why is it behaving like this?" (unknown-unknowns)
```

The three pillars: **metrics** (numbers over time), **logs** (discrete events), **traces** (request journeys across services) — each gets its own page next.

## Why Does It Exist?

Because distributed systems broke monitoring's assumption. On one server, "CPU high" was a diagnosis. Across 40 microservices, the same alert is a riddle:

```text
checkout p95 spiked — which of 40 services? which deploy? which dependency?
which tenant? which code path? — predicted dashboards can't answer unprompted questions
```

Monitoring still matters — it's your pager, your known-failure defense. Observability is the *investigation capability* — the difference between a dashboard wall and the ability to ask any new question of production without shipping new code.

## Layer 1 — Simple Explanation

- **Monitoring** = the **smoke detector**: preset sensors, preset alarm — brilliant at fires, silent about "why does the hallway smell like gas?"
- **Observability** = the **full building telemetry + investigator's kit**: temperature per room, door logs, airflow traces — so any new question ("why is floor 3 cold?") gets answered from data you already collect

The test: *can you answer a question nobody prepared a dashboard for, in minutes, without redeploying?*

## Layer 2 — Engineer's View

**What makes a system observable (the properties, not the products):**

| Property | Meaning |
|---|---|
| **High-cardinality labels** | trace_id, user_id, cart_id on telemetry — averages hide culprits (p95 page) |
| **High-dimensionality** | any slice: service×version×region×endpoint×tenant |
| **Correlated signals** | metrics ↔ logs ↔ traces linked by shared IDs |
| **Structured events** | fields, not prose (Logging page) |
| **Retrievable history** | long-enough retention for patterns, not just incidents |

**The pillars, division of labor (and why you need all three):**

| Pillar | Best at | Bad at |
|---|---|---|
| Metrics | alerting, trends, cheap forever | per-request detail (aggregates only) |
| Logs | detail, context, debugging | search cost, volume |
| Traces | causality *across services*, latency attribution | sampling needed |

**Instrument-first culture (OpenTelemetry — Tracing page):** instrumentation *in the golden path* (Platform phase preview) — every service emits the standard telemetry by default. The alternative — every team hand-rolling logs and begging for dashboards — is the observability poverty most orgs suffer.

**The continuum with the rest of the course:** shift-right (Testing page) required production instrumentation; SLOs (next pages) need user-facing SLIs; canary analysis (CD page) needs cohort-comparable metrics. Observability is the substrate everything downstream stands on.

**Cost honesty:** telemetry is a pipeline you operate — volume, cardinality explosion (a label with unbounded values = your billing incident), retention tiers. Design it like infrastructure, because it is.

## Real-World Example (DevOps flavored)

The question test, live:

```text
14:02  support: "checkout broken for customer X specifically"
Monitoring answer: all dashboards green (her checkout is 1 request in 40k — invisible)
Observability answer: trace_id from her error → span shows payments retry loop
       → logs (same trace_id) show 3DS challenge failing for her BIN range
       → metric slice (card_bins=EU-44xx, svc=payments) shows elevated failures since 13:58
       → correlate to config deploy 13:57 → revert → 8 minutes, one customer's truth found
```

The unprepared question answered from existing telemetry — that's the property you're buying.

## Common Mistakes

- Dashboard walls mistaken for observability — precomputed answers to yesterday's questions
- Unstructured logs ("something went wrong") — prose isn't queryable
- No trace/log/metric correlation IDs — three pillars, no bridges
- Unbounded-cardinality labels as metrics tags (metrics cardinality is expensive — put detail in logs/traces)
- Observability as a team's job rather than a platform property + golden-path default

## Mental Model

> Monitoring is the **smoke detector** — vital, preset, binary. Observability is the **building that records everything and lets you ask it new questions**: why floor 3 is cold, why *this* customer's journey broke — answers assembled from correlated telemetry, not pre-built dashboards. You need the detector *and* the investigative building; only one of them predicts the fire.

## Remember This

1. Monitoring = known-unknowns (alerting); observability = unknown-unknowns (investigation)
2. The test: answer unprepared questions in minutes, without redeploying
3. Three pillars, one correlation layer (shared IDs) — pillars alone aren't the property
4. High cardinality/dimensionality in the right signal: details in logs/traces, aggregates in metrics
5. Instrumentation belongs in the golden path (OTel) — not per-team heroics
6. Telemetry is operable infrastructure: volume, retention, cardinality budgets

## One Sentence

Monitoring alerts on failures you predicted, while observability is the system property — built from correlated metrics, logs, and traces with high-cardinality context — that lets you answer questions about production you never thought to prepare in advance.

## Knowledge Check

1. Why did distributed systems break monitoring's one-server assumptions?
2. Classify: p95 latency, a stack trace, a span tree, an audit event — pillar and best use.
3. Where does unbounded cardinality belong, and where is it a billing incident?
4. Run the question test on your current service: what unprepared question could you NOT answer?

## Further Reading

- Charity Majors / Honeycomb observability writings (the strongest advocacy)
- *Observability Engineering* — Majors, Miranda, Rahl
- Next: [Metrics & Golden Signals](metrics.md)

---

**← Previous:** [Policy as Code & Compliance](../security/policy-as-code.md)
**Next:** [Metrics & Golden Signals](metrics.md) →
**Related:** [Shift Right](../development-practices/shift-left-right.md) · [Logging](logging.md)
