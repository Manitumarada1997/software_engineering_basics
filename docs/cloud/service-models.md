# IaaS, PaaS, SaaS, Serverless

## What Is It?

A spectrum of **how much of the stack the provider manages** — from "they manage hardware" to "they manage nearly everything":

```text
        you manage ──────────────────────────────► provider manages
On-prem    IaaS         CaaS/PaaS        FaaS/Serverless      SaaS
hardware   VM+OS+app    runtime+app      your function        the whole product
```

| Model | You bring | Examples | You still own |
|---|---|---|---|
| **IaaS** | OS, runtime, app | EC2, Azure VMs | OS patching, scaling config, HA |
| **CaaS** | containers + orchestration setup (or managed) | AKS/EKS/GKE | images, workloads |
| **PaaS** | code + config | App Service, Heroku, Cloud Run | app + its scaling settings |
| **FaaS** | a function | Lambda, Azure Functions | code + cold-start design |
| **SaaS** | nothing but usage | Office 365, Salesforce | data, identity integration |

## Why Does It Exist?

Each rung trades **control for reduced operational burden** — the toil conversation from the XP/Lean page, applied to infrastructure. Nobody philosophically prefers managing OS patches on 400 VMs; rungs exist because different workloads need different control points:

- Custom kernel module or GPU driver → IaaS
- Web app with standard runtime → PaaS/Cloud Run
- Event-driven glue → FaaS
- Email → SaaS, obviously

## Layer 1 — Simple Explanation

Eating models:

- **On-prem**: hunt, farm, cook, wash dishes
- **IaaS**: rented kitchen — your knives, your recipes, your cleanup
- **PaaS**: a food-truck slot — plug in, cook; truck maintenance is theirs
- **Serverless**: order-per-dish catering — a meal appears per request; nobody's idle kitchen burns money
- **SaaS**: restaurants

## Layer 2 — Engineer's View

**The real axis: responsibility vs. leverage.** Every rung right removes undifferentiated work (patching, capacity) but adds constraints (runtime choices, portability, provider coupling). Senior-engineer judgment = picking the *lowest* rung that meets requirements — not the most comfortable, not the most impressive.

**Serverless mechanics (the rung with the sharpest edges):**

- **Scale to zero**: no idle cost; per-invocation billing (ms-granular)
- **Cold starts**: new instance = initialize runtime (100ms–seconds) — p99 killer for synchronous paths; mitigate (provisioned concurrency) or avoid (use for async)
- **Event-driven glue is its sweet spot**: queue message → process → done (shop's order-events pipeline)
- **Limits as architecture**: execution timeouts (15m), concurrency caps, payload sizes — serverless shapes your architecture, not just hosts it

**Hidden cost model reality (FinOps preview):**

- IaaS: paying for *provisioned* capacity (idle VM = money)
- Serverless: paying for *used* capacity (but at higher unit rates under constant load)
- Crossover point: steady high load is usually cheaper on IaaS/CaaS; spiky/unknown load is serverless' home turf

**Portability ranking** (your exit-option): VM images (portable-ish) > containers (very) > functions (coupled to provider's event model) > PaaS/SaaS (married). Not always decisive — but a conscious choice, not an accident.

## Real-World Example (DevOps flavored)

ShopEasy's realistic mix:

```text
Checkout core:      AKS (CaaS) — steady load, team owns K8s anyway (next phases)
Image processing:   Lambda/Fuctions triggered from queue — spiky, 2-min bursts
Static storefront:  SaaS/CDN + blob storage
Internal email/CI:  SaaS
Legacy ERP link:    one IaaS VM (vendor requires their OS image) — contained blast radius
```

That's the mature answer: not "we use X", but a per-workload decision table.

## Common Mistakes

- "Serverless = no servers = no ops" — there is no *server* ops; there is *everything else* ops (observability, security, cost, concurrency)
- Forgetting FaaS ceilings until the first production throttling event
- PaaS/FaaS for steady heavy load — the bill inversion
- Marrying every workload to one rung by team ideology rather than workload shape
- Assuming portability you never tested

## Mental Model

> The spectrum is a row of **meal plans**, from self-catering (IaaS) through food trucks (PaaS) to per-bite catering (FaaS). The engineering question is always: *which responsibilities do we actually want to keep?* — keep too many and you're a restaurant nobody asked you to run; give away too many and the menu can't serve your customers.

## Remember This

1. The models are a responsibility spectrum, not a technology choice
2. Pick the *lowest* rung meeting requirements — control only where needed
3. Serverless: scale-to-zero economics, cold-start p99 risk, event-driven sweet spot
4. Cost crossover: spiky→serverless, steady→containers/VMs
5. Portability decreases as you climb; make that a conscious decision
6. Real systems mix rungs per workload — decision tables, not ideology

## One Sentence

IaaS through SaaS is a spectrum trading control for managed responsibility, and engineering judgment means choosing per workload the least amount of infrastructure you're willing to stop owning.

## Knowledge Check

1. Why is serverless often *more* expensive at steady high load?
2. Your Lambda-based API has a p99 cold-start problem — three mitigations?
3. Map your current estate onto the rungs; which choice is ideology, not shape?
4. What do you still operate in a pure FaaS deployment? (Hint: nearly everything except servers.)

## Further Reading

- [AWS compute services overview](https://aws.amazon.com/products/compute/) / Azure equivalent — see the spectrum productized
- Martin Fowler — serverless articles (Sam Newman/"Serverless: just functions?")

---

**← Previous:** [What Cloud Really Is](cloud-fundamentals.md)
**Next:** [Regions & Availability Zones](regions-az.md) →
**Related:** [HA & Disaster Recovery](ha-dr.md) · [FinOps](../advanced/finops.md)
