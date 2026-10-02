# Helm & Operators

## What Is It?

Two layers of "package management + operational knowledge" for Kubernetes:

- **Helm** — the package manager: templated YAML bundles (**charts**) with versions and values-files; renders plain manifests
- **Operators** — the operational expertise encoded as software: **CRDs + controllers** ("I'll run PostgreSQL for you, the way an expert would")

```text
Helm:    packaging + parameterization of what you deploy
Operator: a controller that RUNS it day-2 (upgrade, backup, failover)
```

## Why Does They Exist?

Raw K8s manifests don't compose across environments (dev/staging/prod differ in 20 values), don't version, and don't carry day-2 operations. Two gaps, two answers:

1. **Packaging gap** — 40 interrelated objects per app × N environments → Helm's templates+values
2. **Operations gap** — deploying a database is easy; *operating* it (backups, replication, upgrades) is a human skill → Operators encode the runbook *as a reconciler* (Architecture page's loop, applied to operations)

## Layer 1 — Simple Explanation

- **Helm chart** = an **IKEA flat-pack**: one box, all parts, an assembly manual, and an options sheet (values: color, size)
- **Operator** = the **assembly + maintenance service**: unpacks it, builds it, and — unlike IKEA — keeps servicing it forever (tightens bolts, replaces worn parts, upgrades the model)

## Layer 2 — Engineer's View

**Helm mechanics:**

```bash
helm install payments ./chart -f values.yaml -f values-prod.yaml
# template → render → apply; upgrade/rollback = release history in secrets
helm upgrade payments ./chart --set image.tag=2.1.1 && helm rollback payments 1
```

- **Layered values** (chart defaults ← env values ← `--set`) is the environment-differentials pattern (ConfigMaps page's separation)
- Chart *dependencies* (umbrella charts) compose apps+infra per environment
- The honest limits: post-render drift is invisible to Helm (`helm get manifest` ≠ cluster truth — the gap GitOps closes), templating YAML in Go templates is famously fiddly, and `--set` escapes quote-hell await you

**Operator mechanics (CRD + controller = new object type + its expert):**

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster            # ← a CRD: "PostgreSQL" is now a first-class object
spec: { instances: 3, storage: { size: 50Gi } }
# the operator reconciles: statefulsets, config, replication, failover, backups
```

- The **CRD pattern is K8s' extensibility payoff** (Architecture page): *anything* becomes a declarative object with a reconciler — databases (CloudNativePG), cert-manager's Certificates, ArgoCD's Applications (next phase!), Prometheus's monitoring objects
- Quality varies wildly: audit before trusting — an operator is *someone's runbook, automated*, including its bugs. Prefer CNCF/established operators for stateful things

**Helm vs Operator vs GitOps (they stack, not compete):**

```text
Chart packages manifests → Operator operates stateful day-2
GitOps (ArgoCD) deploys and keeps truth ← next phase resolves the drift gap
```

Common production stack: chart for the app, operators for postgres/redis/certs, ArgoCD applying both from Git.

## Real-World Example (DevOps flavored)

ShopEasy's split:

```yaml
# App delivery: Helm chart per service (umbrella: deployment+service+ingress+hpa+servicemonitor)
# values-dev: 1 replica, debug image, tiny resources
# values-prod: HPA 3-30, prod registry digests, PDBs
# Stateful: CloudNativePG operator — Cluster CR, automated backups to S3, failover <30s
# Certs: cert-manager operator — Certificate CRs → ACME → Secrets (TLS page, automated)
```

The before/after that matters: DB upgrades went from weekend runbooks (human, error-prone) to operator-managed rolling replica upgrades with a versioned CR — the runbook became code with a reconciliation guarantee.

## Common Mistakes

- Believing Helm manages the running app — it renders; drift is invisible to it
- `helm upgrade --set` snowflakes instead of values files in Git
- Operators adopted unaudited — running an unknown team's runbook as cluster-admin
- Using an operator for something trivial (a Deployment suffices) or none for something stateful (you become the operator, badly)
- Umbrella charts with 200 transitive values — configuration sprawl

## Mental Model

> Helm is the **flat-pack with an options sheet** — efficient packaging, zero ongoing relationship. The Operator is the **lifetime service contract**: a domain expert (as a controller) who reads your one-line spec and reconciles the entire operational reality toward it forever. Package with the flat-pack; subscribe to the service for anything with state.

## Remember This

1. Helm = templated, versioned packaging; values-files separate environment deltas
2. Helm is blind to post-deploy drift — GitOps (next phase) closes that
3. Operator = CRD + controller: operational expertise as a reconciler
4. CRDs are the platform's extension mechanism — ArgoCD/monitoring/certs all arrive this way
5. Audit operators — you're adopting someone's automated runbook
6. Typical stack: charts for apps, operators for stateful + platform services, GitOps over both

## One Sentence

Helm packages parameterized manifests for deployment while Operators encode day-2 operational expertise as custom controllers — packaging and operations, both riding Kubernetes' reconciliation loop.

## Knowledge Check

1. What can't Helm see after deployment, and which technology fixes it?
2. Explain an Operator strictly in the Architecture page's reconciliation vocabulary.
3. When should you NOT adopt an operator for a stateful workload?
4. Compose chart/operator/GitOps for a web app + Postgres + TLS — who owns what?

## Further Reading

- [helm.sh docs](https://helm.sh/docs/) · [Operator pattern — kubernetes.io](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/)
- Next: [Kubernetes Troubleshooting](troubleshooting.md)

---

**← Previous:** [Autoscaling & Scheduling](autoscaling.md)
**Next:** [Kubernetes Troubleshooting](troubleshooting.md) →
**Related:** [Kubernetes Architecture](architecture.md) · [GitOps](../iac/gitops.md)
