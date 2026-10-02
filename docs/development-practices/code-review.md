# Code Review

## What Is It?

Code review is the systematic examination of a change by someone other than its author, before it integrates — today, usually via **pull request (PR)** / pull-request workflow with branch protection.

## Why Does It Exist?

Software engineering's definition was *programming + time + other people*. Code review is the practice where all three collide:

1. **Knowledge distribution** — no module should have exactly one person who understands it (the "hit by a bus" factor)
2. **Defect detection** — a second set of eyes catches what tests and compilers can't: wrong assumptions, missing edge cases, security holes
3. **Shared standards** — the review queue is where a team's actual (not documented) quality bar lives
4. **Collective ownership** — reviewed code is *team* code

It's one of the oldest verified practices: Fagan inspections (1976, IBM) found reviews caught ~60% of defects — and studies since consistently show review finds different defect classes than testing.

## What Problem Does It Solve?

ShopEasy without review: the brilliant, terrifying engineer merges unreviewed refactors at will. Speed doubles for six months. Then he resigns — and the team discovers payment logic only he understood, with zero tests, and comments in the commit messages rather than the code. Velocity collapses for a year.

Review is **knowledge insurance paid in 30-minute installments**.

## Layer 1 — Simple Explanation

Code review is a **newspaper editor**: the journalist writes (best they can), the editor checks facts, clarity, and house style — then publishes. The editor isn't smarter than the journalist; they're *not the person who wrote it*, which is the entire point. Fresh eyes see what authorship hides.

## Layer 2 — Engineer's View

**What reviews are good at (and what they're not):**

| Finds well | Finds poorly |
|---|---|
| Design/assumption errors | Memory leaks, race conditions |
| Missing edge cases | Performance under load |
| Security logic flaws (auth checks, injection) | Anything needing runtime observation |
| Readability, naming, maintainability | Bugs that tests catch cheaper |
| Knowledge sharing (always) | |

Conclusion of modern research (Google, SmartBear studies): review value is **sublinear in size** — beyond ~200–400 lines, defect detection per line drops off a cliff. Small PRs aren't a style preference; they're where review mathematically works.

**The review hierarchy of concerns (in order):**

1. *Correctness* — does it do the right thing, including edge cases?
2. *Security* — trust boundaries, injection, authZ
3. *Design* — does it fit the architecture? Is it testable?
4. *Maintainability* — will a stranger understand it in a year?
5. *Style* — this is what **linters/formatters** are for. Humans reviewing formatting is waste.

**Process mechanics that separate healthy from theater:**

- Author self-reviews first (annotate intent; catch your own 20%)
- Reviewer responds within hours, not days — review latency *is* flow time (Kanban again)
- Block on correctness/design; comment nitpicks as optional (`nit:`)
- Automated gates (CI, SonarQube) run *before* humans — never burn human attention on what machines decide
- Small, focused PRs — one logical change each

**Conversational code review** (pairing, mob review) trades asynchronous convenience for speed and depth — XP's original position: review *while writing* is the extreme version.

## Real-World Example (DevOps flavored)

You review pipeline and IaC changes too — and the stakes are the same:

```yaml
# A PR to a release pipeline — reviewer catches:
- task: AzureCLI@2
  inputs:
    script: az keyvault secret show --vault-name prod-kv -n db-password  # ❌ secret into build log
```

- A Terraform PR deleting a database without `prevent_destroy` — review is the last human gate
- A Dockerfile PR switching `COPY . .` before dependency install — review catches the cache-busting build time regression
- Branch policies in Azure DevOps (min reviewers, linked work items, comment resolution) are the encoded review contract

## Common Mistakes

- The 3,000-line "quick fix" PR — approval becomes rubber-stamping
- Nitpick wars about formatting — automate, don't argue
- LGTM-without-reading — worse than no review (false confidence + no knowledge spread)
- Using review as a status/ego arena; blame tone kills psychological safety and with it honest reporting
- No response SLA — PRs rot, authors context-switch, batching follows
- Reviewing the *person* instead of the *code*

## Mental Model

> Code review is the **editor's desk between the writer and the printing press**. The editor's job isn't to rewrite the story — it's to make sure nothing false, unreadable, or dangerous reaches print, and that at least two people understand every page.

## Remember This

1. Review exists for defect finding AND knowledge distribution — the second often matters more
2. Effectiveness collapses beyond ~200–400 line PRs; small batches are the enabler
3. Machines first (CI, linters, SonarQube), humans second — human attention on human questions
4. Review latency is flow time; hours, not days
5. Correctness → security → design → maintainability → (style: automated)
6. IaC/pipeline PRs deserve the same rigor as application code — they can destroy infrastructure

## One Sentence

Code review is the deliberate insertion of a second, non-author brain between a change and the shared mainline — catching what machines cannot while distributing the knowledge that keeps teams resilient.

## Knowledge Check

1. Why does defect detection collapse on large PRs, and what practice fixes it?
2. Which defect classes should reviews *not* be your primary defense for?
3. Why is "knowledge insurance" sometimes worth more than the bugs caught?
4. What should be automated away before a human ever opens the PR?

## Further Reading

- *How Google Does Code Review* — Google's engineering practices documentation (free online)
- SmartBear's "Best Practices for Peer Code Review" (the 200-line study)
- Fagan, M. (1976), "Design and code inspections..."

---

**← Previous:** [Trunk-Based Development](trunk-based.md)
**Next:** [Semantic Versioning](versioning.md) →
**Related:** [Code Review](code-review.md) · [TDD & BDD](tdd-bdd.md) · [Shift Left](shift-left-right.md)
