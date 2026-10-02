# Scrum

## What Is It?

Scrum is a **lightweight framework** for delivering value in iterative cycles. It structures Agile into concrete roles, events, and artifacts. Created by Jeff Sutherland and Ken Schwaber in the early 1990s; codified in the [Scrum Guide](https://scrumguides.org) (updated periodically — always check the latest).

```text
Scrum = 3 roles + 5 events + 3 artifacts, in iterations called Sprints
```

| Element | What it is |
|---|---|
| **Roles** | Product Owner · Scrum Master · Developers |
| **Events** | Sprint · Sprint Planning · Daily Scrum · Sprint Review · Sprint Retrospective |
| **Artifacts** | Product Backlog · Sprint Backlog · Increment |

## Why Does It Exist?

Agile said *shorten the feedback loop* — but a team needs an operating system to actually run loops. Scrum provides the minimum viable structure: who decides what (Product Owner), who protects the process (Scrum Master), how long a loop is (Sprint), and the ceremonies that force feedback (Review = external feedback, Retrospective = internal feedback).

It's deliberately incomplete — "a framework within which people can address complex adaptive problems" — not a methodology with all answers.

## What Problem Does It Solve?

ShopEasy at 15 developers, trying "just be Agile":

- Priorities change daily via chat messages; nobody knows what's most important
- Features are "80% done" for three weeks straight
- Work arrives mid-sprint and scrambles everything
- The team never improves its own process because nobody stops to inspect it

Scrum adds the two feedback loops Agile needs: **steering** (Review — are we building the right thing?) and **process improvement** (Retro — are we building things right?).

## Layer 1 — Simple Explanation

A Sprint is a **2-week pot with a lid**. You decide what goes in at the start, you don't add more mid-cook, and at the end everyone tastes the result.

- **Product Owner** = decides the recipe (ordered backlog)
- **Developers** = the kitchen team, self-organizing on *how* to cook
- **Scrum Master** = keeps the kitchen sane — removes blockers, guards the rules, coaches
- **Daily Scrum** = 15-min kitchen huddle: what's blocking the burner?
- **Review** = customers taste it
- **Retro** = the team asks: how do we cook better next time?

## Layer 2 — Engineer's View

**The Product Backlog is a priority queue, not a wish list.** One ordered list, one owner. Ordering forces trade-off decisions — the PO's real job is saying *no*.

**"Done" is a technical contract — the Definition of Done (DoD).** A professional DoD includes tests passing, code reviewed, security gates green, deployed to a real environment. Notice: **your CI/CD pipeline is the enforcement mechanism for the DoD.** If "done" includes "quality gate passed," the pipeline makes "done" objective rather than an opinion.

**Sprint = commitments plus cadence.** All system events (planning, review, retro, release) snap to a rhythm. Cadence creates *predictability of process* even when content is unpredictable — which lets release trains, environments planning, and dependent teams coordinate.

**Sprint length is a risk dial.** Shorter sprints = faster feedback, more overhead. Two weeks is the common equilibrium; one week for high uncertainty.

**Scope flexes, quality doesn't.** The team negotiates *what* fits in a sprint; the DoD is never negotiable. Teams that cut tests to hit sprint goals are borrowing from a loan shark.

## Real-World Example (DevOps flavored)

ShopEasy's platform team runs Scrum. Their backlog: "Helm chart for payments service", "Rotate cluster certificates", "Incident postmortem actions". Sprint Review demos the pipeline: "Any repo now gets a CI template in one click." Their DoD: *merged + pipeline green + deployed to staging + docs updated*. The DevOps engineer's role in Scrum is unremarkable and essential — the work is stories like anyone else's, and the pipeline is how "done" gets proven.

## Common Mistakes

- **Scrum theater:** all ceremonies, no empowered PO, no real backlog ordering
- **Scrum Master as project manager** — the role serves the team, doesn't command it
- **Water-Scrum-Fall:** sprints inside a fixed-scope annual plan
- **Zombie sprints:** indefinite sprints with no review feedback from stakeholders
- **Velocity as a performance metric** — it's a planning aid; weaponizing it creates story-point inflation
- **Skipping retros** when busy — that's skipping the improvement loop

## Mental Model

> Scrum is the **operating system that runs the Agile program**: fixed-length time-boxes as the clock interrupt, the backlog as the input queue, the increment as the output, and Review/Retro as the two feedback interrupt handlers.

## Remember This

1. 3 roles (PO, SM, Developers), 5 events, 3 artifacts — that's all of Scrum
2. The PO's job is an *ordered* backlog — priority is a queue, not a label
3. Definition of Done is the quality contract; CI/CD pipelines enforce it objectively
4. Two feedback loops: Review (product) and Retro (process)
5. Scope flexes in a sprint; quality never does
6. Velocity measures planning capacity, never performance

## One Sentence

Scrum turns Agile into a runnable operating system — fixed sprints, one ordered backlog, and mandatory feedback loops — while leaving the engineering practices to the team.

## Knowledge Check

1. Why is an *ordered* backlog more powerful than a list with priority labels?
2. How does a CI/CD pipeline make "Done" objective?
3. What breaks when velocity becomes a performance metric?
4. Why does Scrum define nothing about engineering practices (testing, CI)?

## Further Reading

- [The Scrum Guide](https://scrumguides.org) — the 13-page source of truth
- *Scrum: The Art of Doing Twice the Work in Half the Time* — Jeff Sutherland

---

**← Previous:** [Agile](agile.md)
**Next:** [Kanban](kanban.md) →
**Related:** [XP, Lean & SAFe](xp-lean-safe.md) · [Requirements & Work Items](requirements-work-items.md)
