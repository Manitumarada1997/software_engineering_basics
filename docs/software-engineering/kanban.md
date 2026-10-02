# Kanban

## What Is It?

Kanban is a method for **managing flow** — visualizing work, limiting work-in-progress (WIP), and improving continuously. It came from Toyota's manufacturing (Taiichi Ohno, 1950s) and was adapted to knowledge work by David Anderson (~2007–2010).

Where Scrum structures *time* (fixed sprints), Kanban structures *flow*: work is pulled continuously, never batched into iterations.

```text
Scrum:   [2-week box] → [2-week box] → [2-week box]
Kanban:  work flows continuously →→→→→→→→→→→→→→→→→→→
```

## Why Does It Exist?

Not all work fits sprints. Operations work — incidents, support tickets, ad-hoc requests — arrives unpredictably. You cannot put "the production outage" into a sprint plan. A method was needed for *arriving, variable, continuous* work: that's Kanban.

Its manufacturing insight applies perfectly to software: **the bottleneck governs throughput**. Optimizing a non-bottleneck step is theater.

## Layer 1 — Simple Explanation

A Kanban board is a **hospital triage system for work**:

```text
┌─────────┬──────────┬──────────┬────────┬──────┐
│  To Do  │ Analysis │  Doing   │  Test  │ Done │
│         │    [3]   │   [2]    │   [2]  │      │
└─────────┴──────────┴──────────┴────────┴──────┘
```

Columns are workflow stages; cards are work items; bracketed numbers are **WIP limits** — "at most 2 items in Doing at once."

The counter-intuitive rule: **when a column is full, you may not start new work — you must help finish what's stuck.** A road with fewer cars moves faster.

## Layer 2 — Engineer's View

**Core metrics — the flow lens:**

| Metric | Meaning |
|---|---|
| **Cycle time** | Clock time from *start of work* to *done* |
| **Lead time** | From *request* to *done* (includes queue wait) |
| **Throughput** | Items finished per unit time |
| **WIP** | Items in progress right now |

**Little's Law** ties them together:

```text
Cycle time = Work In Progress ÷ Throughput
```

With throughput roughly fixed by team capacity, the *only* way to deliver faster is **less WIP**. This single equation is the mathematical justification for WIP limits — and for everything from small pull requests to small batches in CD. (Remember it; it returns in the CI/CD phase and in Platform Engineering metrics.)

**Kanban's practices (Anderson):**

1. Visualize the workflow (the board)
2. Limit WIP
3. Manage flow (measure cycle time, find bottlenecks)
4. Make policies explicit (what does "Doing" mean? when may a card move?)
5. Improve collaboratively (Theory of Constraints, models)

**Cadence without sprints:** improvement happens through regular service-delivery and replenishment reviews instead of sprint retrospectives.

## Real-World Example (DevOps flavored)

ShopEasy's **SRE/platform team uses Kanban** — the industry-standard pattern (Google SRE explicitly recommends flow-based boards for ops teams):

- Columns: Triage → Investigating → In Progress → Waiting (external) → Done
- Triage SLAs: a Sev-1 incident enters work immediately, interrupting anything else (expedite swimlane)
- WIP limit on "In Progress": 3 for a team of 6 — context-switching during incidents causes *second* incidents
- Cycle-time tracking exposes the real bottleneck: usually "Waiting (external)" — approvals, other teams

If you've worked an Azure DevOps board with swimlanes for expedite items, you've lived this.

## Scrum vs Kanban

| Dimension | Scrum | Kanban |
|---|---|---|
| Cadence | Fixed sprints | Continuous flow |
| Roles | PO, SM, Developers | None prescribed |
| Change during iteration | Discouraged | Expected anytime |
| Key metric | Velocity (planning aid) | Cycle time / flow |
| Best for | Product feature work | Ops, support, variable intake |

Many teams run **Scrumban**: sprint cadence + WIP limits + flow metrics.

## Common Mistakes

- A board without WIP limits is a decorated to-do list — flow management never started
- Mixing work types without swimlanes (features and incidents need different SLAs)
- Optimizing non-bottleneck teams instead of finding *the* bottleneck
- Measuring utilization (busy-ness) instead of flow — 100%-utilized systems have unbounded queues

## Mental Model

> Kanban is a **highway with metering lights**. The WIP limit is the light that lets fewer cars in — so cars already on the road keep moving. Scrum closes the whole road every two weeks and reopens for a new convoy; Kanban never closes it.

## Remember This

1. Kanban manages continuous flow; Scrum manages time-boxed batches
2. Little's Law: cycle time = WIP ÷ throughput → less WIP = faster delivery
3. WIP limits are the mechanism; the board is just visualization
4. Find and fix the *bottleneck* — optimizing anything else is theater
5. Default choice for ops/SRE/platform teams with unpredictable intake
6. High utilization causes queues, not speed

## One Sentence

Kanban visualizes work, limits work-in-progress, and manages flow so that a team facing unpredictable demand still delivers predictably.

## Knowledge Check

1. Your board has 25 cards "In Progress" for 6 engineers. Using Little's Law, what happens to cycle time and what's the fix?
2. Why is Kanban the default recommendation for SRE teams rather than Scrum?
3. Why doesn't raising utilization at a non-bottleneck step increase throughput?
4. What is the practical difference between lead time and cycle time?

## Further Reading

- *Kanban: Successful Evolutionary Change for Your Technology Business* — David J. Anderson
- [Little's Law](https://en.wikipedia.org/wiki/Little%27s_law) — three lines of arithmetic that explain half of engineering operations

---

**← Previous:** [Scrum](scrum.md)
**Next:** [XP, Lean & SAFe](xp-lean-safe.md) →
**Related:** [Agile](agile.md) · [SDLC](sdlc.md)
