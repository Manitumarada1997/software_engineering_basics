# What Cloud Really Is (Virtualization First)

## What Is It?

**Virtualization** — one physical machine hosts many isolated virtual machines (hypervisor partitions CPU, memory, network, storage).

**Cloud computing** — virtualization *as an on-demand, pay-per-use, API-driven service*: NIST's definition boils down to five properties:

1. On-demand self-service (API/console, no human in the loop)
2. Broad network access
3. Resource pooling (multi-tenant)
4. Rapid elasticity
5. Measured service (pay per use)

AWS EC2 (2006) turned "rent a VM" from a ticket queue into an API call. The rest is history and architecture.

## Why Does It Exist?

The economics it replaced — capacity planning by procurement:

```text
2003: forecast peak load → buy servers → 3-month wait → over-provision 5×
      (idle 340 days/year, exhausted on day 341)
Datacenter = capital expense, sunk, wrong-sized, someone's pet servers
```

Cloud converts **CapEx into OpEx**, and — the deeper change — **infrastructure into software**: a load balancer is an API call, a network is config, an entire environment is code (the IaC phase exists because of this property). Combine with the earlier lessons: elastic capacity + APIs = autoscaling, immutable infrastructure, GitOps — none of which were practical with physical servers.

## Layer 1 — Simple Explanation

Old world: **owning a generator** — huge upfront cost, sized for the hottest summer day, idle otherwise, your problem when it breaks.
Cloud: **the power grid** — flip a switch, pay per kWh, the utility handles scale and failures. And because it's metered, leaving the lights on *costs you visibly* (FinOps phase foreshadowed).

## Layer 2 — Engineer's View

**The enabling stack, one level down:**

```text
Physical host → Hypervisor (VMware/KVM/Hyper-V) → VMs
             → Containers (shared kernel — Linux phase)
             → Serverless (someone else's containers + scale-to-zero)
```

VMs virtualize *hardware* (strong isolation, minutes to start); containers virtualize the *OS view* (namespaces, milliseconds). Clouds run containers inside VMs — you get both isolation levels in every managed K8s node.

**Multitenancy and the isolation contract:** your VM shares a physical host with strangers. Isolation = hypervisor boundary (+ for the paranoid: dedicated hosts). This is the threat model difference vs. your own racks — and why compliance regimes ask specific cloud questions.

**The three mental shifts from on-prem to cloud (the ones engineers actually struggle with):**

| On-prem instinct | Cloud discipline |
|---|---|
| Servers are precious pets (name them, nurse them) | Cattle: disposable, replace via automation (immutable infra) |
| Network is physical topology | Network is *software* (VPC = config; SDN underlies it) |
| Security = perimeter | Security = identity + policy per resource (IAM phase) |

**Shared responsibility — the sentence to memorize:** *you* are always responsible for your data, identity, and configuration; the provider's responsibility grows with the managed-ness of the service (VM: they secure the hypervisor, you patch the guest OS; managed DB: they patch; SaaS: nearly everything). Most "cloud breaches" are misconfiguration — the customer's side of the line.

**Regions/AZs preview:** cloud physical topology = regions (geographies) containing isolated availability zones (one or more DCs). The unit of HA design (next pages).

## Real-World Example (DevOps flavored)

ShopEasy's migration in one pipeline: what used to be a 6-week hardware refresh became — VMSS/Autoscaling behind an LB, defined in Terraform, deployed per environment in minutes, deleted when done (environments-as-code). The cultural change your org actually felt: "can we get a staging cluster?" stopped being a question with a price tag and became a PR review.

## Common Mistakes

- Lifting-and-shifting pets: VMs named `shopeasy-app-01` hand-tended forever — paying cloud prices for datacenter practices
- Assuming the provider secures *your* configuration (open S3 buckets say hello)
- No cost telemetry from day one — the meter runs even when nothing deploys
- Treating cloud as "someone else's computer" without understanding shared responsibility

## Mental Model

> Virtualization let one building house many tenants' offices. Cloud made the building **a vending machine**: offices by the hour, floors on demand, billed by the lightbulb — and because everything is an API, the whole building can be described in a file and rebuilt by a robot (your job).

## Remember This

1. Virtualization = partitioning; cloud = virtualization as metered, API-driven service (NIST's 5 properties)
2. Converts CapEx→OpEx and — more importantly — infrastructure into software
3. VMs isolate hardware; containers isolate the OS view; clouds stack both
4. Pets→cattle, physical→software networks, perimeter→identity: the three mental shifts
5. Shared responsibility: your data/identity/config is always yours
6. Most cloud incidents are misconfiguration, not provider failure

## One Sentence

Virtualization partitions one machine into many, and cloud computing sells that capability as a metered, self-service API — turning infrastructure from procured hardware into versionable software.

## Knowledge Check

1. Which NIST property makes GitOps possible, and why?
2. Contrast VM and container isolation in one sentence each.
3. Whose responsibility is an open S3 bucket? Whose is a hypervisor escape?
4. Name the three mental shifts from on-prem instincts to cloud discipline.

## Further Reading

- NIST SP 800-145 — *The NIST Definition of Cloud Computing* (2 pages, foundational)
- AWS/Azure/GCP "shared responsibility model" docs

---

**← Previous:** [API Gateway](../networking/api-gateway.md)
**Next:** [IaaS, PaaS, SaaS, Serverless](service-models.md) →
**Related:** [Compute Options](compute.md) · [Roadmap](../roadmap.md)
