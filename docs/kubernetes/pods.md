# Pods

## What Is It?

A Pod is the **smallest deployable unit** in Kubernetes: one or more containers that share:

- **one network namespace** — same IP, same port space, `localhost` between them
- **shared volumes** — common storage mounts
- **a lifecycle** — scheduled together, died together, rescheduled together

Why not manage containers directly? Because real units of deployment are often *groups*: app + log shipper + sidecar proxy (the mesh pattern). The Pod is the group abstraction — the "co-located co-fated" bundle.

```yaml
apiVersion: v1
kind: Pod
metadata: { name: payments }
spec:
  containers:
  - name: app
    image: reg/shop/payments:2.1.0        # never :latest (Versioning page)
    resources:
      requests: { cpu: 250m, memory: 512Mi }   # scheduling + weight (cgroups)
      limits:   { cpu: "1",  memory: 512Mi }   # ceilings (throttle / OOMKill)
    livenessProbe:  { httpGet: { path: /healthz, port: 8080 } }
    readinessProbe: { httpGet: { path: /ready,   port: 8080 } }
```

## Why Does It Exist?

Two design problems the Pod solves:

1. **Grouping**: sidecars/proxies/shippers must live *exactly* with their app (same node, same fate). Docker-compose-on-a-server did this; K8s needed it as a schedulable object
2. **The atomic scheduling unit**: resources are requested *per pod* (sum of containers) — the scheduler bin-packs pods, not containers

Google's Borg had the same insight ("job/task"); K8s generalized it as the Pod. (Historical quirk: "pod" — a pod of whales, since containers-in-same-netns was originally "containers as whales, Docker's logo".)

## Layer 1 — Simple Explanation

A Pod is a **shared office room**: everyone inside shares the address (IP), the phone line (ports), the whiteboard (volumes) — and the room's rental contract (lifecycle). The building (node) can evict/relocate the *room*, never one occupant alone.

The house rule that follows: **offices hold people who must be together**. Don't put the database in the payments room "to save space" — they scale, fail, and update differently. One process-family per Pod.

## Layer 2 — Engineer's View

**Probes — your declaration of health (the kubelet blind-spot fixer):**

| Probe | Answers | Failure action |
|---|---|---|
| **liveness** | is the app *stuck/deadlocked*? | restart container |
| **readiness** | can it *serve traffic* (deps ok)? | remove from Service endpoints |
| **startup** | still initializing (slow boot)? | pause other probes |

The critical design discipline: readiness = "safe to receive a request" (DB connected? warmed?), liveness = "hopelessly wedged". A liveness probe that fails on dependency trouble causes **restart storms** (cascading restart, the LB page's thermostat pathology, K8s edition).

**Resources — requests/limits map to cgroup semantics exactly:**

```text
requests → cpu.weight / memory floor for scheduling; guaranteed (Guaranteed/Burstable QoS)
limits   → cpu.max (throttle!) / memory.max (OOMKill = exit 137)
```

- Pods with requests=limits everywhere = **Guaranteed QoS** — last evicted under node pressure
- No requests = **BestEffort** — first evicted, first throttled. Your "mysterious restarts" during node memory pressure are the eviction ranking working as designed

**Sidecars — the pattern that justifies Pods** (and its sunset):

```yaml
containers:
- name: app ...
- name: envoy          # mesh proxy: same netns → intercepts app traffic
  image: envoy...
# also classic: log shippers, TLS terminators, adapters
```

(K8s 1.28+ native sidecar containers fix their lifecycle quirk — init-before-app, die-after — that service meshes had hacked around.)

**Pod lifecycle states worth fluency:** Pending (scheduled? image pull?) → Running (≠ ready!) → Succeeded/Failed (terminal; pods don't "restart", *containers* do — pod restart = new pod for Jobs) → Evicted (node pressure). `kubectl get pod -w` + `describe`'s Events are the narrative.

**Ephemerality — the zen of pods:** a pod is cattle (Cloud page). You should be emotionally prepared to `kubectl delete pod <anything>` — that's a feature check for your architecture (stateless discipline: no local state, graceful TERM handling (Signals!), probes honest).

## Real-World Example (DevOps flavored)

The two probe-tuning incidents every team lives once:

```text
1. Readiness = liveness on a DB-dependent app; DB blips → all pods "unready" → then
   liveness fails too → mass restarts during the DB outage → thundering herd on recovery
   Fix: readiness reflects dependency; liveness only checks self (deadlock), with
   generous failureThreshold + terminationGracePeriodSeconds

2. JVM with limits=1.5Gi, no requests; node pressure → BestEffort eviction roulette
   Fix: requests=limits (Guaranteed), MaxRAMPercentage=70 inside
```

## Common Mistakes

- Multiple unrelated containers per pod (scale/fail/update coupling abuse)
- `latest` images + no imagePullPolicy thought — unreproducible pods
- Liveness probes testing dependencies — restart cascades
- No requests ("scheduler will figure it out" — it can't; Pending pods and evictions follow)
- Treating Running as Ready; no readiness gates on dep
- Writing state to pod filesystem and wondering where it went

## Mental Model

> A Pod is a **shared office room**: one address, one phone line, one rental contract for everyone inside. The scheduler books *rooms*; kubelet watches *rooms*; probes are the room's honesty about whether its occupants can actually work. And offices are disposable — never store anything you'd cry about in the wallpaper.

## Remember This

1. Pod = co-located, co-fated container group sharing netns/volumes/lifecycle — the scheduling unit
2. One process-family per pod; sidecars are the legitimate multi-container case
3. liveness = wedged (restart) · readiness = can serve (gate traffic) · startup = booting
4. requests/limits = cgroup weight/ceilings; QoS class = eviction ranking
5. Pods are cattle: no local state, TERM-honoring, honestly probed
6. Containers restart; pods don't (Jobs) — lifecycle vocabulary matters for debugging

## One Sentence

A Pod is Kubernetes' atom — a group of containers sharing one network, storage, and fate, scheduled and health-managed as a unit, with probes and resource requests as its contract with the platform.

## Knowledge Check

1. Why must a sidecar be a *second container in the pod* rather than a second pod?
2. Design liveness vs readiness for an app whose DB is flaky — what does each check?
3. Your BestEffort pods keep dying on busy nodes. Mechanism? Fix?
4. What's the actual difference between "container restarted" and "pod restarted"?

## Further Reading

- [Pods — kubernetes.io](https://kubernetes.io/docs/concepts/workloads/pods/)
- Next: [Deployments & ReplicaSets](deployments.md)

---

**← Previous:** [Kubernetes Architecture](architecture.md)
**Next:** [Deployments & ReplicaSets](deployments.md) →
**Related:** [cgroups](../linux/cgroups.md) · [Namespaces](../linux/namespaces.md)
