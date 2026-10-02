# Autoscaling & Scheduling

## What Is It?

Two cooperating subsystems:

- **Scheduling** — which node runs each pod (bin-packing with constraints)
- **Autoscaling** — three orthogonal dimensions:

| Dimension | Tool | Question |
|---|---|---|
| Pods | **HPA** (horizontal pod autoscaler) | how many replicas? |
| Size | **VPA** (vertical) | how big should each be? |
| Nodes | **cluster-autoscaler / Karpenter** | how many machines? |

## Why Does It Exist?

The Cloud page's elasticity loop, decomposed for the container fleet: traffic → HPA adds pods → pods Pending (no room) → cluster-autoscaler adds nodes → cloud ASG scales the machine tier. Each layer reacts to a different signal at a different cadence — understanding the *chain* (and its lags) is the skill.

## Layer 1 — Simple Explanation

- **Scheduler** = the **hotel room allocator**: matches guests (pods) to rooms (nodes) by size, group preferences (affinity), and floor restrictions (taints) — one at a time, as guests arrive
- **HPA** = the **front desk calling in staff** as the queue grows (measured by a metric, staffed by replicas)
- **Cluster autoscaler** = **booking more floors** from the building owner when no rooms fit anyone — takes minutes (procurement!)
- **Karpenter** = a **modern property manager**: skips the fixed-floor model, rents exactly-sized rooms directly from the market in seconds

## Layer 2 — Engineer's View

**Scheduling constraints vocabulary (the vocabulary of "why is my pod Pending"):**

| Mechanism | Effect |
|---|---|
| resource requests | the room size the allocator needs |
| nodeSelector / nodeAffinity | floor preferences |
| taints + tolerations | floors that repel guests unless tolerated |
| topology spread | guests distributed across AZs/racks |
| priorityClass + preemption | VIPs may evict economy guests |

`kubectl describe pod` → Events narrates every rejection. Pending pods with no node-fit message = quota or unsatisfiable constraints; "0/N nodes available" enumerates the reasons.

**HPA mechanics (know the two failure modes):**

```yaml
scaleTargetRef: { kind: Deployment, name: payments }
minReplicas: 3
maxReplicas: 30
metrics: [{ type: Resource, resource: { name: cpu, target: { averageUtilization: 60 } } }]
```

- Works on **requests-relative** utilization (no requests = broken HPA — everything connects back to Pods page)
- React interval ~15s + rollout-of-metrics lag: **HPA is reactive** — pre-scale (minReplicas bump) for known events (the Cloud page's flash-sale law)
- *Custom metrics* (queue depth, req/s) usually beat CPU for honest load signals
- **HPA vs VPA conflict**: both fighting the same resource → never run VPA in Auto mode on HPA targets (VPA for stateful/one-off, HPA for scalable)

**Cluster autoscaler vs Karpenter:**

| | CA (classic) | Karpenter |
|---|---|---|
| Model | scales existing node groups | creates right-sized nodes directly |
| Fit | templated families | per-pod requirement aggregation |
| Speed | ASG-bound | ~40s node provisioning |
| Consolidation | limited | aggressive spot/bin-packing |

Either way, remember the chain lag: pod-scale seconds, node-scale ~1–2 min, plus image pull — cold capacity is never instant; design headroom and pre-warming for the peaks that matter.

**Spot in the fleet:** Karpenter/CA spot node pools = the Compute page's economics — with `topologySpreadConstraints` + graceful drain + disruption budgets (PDBs limit concurrent evictions) so spot death doesn't cascade.

## Real-World Example (DevOps flavored)

The "autoscaling doesn't work" triage checklist:

```bash
kubectl get hpa                     # targets unknown? → metrics-server missing / no requests
kubectl describe hpa payments       # "insufficient metrics" = requests unset (the usual)
kubectl top pod -l app=payments     # actual utilization the HPA sees
kubectl get pods -o wide            # scale-up pods Pending? → next layer:
kubectl events --for node/...       # autoscaler/Karpenter: instance capacity, quotas, max nodes
```

The layered answer: requests set → HPA tracks → Karpenter provisions → spot+PDBs keep it affordable and safe. Missing any layer silently breaks the layer above.

## Common Mistakes

- No resource requests: HPA blind, scheduler blind, Pending forever — the root cause of half of all "K8s is broken" tickets
- HPA on CPU for an I/O-bound or queue-driven workload (wrong sensor — Cloud page's thermostat lesson)
- VPA-auto on HPA targets — the controllers fight
- No PodDisruptionBudgets — node upgrades/spot death evict everything at once
- Expecting instant elasticity: cold pods need image pulls + warmup; pre-scale known peaks

## Mental Model

> The scheduler is a **room allocator matching guests to floors by size, preference, and floor rules**; HPA is the **front desk staffing by queue**; the node autoscaler is **procurement renting more floors** (minutes, not seconds). The whole hotel is a chain of lags — the engineer's job is setting the right sensor at each layer and pre-heating the rooms for known rush hours.

## Remember This

1. Scheduling = requests + affinity + taints + spread; Pending-pod Events narrate the failure
2. HPA scales replicas by utilization-vs-requests (custom metrics often better); reactive → pre-scale
3. HPA and VPA don't mix on one target; VPA for the fixed, HPA for the scalable
4. Cluster autoscaler scales groups; Karpenter right-sizes nodes directly — node lag is real either way
5. Spot + PDBs + topology spread = economical without cascading evictions
6. Unset requests break *every* layer above silently

## One Sentence

Scheduling bin-packs constrained pods onto nodes while HPA, VPA, and node autoscalers scale the three dimensions of capacity — replicas, size, and machines — in a chain of feedback loops whose weakest link is usually a missing resource request.

## Knowledge Check

1. Trace the full chain from traffic spike to a new ready pod serving requests, with timestamps per layer.
2. HPA shows "unknown" targets. The two most likely causes?
3. Why do HPA and VPA conflict, mechanically?
4. What does a PodDisruptionBudget protect against, and from what three events?

## Further Reading

- [HPA](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) / [Scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/) — kubernetes.io
- Karpenter docs — the modern node story

---

**← Previous:** [Namespaces, RBAC & Network Policies](rbac-policies.md)
**Next:** [Helm & Operators](helm-operators.md) →
**Related:** [Load Balancing & Autoscaling (cloud)](../cloud/lb-autoscaling.md)
