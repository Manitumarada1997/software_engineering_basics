# Agile

## What Is It?

Agile is a **set of values and principles** for software development under uncertainty, published as the *Manifesto for Agile Software Development* (2001) by 17 practitioners frustrated with heavyweight, plan-driven processes.

```text
We value:
Individuals and interactions        over  processes and tools
Working software                     over  comprehensive documentation
Customer collaboration               over  contract negotiation
Responding to change                 over  following a plan

That is, while there is value in the items on the right,
we value the items on the left MORE.
```

Agile is **not** Scrum, not daily standups, not Jira. Those are implementations. Agile is the underlying stance: when you can't know requirements upfront, win by *shortening the feedback loop*.

## Why Does It Exist?

Waterfall's fatal flaw was one-way flow with feedback only at the end. By the late 1990s, the industry had a graveyard of failed big-bang projects. Parallel experiments — Scrum (Jeff Sutherland, Ken Schwaber), XP (Kent Beck), Crystal (Alistair Cockburn) — all converged on the same discovery:

> **When requirements are uncertain, the winning strategy is to deliver in small increments and correct course using real feedback.**

The 2001 manifesto named what these methods shared.

## What Problem Does It Solve?

ShopEasy with Waterfall, Year 3: a 6-month release plan for "ShopEasy 3.0". At month 5, a competitor ships one-tap checkout. ShopEasy's market has moved; the plan hasn't. The release ships into the wrong world.

With Agile: every 2 weeks, something real ships to users. The competitor's move is noticed in *one iteration*, reprioritized in the *next*. Feedback replaces prophecy.

## Layer 1 — Simple Explanation

Waterfall bets everything on **predicting** the destination. Agile drives at night with headlights: you can only see 200 meters, but you can drive all night making constant small corrections — and you arrive at a destination that actually exists.

## Layer 2 — Engineer's View

**1. Agile is risk management.** Each iteration is a small, cheap experiment: build a slice, measure, adjust. The cost of being wrong is one iteration, not one year. It converts the big integration cliff into a staircase.

```mermaid
flowchart LR
    Plan1[Plan 2 weeks] --> Build1[Build] --> Demo1[Demo/Feedback]
    Demo1 --> Plan2[Plan next 2 weeks] --> Build2[Build] --> Demo2[Demo/Feedback]
    Demo2 --> PlanN[...]
```

**2. Working software is the progress metric.** Documents can be 100% complete and 100% wrong. Running, tested software is the only honest status — this value is exactly what CI/CD industrializes: the pipeline proves "working" on every commit.

**3. The principles most relevant to DevOps:**

- Working software delivered frequently (weeks, not months)
- Business people and developers work together daily
- Sustainable pace — agility dies with burnout
- Continuous attention to technical excellence
- Simplicity — maximizing work *not* done
- Self-organizing teams produce the best architectures

Notice: DevOps is the **logical conclusion** of Agile — if iterations are 2 weeks but deployment takes 3 days of manual risk, deployment becomes the bottleneck, and automating it (CI/CD) is the Agile move. Agile fixed development's loop; DevOps fixed the deployment loop; SRE fixed the operations loop. One story.

## Real-World Example

Spotify-era folklore aside, a concrete pattern: Amazon's "you build it, you run it" (from a 2006 Werner Vogels talk) combined Agile iterations with operational ownership — the direct ancestor of modern DevOps and platform team models. Agile without operational feedback is only half a loop.

## Trade-offs & Common Mistakes

- **Agile ≠ no planning.** It's continuous re-planning. Teams that "do Agile" by abandoning roadmaps are doing chaos.
- **Fake agility:** 2-week sprints but a 6-month fixed scope and no feedback from users = Water-Scrum-Fall.
- **Tool worship:** installing Jira changes nothing; the feedback loop is the point.
- **Technical debt denial:** "we'll refactor later" every sprint. Agile explicitly demands continuous technical excellence — the principle most often ignored.
- **No product owner:** without someone empowered to steer, sprints are just meetings.

## Mental Model

> Agile is **driving with headlights** instead of following a map drawn in daylight for a road that no longer exists. Scrum and Kanban are driving schools; the pipeline is the engine.

## Remember This

1. Agile = values and principles (2001 Manifesto), not rituals or tools
2. Core move: shorten the feedback loop; deliver working software in small increments
3. It is risk management — the cost of being wrong drops to one iteration
4. Working software is the only honest progress metric
5. DevOps is Agile applied to build/deploy/operate — one continuous story
6. Fake agility: fixed scope + sprint theater + no user feedback

## One Sentence

Agile replaces up-front prediction with short, frequent feedback loops built around working software, because under uncertainty the fastest learner wins.

## Knowledge Check

1. Why is "installing Jira" not Agile?
2. Explain how CI/CD is the logical continuation of Agile values.
3. What makes a 2-week-sprint team with a 6-month fixed scope "Water-Scrum-Fall"?
4. Which Agile principle do teams most often ignore, and what does it cause?

## Further Reading

- [Agile Manifesto](https://agilemanifesto.org) — read it; it's 1 page
- *Agile Software Development with Scrum* — Schwaber & Beedle
- *Agile Estimating and Planning* — Mike Cohn

---

**← Previous:** [Waterfall](waterfall.md)
**Next:** [Scrum](scrum.md) →
**Related:** [Kanban](kanban.md) · [XP, Lean & SAFe](xp-lean-safe.md) · [Roadmap](../roadmap.md)
