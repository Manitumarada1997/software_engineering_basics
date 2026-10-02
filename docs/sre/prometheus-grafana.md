# Prometheus & Grafana

## What Is It?

The default metrics stack of the cloud-native world:

- **Prometheus** — pull-based time-series database + query language (PromQL): scrapes `/metrics` endpoints on a schedule, stores with labels, evaluates alert rules
- **Grafana** — visualization: dashboards over Prometheus (and everything else)

```text
targets expose /metrics (text format) → prometheus scrapes (15s) → TSDB
alert rules (PromQL) → Alertmanager → pager            Grafana queries PromQL → panels
```

## Why Does It Exist?

The Soundcloud-born (2015), CNCF-graduated answer to metrics at fleet scale. Design choices you must understand because they explain every quirk:

1. **Pull, not push**: Prometheus decides what to scrape when — service discovery against K8s labels. (Upside: health-check-by-definition, batch targets; trade: NAT-shortlived jobs are awkward → Pushgateway; remote fleets → federation/Thanos)
2. **Labels are the schema**: dimensions first, rows derived — `{service, route, code}` slices ad hoc (Metrics page)
3. **Local TSDB, retention-bounded**: simplicity over durability — long-term/global = Thanos/Mimir layer
4. **Alerting as code**: rules in config → reviewed, versioned (the whole course's pattern, telemetry edition)

## Layer 1 — Simple Explanation

Prometheus is the **census taker**: rounds every household (target) on schedule, records the numbers on standardized forms (metrics format), files them by label-cabinets (the TSDB) — and rings a bell (Alertmanager) when a rule trips. Grafana is the **wall of charts** drawn live from the census archive — any slice, any time range, any combination.

## Layer 2 — Engineer's View

**PromQL — the language of one-liner insight (learn it; it compounds):**

```promql
# error ratio per service (the SLI)
sum by (service)(rate(http_requests_total{code=~"5.."}[5m]))
/ sum by (service)(rate(http_requests_total[5m]))

# p99 latency (histogram — Metrics page's warning: aggregate buckets, not quantiles)
histogram_quantile(0.99, sum by (le,service)(rate(http_request_duration_seconds_bucket[5m])))

# saturation: connection pool
max by (job)(db_pool_in_use / db_pool_max)
```

The `[5m]` ranges + `rate()`/`quantile` pattern is 80% of production queries.

**Exposition — where metrics come from:**

```text
client libraries (app code) — counters/histograms with meaning
exporters — translators: node_exporter (USE method's data!), postgres/mysqld, blackbox (probes)
kubelet/cAdvisor — cgroup accounting (the cgroups page, scraped!)
```

**The K8s-native reality**: ServiceMonitor/PodMonitor CRDs (the Operator pattern — Helm page) declare scrape targets as YAML — observability as code, GitOps-able.

**Alertmanager — the routing brain:**

```yaml
route: { receiver: team-payments, group_by: [alertname, service], group_wait: 30s }
receivers: [ { slack: team-channel }, { pagerduty: { severity: critical-only } } ]
inhibition: { source: NodeDown, target: [.*HighLatency] }   # don't page the storm's victims
```

Grouping, dedup, **inhibition** (the "node down silences its pods' alerts" rule — alert-storm suppression), silences (time-boxed, owned — Alerting page).

**The scale ceiling — know the escape hatches:** single Prometheus = per-cluster visibility. Global/long-term: **Thanos/Mimir** (sidecar-upload to object storage, global query view), **VictoriaMetrics** (drop-in, efficient) — the same data model, bigger shoe.

## Real-World Example (DevOps flavored)

The standard deployment you'll build (or inherit):

```yaml
# kube-prometheus-stack (helm) — the whole SRE substrate in one chart:
prometheus (scrape all, 30d retention) + alertmanager (routing above)
grafana (dashboards as code — ConfigMaps in Git, not hand-clicked panels)
node-exporter (USE) + kube-state-metrics (object states, not just resources)
# + the golden dashboards per service (Metrics page's RED/USE panels) — templated,
#   so a new service appears on dashboards by label, not by copy-paste
```

The cultural rule that makes it real: **dashboards in Git** — a hand-crafted panel is a snowflake (Drift page, visualization edition).

## Common Mistakes

- Pushing metrics to Prometheus (it's pull — use the exporter/pushgateway patterns)
- Cardinally-explosive labels (Metrics page's billing incident)
- Missing exporters: node down, no USE data — flying blind on saturation
- Alert rules on raw counters (`requests_total > 1000`?!) instead of rates/ratios
- No inhibition/grouping — one node failure pages 40 alerts (Alerting page's storm)
- Hand-built Grafana panels — unversioned, unreproducible

## Mental Model

> Prometheus is the **census bureau**: scheduled rounds, standardized forms, label-cabinets, and rule-bells wired to the pager desk (Alertmanager). Grafana is the **cartographer**, drawing any map the archive supports. The bureau's single-city scope is real — federation (Thanos) builds the national archive.

## Remember This

1. Pull-based, label-schema'd, retention-local TSDB — design explains every quirk
2. PromQL: rate/ratio/quantile over ranges — the one-liner insight language
3. Metrics from libraries + exporters + cgroups-scrapes; ServiceMonitors as code
4. Alertmanager = routing: grouping, inhibition, silences — the storm suppressors
5. Long-term/global = Thanos/Mimir/VictoriaMetrics on the same model
6. Dashboards as code (Git) — hand-panels are snowflakes

## One Sentence

Prometheus scrapes, stores, and alert-evaluates labeled time-series with PromQL, Grafana visualizes them — together the default observability substrate of cloud-native systems, with scaling layers (Thanos/Mimir) extending the same model fleet-wide.

## Knowledge Check

1. Why does pull-based design make target health implicit? What breaks it?
2. Write the query: "p95 latency for route /pay, by version, last 5m."
3. Which Alertmanager feature prevents one node's death paging forty alerts?
4. When do you reach for Thanos, and what stays unchanged?

## Further Reading

- [prometheus.io docs](https://prometheus.io/docs/) — best-in-class
- Next: [Alerting](alerting.md)

---

**← Previous:** [Tracing & OpenTelemetry](tracing-otel.md)
**Next:** [Alerting](alerting.md) →
**Related:** [Metrics & Golden Signals](metrics.md) · [Logs & Troubleshooting](../linux/logs-troubleshooting.md)
