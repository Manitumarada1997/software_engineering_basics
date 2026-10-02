# Metrics & the Four Golden Signals

## What Is It?

A **metric** is a named number over time with a small set of labels — the cheapest, most alertable telemetry:

```text
http_requests_total{service="payments", code="500", route="/pay"}   ← counter
request_duration_seconds{service="payments"}                        ← histogram (quantiles)
```

The **four golden signals** (Google SRE) — the minimum health view of any service:

| Signal | Question | Alerts on |
|---|---|---|
| **Latency** | how long do requests take? | p95/p99 (never averages — you know why) |
| **Traffic** | how much demand? | rps, concurrent users |
| **Errors** | how many fail? | rate (5xx + *slow* successes!) |
| **Saturation** | how full are the resources? | queue depth, CPU/mem/disk/connection pools |

## Why Does It Exist?

Metrics compress reality into cheap, comparable, alertable series — *aggregates by design* (cardinality costs — Observability page) — enabling the two things logs/traces can't: **alerting in real time** and **trend analysis over months**. The golden signals exist because they're the minimal set answering "is this service healthy *for its users*, right now" — before you know what's wrong.

The four signals also encode the SRE mindset: latency counts *failed-but-slow* requests as problems; saturation is the *leading* indicator (traffic+latency rising = saturation tomorrow).

## Layer 1 — Simple Explanation

A doctor's **vital signs**: pulse (traffic), blood pressure (latency), temperature (errors), oxygen saturation (saturation) — four numbers, checked continuously, that say "something's wrong" long before diagnosis. Diagnosis needs the full kit (logs, traces — the observability building); vitals are what the pager watches.

## Layer 2 — Engineer's View

**The metric types — semantics matter:**

| Type | Example | Averaging rule |
|---|---|---|
| **Counter** | requests_total, errors_total | only `rate()` — meaningless raw |
| **Gauge** | queue_depth, memory_used | last value |
| **Histogram** | request duration buckets | **percentiles** (p95/p99) via buckets |
| Summary | client-side quantiles | precomputed, not aggregatable (avoid) |

The histogram-vs-summary distinction matters fleet-wide: only histograms aggregate across instances (a p99 of p99s is a lie; buckets sum correctly).

**Latency must be measured at success *and* failure separately** — the classic blind spot: errors return fast, dragging the *average* down while user pain spikes. `rate(errors[5m])` and `p99(success)` are separate signals.

**The USE and RED checklists (two complementary dashboards):**

```text
RED (per service, user-facing):  Rate · Errors · Duration        ← golden-signal view
USE (per resource, capacity):    Utilization · Saturation · Errors ← the Linux page's method, metricized
```

USE finds exhausted resources; RED finds unhappy services. Incidents usually need both — "payments RED is red (errors)" + "db connections USE is red (saturated)" = the story.

**PromQL thinking (the query language of the metrics world — full page next):**

```promql
sum(rate(http_requests_total{service="payments",code=~"5.."}[5m]))
  / sum(rate(http_requests_total{service="payments"}[5m]))     # error ratio — an SLI!
histogram_quantile(0.99, sum by (le)(rate(request_duration_seconds_bucket{service="payments"}[5m])))
```

**Labels discipline:** service, route, status, version, region — *bounded* cardinality (the Observability page's budget). Per-user/trace details go to logs/traces, never metric labels.

## Real-World Example (DevOps flavored)

The golden-signals dashboard every ShopEasy service gets by default (golden-path metrics, from the Pod's probes to the SLO):

```yaml
# standard panels per service (template, not per-team effort):
- Traffic:  sum(rate(http_requests_total{svc="$s"}[5m])) by (route)
- Errors:   error ratio (query above), alert > 1% for 5m → SLO budget burn (later page)
- Latency:  p50/p95/p99 by route; alert: p99 > 800ms for 10m
- Saturation: per-dependency pool use, queue depth, CPU throttled-rate (cgroups page!)
```

And the incident the four saved: traffic flat, errors flat, **latency p99 climbing + connection-pool saturation rising** → leading indicators caught a leak 40 minutes before errors would have paged.

## Common Mistakes

- Averaging latency (the Testing page's law, now operational)
- Counting only failed statuses — slow successes are user pain
- Summary quantiles across instances (mathematically invalid aggregation)
- Unbounded label cardinality (user_id on a metric = the billing incident)
- Dashboards without saturation — the leading signal missing, pages arrive late

## Mental Model

> Metrics are the **vital-signs monitors at every bedside**: cheap, continuous, comparable across the whole hospital; the four golden signals are the standard panel — pulse, pressure, temperature, oxygen. They tell you *which patient* is in trouble now; the observability building tells you *why*.

## Remember This

1. Golden signals: latency, traffic, errors, saturation — minimal user-facing health
2. Counter→rate, histogram→percentiles; never averages, never Summary-quantile aggregation
3. Latency split by success/error; slow-success is an incident
4. RED per service (user view) + USE per resource (capacity view) — incidents need both
5. Bounded label cardinality: detail belongs in logs/traces
6. Saturation is the leading indicator — dashboards without it page late

## One Sentence

Metrics are cheap aggregate time-series that power alerting and trend analysis, with the four golden signals — latency, traffic, errors, saturation — as the minimal panel telling you a service is hurting before you know why.

## Knowledge Check

1. Why is a fleet p99 computed from per-instance p99s a lie, and what's the fix?
2. Where do the golden signals and USE overlap, and where do they diverge?
3. Your p99 is flat but users complain of slowness — three metric hypotheses.
4. Which signal is *leading*, and why does its absence cost you 40 minutes?

## Further Reading

- Google SRE book ch. 6 — the four golden signals (source)
- Next: [Logging](logging.md) · [Prometheus & Grafana](prometheus-grafana.md)

---

**← Previous:** [Monitoring vs Observability](observability.md)
**Next:** [Logging](logging.md) →
**Related:** [Logs & Troubleshooting (Linux)](../linux/logs-troubleshooting.md) · [Alerting](alerting.md)
