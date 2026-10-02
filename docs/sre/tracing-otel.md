# Distributed Tracing & OpenTelemetry

## What Is It?

- **Distributed tracing**: recording one request's **journey across services** as a tree of **spans** (timed operations with context), reassembled by shared **trace_id**:

```text
trace a1b2c3 (1,240ms total)
├─ gateway: 1,230ms
│  ├─ auth-svc: 12ms
│  └─ checkout-svc: 1,210ms
│     ├─ payments-svc: 840ms  ←── the culprit's neighborhood
│     │  ├─ 3ds-check (external): 790ms
│     │  └─ db-write: 45ms
│     └─ inventory-svc: 360ms (parallel)
```

- **OpenTelemetry (OTel)**: the vendor-neutral standard — APIs/SDKs/protocol (OTLP) for emitting traces, metrics, *and* logs with one instrumentation — the CNCF's answer to instrumentation fragmentation.

## Why Does It Exist?

Because latency in distributed systems is a **causality problem** metrics can't solve: p99 is high — but *whose p99*? The request's path crosses 8 services; only the span tree shows that 790 of the 1,240ms lived in one external 3DS call. Tracing attributes latency to *mechanism*, not just location.

OTel exists because per-vendor instrumentation (write for Jaeger, rewrite for Datadog...) was lock-in on the *telemetry layer* — the industry standardized the emitter, leaving backends competing on storage/analysis (the OCI story's observability replay).

## Layer 1 — Simple Explanation

The trace is the **receipt for a package delivery**: every scan station (service) timestamps its handling — and when the package arrives late, the receipt shows it sat in the Rotterdam depot for two days, not "somewhere in Europe."

Context propagation is the **barcode scanned at every station** (trace_id riding the headers of every hop) — lose it and the journey becomes disconnected anecdotes.

## Layer 2 — Engineer's View

**Mechanics worth knowing:**

| Concept | Role |
|---|---|
| **Span** | one operation: name, start/end, tags, status |
| **Context propagation** | W3C traceparent header — the barcode across HTTP/gRPC/queues |
| **Sampling** | you can't afford 100% — head-based (decide at start) vs **tail-based** (keep the interesting: errors + slow) |
| **Baggage** | key-values traveling with context (tenant, experiment) |

Tail-based sampling is the operational sophistication worth the effort: keep *all* the p99 traces, 1% of the rest — your error budget's evidence base.

**The debugging superpower — latency attribution:**

```text
"p99 checkout slow" → sample slow traces → span trees show:
  60%: 3ds external latency (open new provider?)
  30%: db-write on shard-3 (from spans' db attributes → slow query)
  10%: retry storms (spans named retry → backoff bug)
```

One analysis, three distinct root causes — this is why traces outrank dashboards for "why."

**OpenTelemetry as platform strategy:**

```yaml
# the operator auto-instruments (no per-app effort — the golden-path move):
env:
- { name: OTEL_EXPORTER_OTLP_ENDPOINT, value: collector.observability:4317 }
- { name: OTEL_SERVICE_NAME, valueFrom: { fieldRef: { fieldPath: metadata.labels['app'] } } }
# collector: sample, enrich (k8s metadata), route (backend), and REDUCE cost
```

The **OTel Collector** is the spine: receivers → processors (sampling, redaction of PII!) → exporters — swap vendors by config, not by re-instrumentation. Trace-log correlation: logs carrying trace_id (Logging page) = click from span to exact log lines.

**Cross-async tracing** — the hard part worth knowing exists: context through *message queues* (propagate in message headers) — or your "journey" breaks exactly where modern architectures are most interesting.

## Real-World Example (DevOps flavored)

The quarterly performance review powered by traces:

```text
Query: checkout traces > 1s, group by dominant span
Finding: payments 3ds-call = 62% of checkout latency p95
Action: parallelize 3ds with risk-precheck; cache low-risk decisions
Result: p95 1,240ms → 710ms — measured by the same traces (before/after trees)
```

Traces aren't only for incidents — they're the *evidence base* for optimization investment.

## Common Mistakes

- Instrumenting without propagation (spans that never connect — trees of stumps)
- 100% head sampling at any scale — or 0.1% random (never catching the rare-bad)
- Spans without attributes (db statement hash, peer) — trees without diagnostic leaves
- Per-vendor SDKs baked into apps (lock-in OTel retired)
- Breaking context at the queue boundary — the journey's most valuable leg unrecorded

## Mental Model

> A trace is the **package's scan-by-scan receipt**: every station timed, the barcode (trace context) carried throughout, so "late" resolves to *which depot, how long, why*. OTel is the **standard barcode** every station and every courier agrees on — backends compete on filing systems, not on barcodes.

## Remember This

1. Traces attribute latency to causality across services — the "why is p99 high" instrument
2. Context propagation (W3C headers) is the make-or-break; queues need explicit care
3. Tail-based sampling: keep errors + slow traces — the interesting minority
4. OTel = vendor-neutral emitter + collector spine; swap backends by config
5. Collector does sampling, PII redaction, routing — the telemetry pipeline's brain
6. Traces justify optimization investment with before/after evidence

## One Sentence

Distributed tracing reconstructs a request's cross-service journey as a correlated span tree — attributing latency to causes — with OpenTelemetry as the vendor-neutral standard for emitting and routing that evidence.

## Knowledge Check

1. Why can't metrics alone resolve "p99 is slow" in an 8-service chain?
2. Explain tail-based sampling's value for error budgets.
3. Where does context propagation commonly break in event-driven systems, and what's the fix?
4. Your org wants to switch tracing vendors — what does OTel make this a config change?

## Further Reading

- [opentelemetry.io docs](https://opentelemetry.io/docs/) · W3C Trace Context spec
- Next: [Prometheus & Grafana](prometheus-grafana.md)

---

**← Previous:** [Logging](logging.md)
**Next:** [Prometheus & Grafana](prometheus-grafana.md) →
**Related:** [Observability](observability.md) · [Metrics](metrics.md)
