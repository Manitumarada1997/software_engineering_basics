# Trunk-Based Development

## What Is It?

Trunk-Based Development (TBD) means: **all developers integrate their work into one shared branch — the trunk (`main`) — at least every day or two.** Incomplete features are merged anyway and hidden behind **feature flags**.

```text
Rule 1: everyone merges to main, frequently
Rule 2: branches live < 1–2 days
Rule 3: incomplete work ships dark, behind flags
Rule 4: main is always releasable
```

## Why Does It Exist?

Because of a paradox the previous page set up: long-lived branches avoid immediate risk but create *bigger, later* risk (merge hell, big-bang integration). The industry's elite performers (per DORA/Accelerate research) resolved it by eliminating isolation time itself:

> The best integration strategy is not better merging — it's **never being far away from everyone else**.

TBD is the branching strategy that Continuous Delivery requires. Google, Meta, and most elite delivery organizations run some form of it.

## What Problem Does It Solve?

ShopEasy at 30 developers with GitFlow:

- `develop` is 400 commits ahead of `main`; nobody knows what's actually shippable
- The biweekly "release branch" ritual takes two days of cherry-picking and firefighting
- Two features touch the same checkout files; they discover each other in week 4, during a 6-hour merge
- A security fix waits for the next release train — a 12-day exposure window

With TBD + flags:

- The security fix merges to `main` and deploys in an hour
- The two checkout features collided in day 2 — a 20-minute conflict, resolved in a 200-line PR
- Every commit proves itself with the full automated test suite — "shippable" is continuously true, not periodically claimed

## Layer 1 — Simple Explanation

Imagine a team of writers producing a book:

- **GitFlow:** each writer works on their own manuscript for a month, then an editor attempts to merge five divergent novels into one coherent book.
- **TBD:** all writers write into the *same* document daily. Disagreements surface immediately while small. Unfinished chapters stay in the file but with a note: "don't print this page yet" — that note is a feature flag.

## Layer 2 — Engineer's View

**Prerequisite 1 — the test suite must make main safe.** If merging to main relies on human review alone, you've just moved the queue. TBD presumes: fast CI (minutes), meaningful automated tests, and branch protection. This is why TBD and CI co-evolved (XP lineage).

**Prerequisite 2 — feature flags make half-built work safe to merge.**

```java
if (flagEnabled("one-tap-checkout")) {
    return oneTapFlow();
}
return legacyCheckout();
```

Branching isolates *code*; flags isolate *behavior*. TBD swaps the first for the second. Cost: flag hygiene — every flag needs an owner and an expiry date, or the codebase becomes a labyrinth of dead branches of logic (see the Feature Flags page).

**Prerequisite 3 — small batches.** Little's Law again (Kanban page): less WIP = faster flow. 200-line PRs reviewed in hours are the unit of TBD. 3,000-line PRs make TBD impossible.

**Mechanics the pros use:**

| Technique | Purpose |
|---|---|
| Branch protection + required CI | main is protected by machines, not promises |
| `--depth 1` / repo caching in agents | make 20-a-day integrations cheap (you know this one) |
| Merge queues (GitHub, Zuul at Google-origin) | serialize merges; each is tested against the *result* of the previous |
| Dark launches / canary | release behavior gradually after merge |
| Abstract branches / branch by abstraction | restructure code in shippable steps |

**Google-scale variant:** *single mono-repo, nearly everyone commits to head*, protected by presubmit tests and a merge queue. TBD at planetary scale — proof it scales further than any alternative.

**What TBD is NOT:** committing broken code to main. "Incomplete" ≠ "broken": code compiles, tests pass, the *feature* is simply invisible.

## Real-World Example (DevOps flavored)

Pipeline reality of a TBD team:

```yaml
# PR validation (every push to a short-lived branch)
trigger: [ merge ]  # PR opened/updated
steps: [ build, unit-tests, sonar-quality-gate, image-scan ]
# main pipeline
trigger: [ main ]
steps: [ deploy-staging, e2e, deploy-prod-canary ]
```

Your work as a DevOps engineer *is* the enabling machinery of TBD: making CI fast enough that merging 10× a day is pleasant. Pipeline duration isn't a convenience metric — it *is* the team's integration cadence.

## Trade-offs

- Requires test automation maturity — non-negotiable prerequisite
- Feature-flag sprawl without hygiene
- Psychological shift: "my code is visible while unfinished" (this is a feature: early feedback)
- Harder with long-cycle hardware/packaged releases — GitFlow territory

## Mental Model

> TBD is a **potluck dinner where everyone brings one dish every evening** instead of a banquet planned for a month. Some dishes are half-seasoned (flags off), but nobody starves, and the kitchen never becomes a battlefield.

## Remember This

1. Everyone merges to main daily; branches live < 1–2 days
2. Feature flags isolate behavior so branches don't isolate code
3. Hard prerequisites: fast CI, strong tests, small PRs
4. Elite DORA performance correlates with TBD (Accelerate research)
5. "Incomplete but green" is the contract; "broken" never is
6. CI speed = integration cadence — your pipeline duration sets the team's rhythm

## One Sentence

Trunk-Based Development keeps every developer within a day of the shared mainline, using CI, small pull requests, and feature flags to make continuous integration cheaper than isolation.

## Knowledge Check

1. Why do feature flags make branch isolation unnecessary?
2. What happens to a TBD team whose CI takes 90 minutes?
3. Explain "incomplete but never broken" to a skeptical lead.
4. Why does Little's Law argue for small PRs?

## Further Reading

- [trunkbaseddevelopment.com](https://trunkbaseddevelopment.com) — the definitive reference
- *Accelerate* — Forsgren, Humble, Kim (DORA research)
- *Continuous Delivery* — Humble & Farley, chapter on branching

---

**← Previous:** [Branching Strategies](branching-strategies.md)
**Next:** [Code Review](code-review.md) →
**Related:** [Git Internals](git-internals.md) · [Feature Flags](feature-flags.md)
