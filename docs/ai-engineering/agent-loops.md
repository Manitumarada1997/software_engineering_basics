# Agent Loops — Control Deep-Dive

## What is it?

The loop is where agent dreams meet production reality. This page is the control engineering: termination, retries, budgets, failure handling — why naive loops fail, and the discipline that makes them safe.

## The naive loop and its four deaths

    while not done:
        reason → act (tool call) → observe

1. **The infinite loop**: the model retries a failing tool forever, convinced the next try differs. Token and API bills accrue. Nothing terminates it.
2. **The loop-with-drift**: circling the same failed approach with escalating verbosity — technically "progressing", practically stuck.
3. **The cascading failure**: one bad observation poisons reasoning; every subsequent step builds on the error.
4. **The expensive success**: the goal *was* achievable, at 40× the cost a workflow would have needed.

## The control kit (each control, its death prevented)

| Control | Mechanism | Prevents |
|---|---|---|
| **Max iterations** | hard cap (e.g., 25) — non-negotiable | infinite loop |
| **Budget ceiling** | max tokens / $ per run — the meter, enforced | expensive spirals |
| **Deadline** | wall-clock limit (the Job's `activeDeadlineSeconds`) | slow death |
| **Loop detection** | hash of (tool, args) seen N times → break + rethink prompt | circling |
| **Retry discipline** | limited retries, exponential backoff *with jitter*, idempotent tools | retry storms (Fundamentals page, replayed) |
| **Error-as-context** | tool errors fed back phrased for self-correction; persistent failure → escalate | cascading confusion |
| **Checkpointing** | state persisted per iteration → resume/replay | losing long runs; unreproducible bugs |
| **Graceful termination** | on limits: summarize progress + partial answer + handoff, never silent death | wasted work, dead UX |

The philosophy: **every limit, when hit, should produce information** — a partial result, a diagnosis, a human handoff — not a timeout stack trace.

## The state machine hiding in the loop (bridge to next page)

The loop's state per iteration: `(goal, plan?, history, last result, budget left)`. Transitions: continue / replan / escalate / terminate. Once you draw it, you notice the loop is just a *degenerate graph* — one node with a self-edge. Production systems quickly need *more structure*: branch points, parallel paths, human gates as first-class nodes. That observation — loops are graphs with one node — is precisely the next page.

## The DevOps mapping

| Loop control | Your world |
|---|---|
| Iteration/budget caps | resource limits — cgroups for agents |
| Retry + jitter | the retry-storm law, unchanged since HTTP |
| Checkpoint/resume | durable execution — queue with acks |
| Graceful termination | degradation ladders — partial service over dead silence |
| Loop detection | flap detection in alerting |

## Remember This

1. Four deaths: infinite, drifting, cascading, expensive — every production incident is one of them
2. Caps on iterations, tokens, dollars, and wall-clock — enforced, not advisory
3. Retry discipline: limited, backoff+jitter, idempotent tools
4. Checkpoint per iteration: resume, replay, debug
5. Every limit hit must produce information: partial answer, diagnosis, or handoff

## One Sentence

An agent loop is safe only when wrapped in control engineering — hard caps on iterations and spend, loop detection, disciplined retries, per-iteration checkpoints, and terminations that yield information instead of silence.

## Knowledge Check

1. Match each of the four deaths to its cheapest effective control.
2. Design the error-message policy that lets loops self-correct instead of cascading.
3. Why is "graceful termination with partial results" a product feature, not just an ops nicety?

---

**← Previous:** [Agentic Systems](agentic-systems.md)
**Next:** [Graphs](graphs.md) →
