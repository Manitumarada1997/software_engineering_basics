# Why Orchestration?

## What Is It?

**Container orchestration**: a system that runs containers across many machines — deciding *where* they run (scheduling), keeping the *desired number* running (reconciliation), wiring *networking and storage*, and rolling out *new versions safely*. Kubernetes (K8s, 2014, from Google's Borg lineage) is the dominant answer; the concept is bigger than the product.

## Why Does It Exist?

Docker solved "run it anywhere" — on *one host*. The moment you have 5 services × 20 instances × 10 nodes, you face what Google's Borg engineers faced at planet scale:

```text
Who decides which node runs which container (bin-packing, taints, resources)?
What happens when a container dies at 3 AM (restart? where? how fast)?
How do 200 replicas get one stable address (discovery + load balancing)?
How does a new version roll out without downtime (and roll back)?
What if a NODE dies (reschedule everything it ran)?
Secrets, config, storage — for 500 containers?
```

Orchestration answers all of it with **one idea**: you declare desired state; the system continuously *reconciles* actual → desired. That loop (stated in systemd's page, perfected in Borg/K8s) is the intellectual core — everything else in this phase is the API surface of that loop.

## Layer 1 — Simple Explanation

Without an orchestrator, you're the **general contractor on a construction site** with 40 workers, calling each by phone: where to stand, what to carry, when to take breaks, redoing the plan on every absence.

With one, you post **one job board**: "I need 20 bricklayers, 5 plumbers, materials delivered here." The site foreman (K8s) continuously walks the site: who's missing? replace. Who's idle? assign. New blueprint? phase it in. You never speak to individual workers again.

## Layer 2 — Engineer's View

**The reconciliation loop — the pattern to internalize:**

```text
for {
    desired := readSpec()          // what you declared (etcd)
    actual := observe()            // what exists (kubelet reports)
    if actual != desired { act() } // create/kill/move until they match
}
```

Why this beats imperative scripts: it's **self-healing** (node dies → loop reschedules), **declarative** (Git holds truth — GitOps follows), and **eventually convergent** after any disruption. This loop appears again in Terraform (drift → plan) and ArgoCD — one pattern wearing three costumes.

**What "orchestrating" concretely coordinates** (the rest of the phase):

| Concern | K8s answer |
|---|---|
| Where to run | Scheduler (resources, affinity, taints) |
| How many / restarts | Controllers (Deployments, ReplicaSets) |
| Stable address | Services + DNS |
| Config/secrets | ConfigMaps, Secrets |
| Storage | Volumes, PV/PVC, StorageClass |
| Version changes | Rolling updates, rollbacks |
| Policy/security | RBAC, NetworkPolicies, admission |

**The cost ledger (why not everyone should run K8s):** operational complexity (upgrades, control-plane HA, CNI/storage choices), the learning cliff for app teams, and a resource floor (~a node-pair minimum for anything serious). Managed control planes (AKS/EKS/GKE) remove *some* toil; the complexity tax remains. Teams with < a handful of services may be better served by Cloud Run-style platforms (Compute page) — orchestration is for *fleets*.

**The historical arc worth knowing:** Borg (Google internal, ~2003) → Omega → Kubernetes (open-sourced 2014, rewritten generically) → CNCF ecosystem (Platform phase's landscape). Docker Swarm/Mesos lost — not on features but on the API model: K8s' declarative, extensible object model (custom resources!) let the ecosystem build *on* it rather than *beside* it.

## Real-World Example (DevOps flavored)

The 3 AM scenario, with and without:

```text
Without: node dies → 14 containers gone → pager fires → you SSH around,
         restart things by memory, misconfigure one, sleep is over
With:    node dies → scheduler notices (node controller) → pods rescheduled
         to survivors in ~seconds → your pager fires only if SLOs actually dipped
```

That delta — *from runbook heroics to automatic convergence* — is what organizations are really buying when they adopt orchestration. Your role shifts from executing recovery to designing states that recover themselves.

## Common Mistakes

- Adopting K8s for a 3-service startup because it's "the standard" — paying fleet tax without a fleet
- Treating it as a VM replacement ("one giant pod per node") rather than a declaration-and-convergence model
- Skipping the concepts (this phase) for kubectl recipes — cargo-culting YAML without the mental model
- Fighting the reconciliation loop with manual `kubectl edit` (drift; GitOps fixes)

## Mental Model

> The orchestrator is a **self-correcting job board for a robot workforce**: you post desired outcomes, not instructions; the foreperpetually compares site to blueprint and repairs the gap. Kill a robot, lose a truck, post a new blueprint — the site *converges* back without you.

## Remember This

1. Orchestration = scheduling + reconciliation + discovery + rollout + policy across many nodes
2. The core idea is the reconciliation loop: declared desired state vs observed, converged continuously
3. It makes systems self-healing and ops declarative — the foundation GitOps extends
4. K8s won via its extensible declarative object model (Borg lineage, CNCF ecosystem)
5. The complexity tax is real: orchestration pays off at *fleet* scale; smaller = simpler platforms
6. Your job shifts from executing recovery to declaring recoverable states

## One Sentence

Container orchestration turns a fleet of disposable containers into a self-healing system by continuously reconciling observed state against declared desired state — deciding placement, quantity, networking, and rollout without human intervention.

## Knowledge Check

1. Write the reconciliation loop in pseudocode and map each line to a K8s component.
2. Why does declarative + reconcile beat imperative scripts for failure recovery?
3. When is Kubernetes the *wrong* choice? Name three signals.
4. What did K8s' extensibility (CRDs) enable that features alone couldn't?

## Further Reading

- [Kubernetes.io — concepts](https://kubernetes.io/docs/concepts/) (the next pages parallel it)
- *Kubernetes: Up and Running* — Hightower et al., ch. 1–2
- "Borg, Omega, and Kubernetes" — Google ACM paper (the lineage)

---

**← Previous:** [Container Networking & Storage](../containers/networking-storage.md)
**Next:** [Kubernetes Architecture](architecture.md) →
**Related:** [systemd](../linux/systemd.md) (the earlier, smaller reconciler) · [GitOps](../iac/gitops.md)
