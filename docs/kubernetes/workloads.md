# StatefulSets, DaemonSets, Jobs & CronJobs

## What Is It?

The specialized workload controllers — each drops a different assumption the Deployment makes:

| Controller | Assumption dropped | Gets instead |
|---|---|---|
| **StatefulSet** | pods are interchangeable | stable identity, storage, ordered lifecycle |
| **DaemonSet** | pods go wherever scheduled | one pod per (selected) node, always |
| **Job** | pods run forever | run-to-completion with retries |
| **CronJob** | — | Job on a schedule |

## Why Does It Exist?

Deployments assume disposable identical replicas (cattle). Real fleets contain:

- **Identity-needing state** (databases, brokers): replica-0 must be findable *by name*, keep its disk, and bootstrap in order → StatefulSet
- **Node-scoped agents** (log shippers, CNI daemons, monitoring probes): must run *on every node*, everywhere, always → DaemonSet
- **Finite work** (migrations, batch, ML): must complete, retry on failure, not "restart forever" → Job

## Layer 1 — Simple Explanation

- **StatefulSet**: a **numbered locker room** — players keep their name (payments-0), their locker (PV), and their position (order) no matter how often the team re-forms
- **DaemonSet**: the **janitor assigned to every floor** — buildings can't lack one; new floor built → janitor appears automatically
- **Job**: a **contractor with a completion clause** — paid when done, re-attempted if failed, never "restarted" indefinitely
- **CronJob**: the same contractor on a **standing weekly appointment**

## Layer 2 — Engineer's View

**StatefulSet specifics (what "stable identity" buys):**

```yaml
serviceName: payments-db           # headless Service → per-pod DNS (Services page!)
podManagementPolicy: OrderedReady  # 0,1,2... (vs Parallel)
volumeClaimTemplates:              # payments-db-data-payments-db-0 — per-replica PVs
```

- Pod DNS: `payments-db-0.payments-db.ns.svc.cluster.local` — peers find *the primary* by name (the bootstrapping trick: everyone tries to join 0)
- Rolling updates go 2→0 (highest first), and `partition`/canary semantics exist for controlled DB upgrades
- The honest caveat: StatefulSets give you the *mechanism* for stateful operation, not the *operations*. Managed databases (Storage page) or operators (next page) still do the HA/failover/backup work most teams underestimate

**DaemonSet specifics:**

- Tolerations for control-plane taints decide whether system daemons run there
- Rolling update per node with maxUnavailable — your logging agent's outage strategy
- The platform pattern: node exporters, Cilium, Fluentd, Falco — *infrastructure as pods*

**Job/CronJob specifics — the completion contract:**

```yaml
backoffLimit: 4            # retries before Failed
completions: 1 / parallelism: 1
activeDeadlineSeconds: 3600   # kill switch
# CronJob: concurrencyPolicy: Forbid|Replace|Allow ← the overlap decision
```

- Job pods get restarted *until success count reached* — that's the semantic difference from Deployments (restart *toward completion*, not *toward presence*)
- CronJob pitfalls you've hit: timezone (server TZ vs UTC — set it explicitly), `Forbid` for non-concurrent jobs (the "two migrations racing" incident), startingDeadlineSeconds after controller downtime, and idempotency — *the job must be safe to run twice* (at-least-once semantics)

**The picking-order heuristic (write it on the wall):**

```text
Deployment (default) → StatefulSet (identity/state) → DaemonSet (per-node) → Job (finite)
And always ask first: does a *managed service* make this whole problem disappear?
```

## Real-World Example (DevOps flavored)

ShopEasy's mixed estate:

```yaml
# Kafka (self-managed era): StatefulSet, 3 brokers, per-broker PVs, headless svc
# Node monitoring: DaemonSet node-exporter (tolerations: all system taints)
# Nightly cleanup: CronJob 0 2 * * *, Forbid, idempotent (DELETE ... WHERE expired)
# DB migrations: Job run per-deploy (helm hook: pre-upgrade), backoffLimit 0, 
#   fail fast → pipeline blocks → expand/contract discipline enforced
```

The migration-Job-as-pipeline-gate pattern connects CD to K8s: schema change succeeds → rollout proceeds; fails → deploy stops before any pod runs new code.

## Common Mistakes

- StatefulSets for things that don't need identity (heavier, slower updates — just use a Deployment)
- CronJobs without `Forbid` on non-idempotent work — the overlapping-run data mess
- Expecting StatefulSet to make a single-node database HA — it makes it *reschedulable*, not redundant
- DaemonSets without update strategy — agents frozen on old versions for months
- Jobs without deadline/backoff limits — hung jobs leaking pods and CPU for weeks

## Mental Model

> Deployment herds anonymous cattle; **StatefulSet** runs a **numbered team with lockers** (stable name + disk + order), **DaemonSet** stations a **janitor on every floor**, and **Job/CronJob** hires **contractors with completion clauses**. Pick the labor contract that matches the work — and check whether a managed vendor does the whole job before you hire at all.

## Remember This

1. Each controller drops one Deployment assumption: interchangeability, anywhere-placement, forever-running
2. StatefulSet: per-pod DNS + PV templates + ordering — the mechanism for state, not the ops
3. DaemonSet: node-scoped agents with tolerations; the platform-infrastructure pattern
4. Jobs: completion-with-retries; CronJobs need Forbid + idempotency (at-least-once)
5. Migration-Jobs as CD gates enforce expand/contract schema discipline
6. Always ask "managed service?" before StatefulSet-ing a database

## One Sentence

StatefulSets give pods stable identity and storage, DaemonSets guarantee one pod per node, and Jobs/CronJobs run finite retryable work — the workload controllers matching K8s' reconciliation to the actual shape of each task.

## Knowledge Check

1. How does per-pod DNS enable database bootstrapping in a StatefulSet?
2. Why must CronJob handlers be idempotent — what delivery semantics does K8s offer?
3. Design the guardrails for a production data-migration Job invoked by CD.
4. Why doesn't a StatefulSet with 1 replica count as HA?

## Further Reading

- [Workload controllers — kubernetes.io](https://kubernetes.io/docs/concepts/workloads/)
- Next: [Namespaces, RBAC & Network Policies](rbac-policies.md)

---

**← Previous:** [ConfigMaps, Secrets & Volumes](config-secrets.md)
**Next:** [Namespaces, RBAC & Network Policies](rbac-policies.md) →
**Related:** [Cloud IAM](../cloud/iam.md) · [Deployment Strategies](../cicd/deployment-strategies.md)
