# Drift & the Infrastructure Lifecycle

## What Is It?

- **Drift**: any difference between declared state (Git) and actual state (reality) — the chronic disease of IaC
- **Infrastructure lifecycle**: the full arc — plan → provision → operate → evolve → deprovision — of which apply is only the middle

## Why Does It Exist?

Drift is not a bug; it's **entropy with a cause list**:

| Source of drift | Example |
|---|---|
| Human | console hotfix at 2 AM |
| Automation outside IaC | cloud auto-tuning, K8s Service-created LBs |
| Provider | default mutations, mandatory migrations |
| Time | certificates, rotating credentials, version upgrades |
| IaC itself | half-failed applies |

The lifecycle matters because IaC culture over-focuses on *creation* — while most cost, risk, and toil live in **operate** (years) and **deprovision** (the zombie-resource graveyard of every cloud bill).

## Layer 1 — Simple Explanation

Drift = the **house slowly diverging from its blueprints**: a pipe rerouted in an emergency, a wall patched, a new outlet added — each individually reasonable, collectively a house nobody fully understands.

Lifecycle = admitting the house has a **whole life**: designed, built, *lived in and maintained for decades*, and eventually **demolished** — planning only the build is how you get haunted houses (running, billing, unowned).

## Layer 2 — Engineer's View

**Drift management — the three responses, chosen per incident:**

```text
detect (scheduled plan / diff tooling)
   ├─ revert: reality is wrong → apply declared state (the 2 AM hotfix undone at 9 AM)
   ├─ adopt:  reality is right  → codify it into Git (the hotfix was correct; PR it)
   └─ investigate: why did reality change? (an automation you didn't know about)
```

Detection tooling: scheduled `plan -detailed-exitcode`, drift modules (env0/scalr), cloud config recorders. The policy that makes it work: **the repo is the only door** (IaC page) — with break-glass paths that *auto-create the adoption PR*.

**Prevention beats detection:**

- No standing console write-access in IaC-managed scopes (IAM page)
- Known auto-mutators (autoscaling-created resources, K8s-managed LBs) *excluded from IaC scope deliberately* — document the boundary
- `ignore_changes` where mutation is by design — an explicit contract, not a shrug

**The lifecycle stages and their disciplines:**

| Stage | Discipline |
|---|---|
| Plan | capacity/cost estimates (FinOps), threat model (Security phase) |
| Provision | plan-reviewed applies, policy gates |
| **Operate** | drift detection, upgrades (versioned module bumps *as routine*), patching cadence (or immutability), observability of the infra itself |
| **Evolve** | refactors behind stable module interfaces — consumers unaffected |
| **Deprovision** | data export/retention plan → destroy → verify billing zero — *decommissioning is a feature with a runbook* |

**The zombie audit (the money page of this phase):** every estate accumulates resources nobody remembers: old snapshots, orphaned disks, forgotten test envs. The quarterly ritual: tag-audit (`owner`, `expires`), cost-per-tag reports, and the courage to delete — with IaC, deletion is one reviewed plan. (Expires-on: resource tags + automated reaper = zombie prevention by design.)

## Real-World Example (DevOps flavored)

The drift postmortem that sells the whole discipline:

```text
Incident: prod LB behavior changed; nobody deployed anything
Timeline: Terraform plan in CI shows: "health_probe ~ modified"
Forensics: audit log → a teammate's console change Tuesday (urgent TLS fix)
Resolution: change was correct → adoption PR codifying it (with the review it
            skipped), console write access revoked, break-glass PR-automation added
Outcome: next hotfix takes the door, not the window — by making the door faster
```

## Common Mistakes

- Drift as alert-noise nobody actioned — detection without the triage policy is decoration
- Adopting silently (`terraform import` without the PR) — the change stays unexplained
- Scope wars: both IaC and K8s managing the same LB (fighting reconcilers)
- No `owner/expires` tags — the zombie estate grows in the dark
- Never upgrading modules (version pins as fossils) — the upgrade debt compounds
- Demolition fear: unused environments kept "just in case" at full price

## Mental Model

> Drift is the **house wandering from its blueprints** — every emergency repair a small undocumented renovation. The cure is cultural: make the blueprint *the fastest way to change anything*, so nobody climbs through the window. The lifecycle adds the honest epilogue: houses are maintained for decades and must be *demolished on purpose* — or they become billing-relevant haunted houses.

## Remember This

1. Drift = declared ≠ actual; sources: humans, out-of-band automation, providers, time
2. Triage per incident: revert / adopt (via PR) / investigate — detection alone is noise
3. Prevention: repo-is-the-only-door, IAM scoping, deliberate boundaries for auto-mutators
4. Operate and deprovision dominate lifetime cost — upgrades routine, demolition runbooked
5. owner/expires tags + reaper = zombie prevention
6. Two controllers over one resource = reconciler war — scope explicitly

## One Sentence

Drift is the chronic divergence between declared and actual infrastructure, managed by detect-revert-or-adopt discipline within a lifecycle whose real cost lives in operating and deliberately ending resources — not creating them.

## Knowledge Check

1. Name four drift sources and the prevention for each.
2. When is `ignore_changes` a contract vs a smell? Give one legitimate use.
3. Design the zombie-audit: tags, cadence, tooling, and the safety rail before deletion.
4. Why do two reconciliation systems on one resource cause incidents? Give the K8s/IaC example.

## Further Reading

- Next: [GitOps](gitops.md) — drift management *continuous* and *in-cluster*
- *Terraform: Up & Running* — lifecycle chapters; cloud billing docs' tag-based cost views

---

**← Previous:** [Ansible](ansible.md)
**Next:** [GitOps](gitops.md) →
**Related:** [Terraform](terraform.md) · [Drift → GitOps](gitops.md)
