# Agents

## What is it?

The escalation ladder, three rungs:

```text
LLM alone:      prompt → answer                        (one shot)
LLM + tools:    prompt → tool call → result → answer   (one delegation)
AGENT:          goal → [observe → reason → act] × N → goal achieved   (a LOOP)
```

An **agent** is an LLM in a loop with tools: given a *goal* (not a question), it decides steps, executes them via tools, observes results, and adjusts — until done or stopped. You already own every component: the loop is the model calling tools (Tool-calling page) repeatedly, with the goal and all intermediate results riding in the context window (Context page).

## What limitation created agents?

Single-shot LLM applications couldn't do *multi-step work*: "investigate why checkout degraded" needs a *plan* — fetch SLOs, check deploys, read traces, query pods — where step 3 depends on step 2's findings. A one-shot prompt can't adapt mid-flight. The missing piece wasn't intelligence; it was **iteration with feedback**. (A cron job is a loop without intelligence; a one-shot LLM is intelligence without a loop. The agent is the intersection.)

## The loop, anatomized (each part is engineering)

```text
GOAL       "find why checkout p95 breached at 14:00"
OBSERVE    each iteration begins with state: goal + actions taken + results so far
REASON     model plans the next action (sometimes explicitly — plan written to context)
ACT        emit a tool call (structured output) — YOUR code executes it
           (observe the result) → back to REASON
TERMINATE  model emits "done" (with final answer) — or a limit trips
```

Everything outside REASON is your harness code (its own page soon): the loop controller, tool registry, context management, termination conditions, logging. **The model is one function call per iteration; the system around it is the product.**

## What makes agents powerful (and immediately dangerous)

- **Adaptive**: plans change on evidence — no hardcoded decision trees
- **Composable**: same loop + different tools = different agents (ops agent, research agent)
- **Degradable to chaos**: wrong turns compound, loops never end, tools misused, costs explode

The risk ledger (each gets its treatment ahead): infinite loops and runaway cost (loops page: iteration caps, budgets), wrong actions (harness page: approval gates), injection through tool results (guardrails: labeled channels), irreproducibility (observability: full traces).

## The DevOps mapping (agent = a reconciler with judgment)

| Agent part | Your world |
|---|---|
| The loop | a reconciliation loop — desired (goal) vs observed, converging |
| Plan-act-observe | canary → measure → decide — the deployment loop's shape |
| Termination limits | job deadlines, `activeDeadlineSeconds` |
| Tool registry | a service catalog with RBAC |
| Harness | the platform around a workload — the difference between a process and a product |

## What came next

A loop with one goal is powerful but shapeless — production needs *control*: termination discipline, structured workflows, memory that persists, and sometimes multiple cooperating agents. Those are exactly the next five pages.

## Remember This

1. Agent = LLM + tools + loop; goal-directed, self-adjusting, multi-step
2. Created because one-shot apps can't adapt mid-task; iteration, not intelligence, was missing
3. The model reasons once per iteration; the surrounding system is the real engineering
4. Power and danger share a source: adaptive behavior without pre-written paths
5. Agent = reconciler with judgment — your loop-debugging instincts apply directly

## One Sentence

An agent is an LLM in a goal-driven loop — observing state, reasoning about the next action, and acting through tools until the goal is met — with the engineering living in the loop, limits, and tools around the model rather than in the model itself.

## Knowledge Check

1. Which ingredient of the agent loop was missing from one-shot LLM apps, exactly?
2. Trace "investigate checkout degradation" through five iterations, naming tool calls.
3. Why is "the model is one function call per iteration" the right mental model for engineers?

---

**← Previous:** [Guardrails & Safety](guardrails.md)
**Next:** [Agentic Systems](agentic-systems.md) →
