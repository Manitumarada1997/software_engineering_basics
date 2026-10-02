# Compute Options

## What Is It?

The cloud's menu of "places your code runs" — one spectrum from most to least you manage:

| Tier | Products | Unit of deployment | You manage |
|---|---|---|---|
| VMs / VMSS | EC2, Azure VM/Scale Sets | machine image | OS, runtime, scaling |
| Containers (managed) | AKS/EKS/GKE | container image | workloads + cluster policy |
| Container-as-service | Cloud Run, App Service (custom), Fargate | container image | just the image |
| App platform (PaaS) | App Service, Heroku, Elastic Beanstalk | code + config | code |
| Functions (FaaS) | Lambda, Azure Functions | function | function code |
| Batch | Batch, EMR, Databricks | job | job definition |

## Why Does It Exist?

Every tier packages the same trade differently: **control vs. operational burden** (Service Models page). Compute options are that spectrum, *productized per unit of deployment* — and the unit matters:

```text
The smaller your unit of deployment, the smaller your unit of scaling:
machine →  container  →  process/function
minutes     seconds       milliseconds
```

That's the entire gravity of the industry's move rightward: finer-grained scaling units = better packing = less idle = faster elasticity.

## Layer 1 — Simple Explanation

A ladder of **who prepares the stage**:

- **VMs**: you rent an empty theater and build everything
- **Managed K8s**: the theater's built; you run the shows (containers)
- **Cloud Run/App Service**: the stage crew exists; you arrive with the script (image/code) and they run it
- **Functions**: you perform one scene per ticket sold; theater appears on demand

## Layer 2 — Engineer's View

**Choosing — the decision table:**

| Question | Points to |
|---|---|
| Custom OS/driver/GPU config? | VMs |
| Team runs K8s anyway / microservice fleet? | Managed K8s |
| Stateless web/service, no cluster ambitions? | Container-as-a-service (Cloud Run) |
| Standard web app, no container build? | PaaS (App Service) |
| Event-driven, spiky, short? | Functions |
| Data/ML batch? | Batch/EMR/Databricks |

**Autoscaling — the tiers differ in *what* scales:**

| Tier | Mechanism | Signal |
|---|---|---|
| VMSS/ASG | add/remove machines | CPU, queue, schedule |
| K8s HPA/VPA/cluster-autoscaler | pods then nodes | metrics (K8s phase) |
| Cloud Run / Functions | request-driven replicas | concurrency/requests — no config culture needed |

Scale-to-zero (functions / Cloud Run) changes economics and p99 (cold starts) — Service Models page's sharp edges apply per product.

**The image pipeline beneath modern compute:** whatever tier, you're shipping *images* — build → scan → registry → deploy (CI/CD + Artifact Repos pages). VM tier = AMI/Packer golden images; container tiers = OCI images. Immutable, versioned, promoted — the same discipline, different artifact types.

**Bare-metal/spot — the cost levers:**

- **Spot/preemptible**: 60–90% off, evictable — for stateless workers, CI agents, batch; *never* for your quorum nodes
- **Reserved/savings plans**: steady baseline load, 1–3y commitment
- FinOps phase formalizes; compute page just flags: the list price is for people not paying attention

## Real-World Example (DevOps flavored)

ShopEasy's compute estate, mapped honestly:

```text
Storefront API:      AKS nodepools (on-demand + spot for batch pods)
Image resize:        Functions (queue-driven, spiky — scale-to-zero overnight)
Reporting batch:     spot-VM node pool via K8s, checkpointed
Legacy ERP adapter:  one reserved VM (vendor image)
CI agents:           spot scale-set with graceful drain
```

One product per workload shape; cost graph flat where it should be, spiky where usage is.

## Common Mistakes

- Choosing by team fashion ("everything must be K8s") instead of workload shape
- Functions for steady load — the bill inversion from the Service Models page
- On-demand pricing for known-steady baseline (paying the ignorance tax)
- Spot instances for stateful singletons (the eviction lottery)
- Snowflake VMs outside any image pipeline — unbuildable, unpatchable

## Mental Model

> Compute tiers are **stages of theater automation**: from renting a plot (VM), to a built theater (K8s), to arriving with props (containers-as-service), to performing scenes on demand (functions). The industry's direction is one-way — smaller units, faster rigging — but you should still pick the smallest stage your show actually needs, not the biggest one your rival rents.

## Remember This

1. One spectrum; the differentiator is your *unit of deployment* (and thus of scaling)
2. Decision by workload shape: OS control → VMs; fleet → K8s; service → Run/PaaS; events → FaaS
3. Finer units = better elasticity and packing — the industry's gravity
4. Autoscaling follows the unit: machines, pods, or request-driven replicas
5. Cost levers: spot for evictable, reserved for steady baseline, on-demand for the unknown
6. Every tier wants an immutable image pipeline feeding it

## One Sentence

Cloud compute tiers — VM through functions — offer the same trade at decreasing units of deployment, letting each workload scale, pack, and pay at the granularity that matches its shape.

## Knowledge Check

1. Why does "unit of deployment" determine "unit of scaling"?
2. Place: CI agents, quorum database, queue consumers, steady web API — spot/reserved/on-demand/Function?
3. What does a VM-tier estate lose by having no golden-image pipeline?
4. Which tier makes autoscaling "boring," and why?

## Further Reading

- Azure Compute / AWS compute services docs — read as one spectrum
- [Google Cloud Run docs](https://cloud.google.com/run/docs) — request-driven scaling reference

---

**← Previous:** [Cloud IAM](iam.md)
**Next:** [Storage & Databases](storage-databases.md) →
**Related:** [Service Models](service-models.md) · [Autoscaling (K8s)](../kubernetes/autoscaling.md)
