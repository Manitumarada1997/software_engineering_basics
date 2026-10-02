# Terraform

## What Is It?

The dominant open-source IaC tool (HashiCorp, 2014): HCL files describe desired cloud resources; Terraform computes a **plan** (diff) and **applies** it — for 100+ providers (AWS/Azure/GCP/K8s/GitHub/Datadog/...).

```hcl
resource "azurerm_postgresql_flexible_server" "db" {
  name                   = "payments-db"
  resource_group_name    = azurerm_resource_group.rg.name
  sku_name               = "B_Standard_B1ms"
  storage_mb             = 32768
  lifecycle { prevent_destroy = true }        # the crown-jewel guard
}
```

## Why Does It Exist?

To be the **vendor-neutral layer over every provider's API** — one language, one plan/apply model, one workflow for anything with an API. The single most important object it manages:

**State — the tool's memory of reality:**

```text
state file = Terraform's map: "resource X in your files" ↔ "real resource ID in the cloud"
             refresh: cloud → state      plan: state vs files → diff      apply: diff → cloud
```

Without state, every run would be blind archaeology ("does this subnet exist? which ID?"). With it, the diff is fast and precise. *All of Terraform's operational weirdness is state weirdness.*

## Layer 1 — Simple Explanation

Terraform is a **renovation contractor with a perfect memory**:

- Your architect's drawing (HCL) = what the house should be
- The **site survey book (state)** = what the contractor *believes* exists
- **Plan** = the quote: "we'll add a bathroom, remove the shelf" — shown before any work
- **Apply** = the work, updating the survey book as it goes

If a stranger changes the house without telling the contractor (console edits), the survey book lies — *drift* — and the next plan either reverts reality or surprises you.

## Layer 2 — Engineer's View

**State operations — the production discipline list:**

| Requirement | Mechanism |
|---|---|
| Shared (team) access | **remote backend** (S3+Dynamo / Azure blob / HCP / Terraform Cloud) — never local files in Git |
| Concurrency safety | **state locking** — one apply at a time |
| Secrets exposure | state stores resource属性 (incl. secrets as plain text!) — encrypt backend, restrict access |
| Recovery | versioned backend / regular snapshots — DR page's law for state |
| Re-association | `terraform import` for existing resources (painful but real) |
| Split | per-environment/component states — blast-radius boundaries (not one mega-state) |

**Modules — the reuse unit:**

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"                    # pin! (Versioning page — supply chain applies)
  cidr    = "10.0.0.0/16"
}
```

- Modules = functions: inputs, outputs, versioned, tested (terratest), consumed like libraries
- The registry + module pinning is the dependency-management story (Maven page) for infrastructure
- Anti-pattern: environments copy-pasting resource blocks instead of one module × N values

**The workflow that makes Terraform safe (pipeline reality):**

```text
PR: fmt + validate + plan → plan posted as review artifact
Review: humans read the diff; OPA/Conftest policies gate (no public IPs, tags required, allowed SKUs)
Merge: apply (scoped OIDC identity — IAM page) → drift detection scheduled nightly
```

**Provider/plugin mechanics worth knowing:** providers do the API talking (the plan is only as fresh as provider versions — pin them); `terraform.lock.hcl` is the dependency lockfile; the 2023 license change (BSL) birthed **OpenTofu** (next page) — same language, MPL fork, CNCF-hosted.

**What Terraform is bad at** (know the edges): day-2 operations of stateful software (operators won — Helm page), drift *response* (it reports, doesn't continuously reconcile — GitOps will), and imperative glue (Ansible's page).

## Real-World Example (DevOps flavored)

The two state incidents every team earns:

```bash
# 1. "Someone deleted the resource in the console":
terraform plan     # shows: resource will be RECREATED (it's gone from reality)
# decision: accept recreation (data loss? snapshot first!) or import the replacement

# 2. "Apply failed halfway" (state partially updated):
terraform state list; terraform refresh
# re-plan against refreshed state — idempotent retry; locking prevented double-apply chaos

# The nightly drift check:
terraform plan -detailed-exitcode   # exit 2 = drift → alert + revert-or-adopt decision
```

## Common Mistakes

- Local state files in Git — collisions, secrets in history, no locking
- One giant state for everything — every apply risks everything; split per env/component
- Unpinned module/provider versions — yesterday-green builds (Versioning page, again)
- No `prevent_destroy` on databases; `-target` applies as routine (fragile partial state)
- Treating plan as CI noise instead of THE review artifact
- Secrets via plain variables (state stores them in cleartext — references to vaults instead)

## Mental Model

> Terraform is a **contractor with a survey book (state), a drawing (HCL), and a quote-first workflow (plan → apply)**. The survey book is the tool's brain: share it safely (remote backend), lock it (one crew at a time), back it up (it *is* your infra's identity), and never let strangers renovate behind its back — or pay in drift.

## Remember This

1. plan/apply with a diff-first workflow; state = the reality↔code map that enables it
2. State discipline: remote + locked + encrypted + versioned + split — all of ops Terraform is state ops
3. Modules are functions; pin versions; registry = dependency management for infra
4. The plan in the PR is the review artifact; policy-as-code gates the dangerous
5. OIDC-scoped apply identity; `prevent_destroy` on crown jewels
6. Terraform builds infra; it doesn't operate software or continuously reconcile (next pages)

## One Sentence

Terraform manages infrastructure as a versioned diff between declared HCL and cloud reality, with its state file as the crucial (and operationally demanding) memory linking code to resource IDs.

## Knowledge Check

1. Why does console access in a Terraform-managed environment create *two* problems, not one?
2. What exactly does the state file contain that demands encryption and access control?
3. Design the safe apply pipeline for a state containing a production database.
4. When does Terraform decide to *replace* rather than *update* — and how do you guard the data?

## Further Reading

- *Terraform: Up & Running* — Brikman (state chapters especially)
- Terraform docs — backends, remote state, import

---

**← Previous:** [IaC Fundamentals](iac-fundamentals.md)
**Next:** [OpenTofu & Pulumi](opentofu-pulumi.md) →
**Related:** [Semantic Versioning](../development-practices/versioning.md) · [Ansible](ansible.md)
