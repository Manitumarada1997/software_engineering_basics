# Infrastructure as Code — Fundamentals

## What Is It?

**IaC**: define infrastructure in machine-consumable files, and let tools *converge reality to the files* — instead of humans clicking consoles or running scripts.

Two axes, four quadrants:

| | **Declarative** (desired state) | **Imperative** (steps) |
|---|---|---|
| **Mutable** (change in place) | (rare) | Ansible playbooks, shell |
| **Immutable** (replace) | **Terraform, Pulumi, CloudFormation** | bake-and-replace (Packer) |

The industry's gravity: **declarative + immutable** — same desired-state idea as K8s manifests (Helm page), applied to networks/VMs/databases.

## Why Does It Exist?

The pre-IaC world's failure modes you've lived (or heard war stories of):

```text
"Who changed the NSG?" — console edits, no history, no owner
Snowflake environments — staging differs from prod in unknown ways
Disaster recovery without a plan: the infra only exists in one region's console
Onboarding = 3 weeks of tribal knowledge
Audit = screenshots
```

IaC's move: **infrastructure gets the software engineering toolkit** — versioning (diffs!), review (PRs!), testing (plan!), reuse (modules!), CI/CD (pipelines!) — the entire course so far, pointed at infrastructure. The DR page's secret: environments reproducible from a Git repo are the ultimate recovery plan.

## Layer 1 — Simple Explanation

- **Imperative** = a **recipe**: "chop onions, heat pan..." — follow steps, hope for the dish
- **Declarative** = the **photo of the finished dish**: "make it look like this" — the tool figures out the steps
- **Mutable infra** = **renovating the same house** forever (each change leaves scars)
- **Immutable infra** = **building a new house** and moving out of the old one (CD page: deployment = replacement)

## Layer 2 — Engineer's View

**The converge loop (one pattern, third costume):**

```text
state files + desired (HCL) → plan (diff reality vs desire) → apply (create/update/destroy)
                ↑____________________state refresh_____________________↓
```

systemd (services), K8s (pods), Terraform (cloud resources): one intellectual core — *declared desired state, continuously or on-demand reconciled*.

**Mutable vs immutable — why immutable won for compute:**

| | Mutable (in-place patch) | Immutable (replace) |
|---|---|---|
| Consistency | config drift accumulates | every instance born identical |
| Rollback | reverse the change (hard) | run the old artifact (easy) |
| Failure mode | half-patched fleet | all-or-nothing per instance |
| Cost | cheap per change | rebuild pipeline required |

(You already run this: containers are immutable infra — the Images page *was* this argument.)

**What IaC demands beyond tools — the disciplines:**

1. **The repo is the only door**: console edits become drift (next pages' enemy). Guard with policy: detect + alert + revert
2. **Modularity**: modules as functions — inputs, outputs, versioned; the platform-team leverage point (Pipeline-as-Product page, infrastructure edition)
3. **Environments as data**: dev/staging/prod = same modules, different values files (the Helm values pattern again)
4. **Stateful secrets never in code**: what *is* infra-as-coded vs what stays in vaults (Security phase) — references, not values

**The blast-radius reality of IaC:** a bad apply deletes prod. Hence the discipline stack — plan review in PRs, policy-as-code guardrails (OPA/Sentinel), `prevent_destroy` lifecycles on crown-jewel resources, and least-privilege pipeline identity (IAM page: OIDC, scoped roles).

## Real-World Example (DevOps flavored)

ShopEasy's IaC estate shape:

```text
infra-repo: modules/ (vpc, aks, postgres, dns) + envs/{dev,staging,prod}/
PR → terraform plan posted as comment → peer + policy review → merge → CI applies
No console access in prod (break-glass only, audited)
New env for an experiment = copy a directory + values — deleted as easily
```

The culture check after adopting: "can we rebuild the whole company from an empty cloud account and a Git clone?" — when yes, DR, auditing, and onboarding all became one problem already solved.

## Common Mistakes

- Console "quick fixes" that become unexplainable drift
- Monolithic 5,000-line templates instead of modules (no review granularity)
- One shared dev/prod state or module version — blast radius unmanaged
- Treating plan output as a formality, not the review artifact
- Secrets in tfvars — variables files are code, not vaults
- No state backup/locking (concurrent applies corrupting reality)

## Mental Model

> IaC turns infrastructure from **a sculpture you carve by hand** into **a recipe+photo others can rebuild from** — every cloud resource versioned like source, reviewed like source, reproduced like source. Mutable infra renovates the one house forever; immutable infra ships in a new house and burns the old one — and only one of those houses accumulates ghosts.

## Remember This

1. IaC = declarative desired state + tools converging reality; infrastructure inherits SDLC tooling
2. Gravity: declarative + immutable (containers were this argument in miniature)
3. One reconciliation pattern across systemd/K8s/Terraform — the course's through-line
4. The repo is the only door; drift is the chronic disease
5. Modules = functions; environments = data
6. IaC's power inverts to danger: plans in PRs, policy guardrails, scoped identities

## One Sentence

Infrastructure as Code defines infrastructure declaratively in versioned files so that environments become reproducible, reviewable, and recoverable — extending the entire software engineering lifecycle to the machines it runs on.

## Knowledge Check

1. Why does immutability make rollback trivial where mutation makes it archaeology?
2. Connect Terraform's plan/apply to the K8s reconcile loop — same sentence, two costumes.
3. What organizational policy does "the repo is the only door" require beyond tooling?
4. Why is "we can rebuild from Git" the ultimate DR statement?

## Further Reading

- *Terraform: Up & Running* — Yevgeniy Brikman, ch. 1 (best IaC fundamentals chapter written)
- Next: [Terraform](terraform.md)

---

**← Previous:** [Kubernetes Troubleshooting](../kubernetes/troubleshooting.md)
**Next:** [Terraform](terraform.md) →
**Related:** [Why Orchestration](../kubernetes/why-orchestration.md) · [Pipeline as Product](../cicd/pipeline-as-product.md)
