# Alerting

## What Is It?

Alerting is the **contract between your telemetry and your humans**: when X is true for Y duration, notify Z *because a specific action is required*. Done right it's the immune system; done wrong, the boy-who-cried-wolf factory that gets muted in month three.

## Why Does It Exist?

Because observation without *response* is decoration (Shift-right page's law). But the deeper reason: **human attention is the scarcest resource in operations** — alert design is attention engineering:

```text
Every non-actionable alert trains the team to ignore alerts.
The ignored channel then misses the real one. That's how outages happen
in fully-instrumented, fully-alerting systems.
```

## Layer 1 — Simple Explanation

Alerts are the **hospital's alarm panel**: each wired to a condition requiring a *specific* intervention (heart alarm → nurse with defibrillator). Hospitals rigorously tune alarms — because alarm fatigue in ICUs measurably kills patients. Same physics: pages at 3 AM that "resolve themselves" teach the on-call to sleep through pages.

## Layer 2 — Engineer's View

**The actionable test — every alert must pass all four:**

```text
1. Is it user-facing? (caused by it or symptom of it — not internal plumbing trivia)
2. Does a human action change the outcome? (else it's a dashboard, not a page)
3. Is it urgent? (minutes matter → page; hours → ticket; never → trend)
4. Is it understood? (runbook exists; recipient knows the first three steps)
```

**Symptom-based vs cause-based — the SRE's sharpest rule:**

```text
❌ cause:  "CPU > 90% on node-3"        (maybe nothing is wrong!)
✅ symptom: "checkout p95 > 800ms for 10m" (users are hurting — the actual question)
```

Alert on **SLIs** (user pain), let dashboards show causes (CPU et al. — the USE panels). Cause-alerts page for real problems with un-instrumented symptoms at best.

**Severity + routing taxonomy:**

| Level | Channel | Meaning |
|---|---|---|
| **Page (critical)** | phone/pager | wake me — user impact now |
| **Ticket (warning)** | queue | fix within days — budget burning (next page) |
| **Trend/info | dashboards | engineering attention, no urgency |

The error-budget era upgrade (next page completes it): *burn-rate alerts* — "budget depleting faster than plan" replaces many raw thresholds, paging on business-agreed pain.

**The mechanics that keep channels sane:**

| Mechanism | Kills |
|---|---|
| `for:` durations | flapping spikes (transient ≠ incident) |
| Alertmanager grouping/inhibition | the 40-alert storm from one node |
| Silences (time-boxed, owned, reason) | known maintenance |
| Runbook links in every alert | the "what do I do" gap — first responder effectiveness |
| Alert postmortems | alerts that fired wrongly/were ignored → tuned or deleted |

**The hygiene metric**: pages per on-call shift (target: < 2 actionable), % alerts acted on, % auto-resolved (noise indicator). Alert review is a standing agenda item — alerts are code, they rot.

## Real-World Example (DevOps flavored)

The alert-cleanup project (the highest-ROI week an SRE team spends):

```text
Before: 140 alerts, ~9 pages/night, 70% "auto-resolved" (noise), team sleeps through pages
Audit:  apply the 4-question test to each
  - 60 cause-alerts → demoted to dashboards
  - 30 flapping → for: durations added / thresholds on rates not spikes
  - 20 non-actionable → deleted
  - 30 symptom/SLI alerts → kept, runbooks attached
After: 4-6 alerts total, ~1 page/night, 100% actionable — and the first real
       3 AM page in weeks got a 5-minute response (the channel was trusted again)
```

## Common Mistakes

- Alerting on causes and internal trivia (disk 85%, cron duration, pod restarts-with-no-symptom)
- No `for:` durations — paging on every transient blip
- Everything critical — the channel that means nothing
- No runbook link — the responder's first action is "ask the chat"
- Never reviewing/deleting alerts — the registry only grows (the SG-archaeology failure mode, pager edition)
- Alerting on averages (Metrics page's law, one last time)

## Mental Model

> Alerting is an **ICU alarm panel tuned against alarm fatigue**: every alarm wired to a condition with a *specific responder and action* (symptoms, not causes; runbook attached). The panel's credibility is the life-saving resource — every false alarm spends it, and a muted panel is worse than none.

## Remember This

1. The actionable test: user-facing, actionable, urgent, understood — all four
2. Symptom-based (SLI) alerts page; causes live on dashboards
3. Page/ticket/trend routing; error-budget burn-rates upgrade threshold design
4. `for:` durations, grouping/inhibition, owned silences — storm suppression
5. Runbook in every alert; alert postmortems tune the system
6. Pages per shift (<2) and action-rate are the health metrics

## One Sentence

Alerting is attention engineering: routing only user-facing, actionable, understood conditions to humans at the right urgency — because the channel's credibility is the resource that makes the real 3 AM page get answered.

## Knowledge Check

1. Apply the four-question test to "node memory 85%" — where does it land and why?
2. Why does a symptom alert page while its cause-alert twin shouldn't?
3. Your team ignores pages. Walk the cleanup: metrics, audit, redesign.
4. What do burn-rate alerts (preview) replace, and what business input do they need?

## Further Reading

- Google SRE book ch. 5-6 (alerting on symptoms, burn rates)
- [Alerting on what matters — Rob Ewaschuk's classic](https://docs.google.com/document/d/199PqyG3UsyXlwieHaqbGiWVa8eMWi8zzAn0YfcW8Yb4/) 
- Next: [SLI, SLO & Error Budgets](slo-error-budgets.md)

---

**← Previous:** [Prometheus & Grafana](prometheus-grafana.md)
**Next:** [SLI, SLO & Error Budgets](slo-error-budgets.md) →
**Related:** [Metrics](metrics.md) · [Incident Management](incident-management.md)
