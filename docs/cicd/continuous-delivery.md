# Continuous Delivery & Continuous Deployment

## What Is It?

Two rungs of the same ladder (Humble/Fowler definitions):

- **Continuous Delivery (CD):** every change that passes the pipeline produces an artifact that is **proven deployable to production at any time**. A human *decides* when to release (one click).
- **Continuous Deployment:** the human is removed — every green change **deploys to production automatically**.

```text
CI  →  Continuous Delivery  →  Continuous Deployment
       (always deployable)      (always deployed)
                    the difference is one approval gate
```

**Delivery is the capability; deployment is the policy.** Teams need the first; whether they enable the second is a business/risk decision.

## Why Does It Exist?

Because of batch size — again. Releases that are rare and big are risky and painful; releases that are frequent and small are boring and safe. The economics:

| | Big-batch releases | Small-batch CD |
|---|---|---|
| Risk per release | High (months of change) | Low (one small change) |
| Failure diagnosis | Needle in a haystack | The one thing that changed |
| Rollback | Dramatic | Trivial |
| Feature delay | Months | Minutes |
| Human stress | Release weekends | None |

Risk *per change* can never be zero — CD reduces **risk per release** by making releases tiny, and total system risk by making practice constant. (Deploying is a skill; a muscle used daily is stronger than one used quarterly.)

## Layer 1 — Simple Explanation

- **CI:** every merge is proven to *build and test*.
- **Continuous Delivery:** every merge is additionally proven to be *deployable* — it's sitting in the warehouse, cleared, ready.
- **Continuous Deployment:** the forklift automatically stocks the shelves the moment goods clear inspection.

The difference between the top two rungs is a **decision**, not a technology.

## Layer 2 — Engineer's View

**What CD actually requires (the checklist most teams fake):**

1. **One artifact, promoted, never rebuilt** (previous page)
2. **Deployment automation** — no manual steps; environments are code (IaC phase)
3. **Deployment strategies** — blue/green, canary (next page); "deploy" ≠ "flip all traffic"
4. **Feature flags** — decouple release from deploy (you know this by now)
5. **Fast, trustworthy rollback** — the *real* precondition. If rollback is scary, every deploy is a bet
6. **Observability** — you cannot safely deploy at 2 PM unattended without knowing the system's health (SRE phase)
7. **Tests that gate truthfully** — the pipeline's word must be reliable, or someone re-checks by hand and you're back to manual

**Deployment pipeline anatomy:**

```mermaid
flowchart LR
    CI[CI: build+test] --> ART[Artifact in repo]
    ART --> S1[Deploy staging<br/>+ smoke tests]
    S1 --> S2[Deploy prod canary 1%<br/>+ SLO checks]
    S2 -->|auto: health OK| S3[Ramp 10→100%]
    S2 -->|auto: degraded| RB[Auto-rollback]
    S3 --> DONE[Release (flags/announce)]
```

**The "deploy ≠ release" separation in practice** — three independent switches:

```text
1. Deploy:  new version is running in the environment
2. Expose:  traffic routes to it (canary/weights)
3. Release: users see the feature (flags)
```

Elite teams control each independently. Legacy practice fused all three into one terrifying Friday-afternoon event.

**On approvals:** continuous *delivery* keeps a human approval — but the approval should be the *only* human step. If approving takes two clicks but the pipeline can't proceed without a QA lead's Excel sign-off, you've built ceremony, not control.

## Real-World Example (DevOps flavored)

ShopEasy adopting CD, as your platform team would drive it:

```text
Before: monthly release train. 6h checklist. Deploy nights. ~15% failure rate.
After:  40 deploys/day across teams. Canary + SLO gates. Auto-rollback.
        Change failure rate 4%. MTTR 22 minutes.
```

The Azure DevOps shape: one multi-stage YAML (`build → dev → staging → canary → prod`), environments with approvals on `prod` (delivery), and — for the payments team — approvals removed (deployment). Both teams, one platform, a policy difference.

## Common Mistakes

- "We do CD" = deploys are *possible* but take 3 days of coordination — capability ≠ practice
- Rebuilding artifacts per environment — destroys the audit chain
- Deployment automation without **rollback** automation — half a brake is no brake
- CD as a big-bang project — start with one service, prove MTTR, expand
- Measuring deploy frequency without change-failure-rate — frequency alone can be gamed into chaos
- Confusing the capability (delivery) with the mandate (deployment) and forcing auto-deploys on a team with weak tests

## Mental Model

> CD is a **conveyor belt to the warehouse door**; continuous deployment is letting the belt load the truck too. If your belt occasionally drops broken goods (flaky tests), or the truck can't reverse (no rollback), you don't have a conveyor — you have a chute.

## Remember This

1. CI proves *buildable*; CD proves *deployable*; continuous deployment removes the human gate
2. Small batches reduce risk per release and make diagnosis trivial
3. Prerequisites: promoted artifacts, automated deploys, rollback, flags, observability
4. Deploy / expose / release are three independent switches
5. Rollback speed is the true enabler of deployment courage
6. Delivery = capability, deployment = policy — measure DORA, not vibes

## One Sentence

Continuous Delivery makes every green change a proven-deployable, one-click release candidate, and continuous deployment goes one step further by releasing them automatically — both by shrinking batches until releasing is boring.

## Knowledge Check

1. What single gate separates delivery from deployment, and why is it a policy not a technology?
2. Name the three independent switches that legacy releases fused together.
3. Why is rollback speed the precondition for deployment courage?
4. Your team deploys weekly with 3 days of coordination. Are they doing CD? What's missing?

## Further Reading

- *Continuous Delivery* — Jez Humble, David Farley (the book; still the reference)
- Jez Humble, ["Continuous Delivery vs Continuous Deployment"](https://continuousdelivery.com/)
- DORA State of DevOps reports —dora.dev

---

**← Previous:** [Artifact Repositories](artifact-repositories.md)
**Next:** [Deployment Strategies](deployment-strategies.md) →
**Related:** [Feature Flags](../development-practices/feature-flags.md) · [Shift Left / Shift Right](../development-practices/shift-left-right.md)
