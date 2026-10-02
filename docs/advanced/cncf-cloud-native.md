# Cloud-Native & the CNCF Landscape

## What Is It?

- **Cloud-native** (CNCF definition, abridged): building systems as *loosely-coupled services* in **containers**, **dynamically orchestrated**, managed via **declarative interfaces and automation** — you've built that definition page by page through this course
- **CNCF** (Cloud Native Computing Foundation): the vendor-neutral home for the ecosystem's projects (Kubernetes, Prometheus, Envoy, OTel, Argo, Backstage...) — graduated/incubating/sandbox tiers marking maturity

The famous **landscape map** (1000+ logos) is navigable now: nearly every square is a page you've read.

## Why Does It Exist?

The CNCF exists for the same reason OCI did (its page): **neutral ground prevents vendor capture of shared infrastructure**. Kubernetes' 2015 donation to a foundation — rather than Google keeping it — is *why* every vendor, cloud, and enterprise standardized on it. The pattern (donate the layer everyone needs, compete above it) built the modern stack.

And "cloud-native" as a term earns its keep by naming the *bundle* of practices that only work together: containers + orchestration + declarative config + immutable delivery + observability — the course's Phases 3–9, one word.

## Layer 1 — Simple Explanation

The CNCF is the **standards body and town square** of the cloud-native city: it doesn't build every shop, but it keeps the roads public (so no vendor can toll them), certifies the trades (graduated projects = inspected and bonded), and posts the city map (the landscape — now readable to you as a *table of contents* rather than wallpaper).

## Layer 2 — Engineer's View

**The landscape, organized by your course (the decoder ring):**

| Landscape category | You know it as |
|---|---|
| Runtime (containerd, gVisor/Kata) | OCI page |
| Orchestration/management (K8s, Helm, Operators, Crossplane) | K8s phase |
| CI/CD (Argo, Flux, Tekton) | CI/CD + GitOps pages |
| Observability (Prometheus, OTel, Jaeger, Fluentd) | SRE phase |
| Service mesh/proxy (Envoy, Istio, Linkerd) | Proxies + Zero Trust pages |
| Policy (OPA/Kyverno) | PaC page |
| Security (Falco, Cosign, Trivy, SPIFFE) | Security phase |
| App definitions (Backstage, Operators) | Platform phase |
| Storage/database (Rook, Vitess, TiKV) | Storage + Sharding pages |

**Maturity tiers — what they actually certify:** graduated = battle-tested at scale with governance (K8s, Prometheus, Argo); incubating = real adoption, growing; sandbox = experiments. Selection heuristic: **graduated by default, incubating deliberately, sandbox never for crown jewels** — the same risk-tiering as everywhere else.

**The strategic readings the map enables:**

- Consolidation watch: multiple projects per square converge (the winners emerge — e.g., OTel absorbing the tracing-instrumentation field)
- Gap watch: a square with no graduated project = either immature category or genuinely unsolved (e.g., long-standing gaps in secrets/configuration)
- Career calibration: graduated-project skills compound (hiring market follows the map)

**Cloud-native's honest boundary:** the definition is *operational*, not moral — a modular monolith on two VMs with great CI/CD and observability is more "cloud-native in spirit" than a 40-service K8s sprawl with none of the practices. The practices are the substance; the logos are the wardrobe (the Monolith page's warning, ecosystem edition).

## Real-World Example (DevOps flavored)

ShopEasy's stack, read off the landscape:

```text
K8s (graduated) · Helm+ArgoCD (graduated) · Prometheus+Grafana (graduated/…) ·
OTel (incubating→graduated track — standardized anyway: vendor-neutral emitter) ·
Kyverno (incubating — deliberate: simpler than OPA for our policy set) ·
Backstage (incubating — accepted: portal criticality, but staffed accordingly)
Rule applied: one deliberate non-graduated bet max per category, reviewed quarterly
```

## Common Mistakes

- Landscape-driven architecture ("let's use more of the map") — logos aren't requirements
- Sandbox projects in critical paths — novelty risk priced at zero
- Missing the exitence of the *graduated tier* as a risk filter
- Believing cloud-native = K8s count: practices over logos
- Ignoring CNCF governance signals (project dormancy = your next migration)

## Mental Model

> The CNCF is the city's **public-roads authority and guild hall**: keeping the layers everyone needs out of any one vendor's hands, certifying the trades by maturity tier, and posting the map. You now read the map as a course index — every category a chapter you've finished.

## Remember This

1. Cloud-native = the bundled practices: containers, orchestration, declarative, immutable, observable
2. CNCF = neutral ground; foundation donation is *why* the stack standardized
3. The landscape is your course's table of contents — read by category
4. Maturity tiers as risk filter: graduated default, incubating deliberately, sandbox never (critical paths)
5. Practices > logos — the definition is operational, not a logo count
6. Watch consolidation (category winners) and dormancy (your future migrations)

## One Sentence

Cloud-native names the bundle of practices this course built — containers, orchestration, declarative delivery, observability — and the CNCF provides the vendor-neutral ground and maturity tiers that let you navigate its thousand-logo landscape as a risk-filtered catalog rather than noise.

## Knowledge Check

1. Place five components of your stack on the landscape by category and maturity tier.
2. Why does foundation governance matter for a project you build on — name one concrete risk it retires.
3. Which landscape squares are consolidating, and what does that predict for your stack?
4. Defend or attack: "our modular monolith is not cloud-native."

## Further Reading

- [CNCF landscape](https://landscape.cncf.io/) — now readable; [Graduated projects list](https://www.cncf.io/projects/)
- Next: [FinOps](finops.md)

---

**← Previous:** [Platform as Product](../platform-engineering/platform-as-product.md)
**Next:** [FinOps](finops.md) →
**Related:** [OCI & Runtimes](../containers/oci-runtimes.md)
