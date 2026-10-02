# OpenTofu & Pulumi — the IaC Landscape

## What Is It?

The credible alternatives to Terraform, each diverging on a different axis:

| | Terraform | **OpenTofu** | **Pulumi** |
|---|---|---|---|
| Language | HCL | HCL (drop-in) | real languages (TS/Go/Python/C#) |
| License | BSL (2023) | MPL 2.0 (CNCF) | Apache (engine), some paywalled features |
| Model | declarative | declarative + growing extras | imperative code → declarative engine |
| Born from | HashiCorp 2014 | Terraform fork 2023 | independent 2018 |

## Why Does It Exist?

**OpenTofu — the license story.** In 2023 HashiCorp switched Terraform from open-source to BSL (no competing commercial use). The community forked the last MPL version; the Linux Foundation/CNCF adopted it; it tracks Terraform's language while adding its own features (e.g., `for_each` in providers, native state encryption). The lesson for your career: **the ecosystems you build on are also business decisions** — vendor risk is part of architecture.

**Pulumi — the paradigm story.** HCL is a DSL with no loops, no types, no tests. Pulumi's insight: infrastructure logic is *programming* — use real languages, real tooling:

```typescript
const subnetCount = az.env == "prod" ? 3 : 1;         // logic, not count tricks
for (const i of range(subnetCount)) new aws.Subnet(`snet-${i}`, {...});
if (config.getBool("enableNat")) { new aws.NatGateway(...) }
// + unit-testable: test("prod has 3 AZs", () => ...)
```

Its engine still builds a *declarative* resource graph (it's not scripts — preview/diff semantics remain) — "imperative authorship, declarative execution."

## Layer 1 — Simple Explanation

- **OpenTofu** = the **same restaurant under new, community ownership** after the landlord changed the lease terms — same menu (HCL), same kitchen, different landlord (foundation)
- **Pulumi** = the **restaurant where you cook in your own kitchen with your own knives** (your language, your editor, your tests) instead of the house's fixed toolset — the food still arrives as a proper plated dish (declarative graph)

## Layer 2 — Engineer's View

**Choosing — the honest decision table:**

| Situation | Pick |
|---|---|
| Existing HCL estate, want open-source guarantee | OpenTofu (migration is near-trivial) |
| Complex infra logic, polyglot teams, test culture | Pulumi |
| New estate, no strong opinions, hiring market | Terraform (ecosystem gravity) |
| Regulated: vendor support contracts | Terraform (HCP) or OpenTofu via support vendors |

All three share: providers model, plan/apply, remote state (Pulumi's service or self-hosted backend), drift detection. The reconciliation skeleton is identical — skills transfer.

**What real languages actually buy (Pulumi's real edges):**

- Abstraction as *code*: component resources = real classes; internal platforms become SDKs (a bridge to the Platform phase — this is Pulumi's serious story)
- Test tooling for infra logic (unit test the *logic*, not the cloud)
- No `count = var.x ? 1 : 0` DSL contortions

**What real languages cost:**

- Reviewing code that can do *anything* (HCL's limits were guardrails)
- Language runtime in CI, style/best-practice sprawl across a polyglot org
- Being the clever one: clever infra code is still a snowflake generator

**The landscape filler (one line each):** CloudFormation/ARM/Bicep — cloud-native, provider-locked, fine if single-cloud forever; CDK — same idea as Pulumi, synthesized to CFN; Crossplane — K8s-control-plane IaC (control planes composing via CRDs — Helm page's pattern, infra edition; watch it).

## Real-World Example (DevOps flavored)

ShopEasy's choice audit:

```text
Estate: 80% HCL modules, 3 clouds, one platform team
Decision: OpenTofu (license certainty, zero-rewrite migration) for the estate;
         Pulumi for the platform's self-service layer (the "infra SDK" the
         golden-path page will need — typed component for "a compliant service")
Result: one tool per *audience* — infra team vs. product teams building services
```

That last line is the mature pattern: tool choice by **consumer**, not by ideology.

## Common Mistakes

- Tool wars as identity — the reconcile-loop skeleton is identical in all of them
- Migrating HCL→Pulumi for "real language" cosmetics while the modules were fine
- Ignoring license/vendor risk until a BSL moment happens to *your* stack
- In Pulumi: writing untestable spaghetti because you *can* (DSL limits were accidental guardrails)
- Forgetting state discipline is identical everywhere — backends, locking, secrets

## Mental Model

> The IaC tools are **contractors sharing the same building code** (declare → plan → apply → state) with different crews: OpenTofu kept the house crew under new ownership; Pulumi lets you bring your own craftsmen with their own tools. Judge them by crew fit — never by which hammer is fashionable.

## Remember This

1. OpenTofu = MPL fork of Terraform (license-driven; CNCF; near drop-in)
2. Pulumi = real languages authoring a declarative engine — logic + tests for infra
3. All share providers/plan/state/drift — the skills, not the syntax, are the career
4. Real languages buy abstraction (platform SDKs) and tests; they cost review surface
5. Vendor/license risk is an architecture input (BSL lesson)
6. Crossplane is the K8s-control-plane direction worth watching

## One Sentence

OpenTofu preserves the Terraform model under an open license while Pulumi replaces the DSL with real programming languages atop the same declarative engine — two divergences from one skeleton, chosen by license posture and audience rather than power.

## Knowledge Check

1. What triggered OpenTofu, and what does the episode teach about stack selection?
2. Pulumi is "imperative authorship, declarative execution" — explain the engine's role.
3. When is provider-locked native IaC (Bicep/CFN) the *right* call?
4. Which Pulumi feature previews the Platform phase, and how?

## Further Reading

- [opentofu.org](https://opentofu.org) · [pulumi.com/docs](https://www.pulumi.com/docs/)
- Next: [Ansible](ansible.md)

---

**← Previous:** [Terraform](terraform.md)
**Next:** [Ansible](ansible.md) →
**Related:** [OCI & Runtimes](../containers/oci-runtimes.md) (the other standardization story)
