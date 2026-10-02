# Multi-Agent Systems

## What is it?

Multiple specialized agents — a researcher, a coder, a reviewer, an orchestrator — collaborating on tasks: one agent's output is another's input, coordinated by a supervisor or shared state.

## Why (the real justifications)

- **Context isolation**: each agent gets a clean window for its subtask — the researcher's 50k tokens of documents don't pollute the coder's window (context budgeting, one level up)
- **Specialization**: different prompts/tools/models per role — the cheap extractor feeding the expensive reasoner (model routing's organizational form)
- **Parallelism**: independent subtasks run concurrently
- **Separation of duties**: a generator and an adversarial reviewer — structurally different prompts catching what one agent misses

## The coordination patterns

| Pattern | Shape | Notes |
|---|---|---|
| **Supervisor/orchestrator** | one agent decomposes, delegates, integrates | the default; most controllable |
| **Pipeline** | agent A → B → C (fixed) | really a graph (Graphs page) with LLM nodes |
| **Peer-to-peer / debate** | agents critique each other | verification by adversarial review; expensive |
| **Hierarchical** | supervisors of supervisors | org-chart-shaped — Conway applies! |

The supervisor pattern in pseudocode: decompose goal → assign subtasks → collect results → verify each (Evals page) → integrate or reassign. It's a manager of flaky, non-deterministic reports — your people-management instincts, amusingly, apply.

## When NOT to multi-agent (the section that saves you the most)

Reach for multiple agents only when a single agent demonstrably fails for one of the four reasons above — context pollution, specialization, parallelism, duties. Otherwise multi-agent *multiplies* everything: cost (N agents × N calls), latency (coordination round-trips), **compounding error** (each handoff is a non-deterministic transform — 95% per step becomes 60% over five steps), and debugging despair (whose fault was step 3's hallucination — the researcher, the coder, or the supervisor's summary?).

**The industry's experience:** most successful "multi-agent" systems are two or three agents in a supervisor pattern — generator + reviewer being the killer app. Swarms of eight are demos; the complexity rarely pays. Same lesson as microservices-at-15-devs: distribution costs multiply; distribute only past measured ceilings.

## Coordination failure modes

| Failure | Symptom | Mitigation |
|---|---|---|
| Telephone game | distortions compound across handoffs | structured (schema) handoffs; verifiable intermediate artifacts |
| Duplicated/conflicting work | two agents "fix" the same thing | explicit ownership in task specs; shared state with locks |
| Orchestration loops | supervisor reassigns endlessly | the Loops page's caps, at the supervisor too |
| Emergent behavior | agents interact in unscripted ways | constrain channels — agents talk *through the orchestrator*, not peer-to-peer raw |

## The DevOps mapping

| Multi-agent | Your world |
|---|---|
| Supervisor pattern | team lead + specialists — delegation with verification |
| Schema handoffs | contracts between services (API page) |
| Compounding error across hops | latency/error addition across call chains |
| Conway's law for agents | your agent org chart *will* mirror your system's structure |

## Remember This

1. Justified by: context isolation, specialization, parallelism, separation of duties
2. Default pattern: supervisor; killer app: generator + adversarial reviewer
3. Compounding error is the physics: N non-deterministic hops = multiplicative reliability loss
4. Structured handoffs (schemas) between agents — prose handoffs distort
5. Most value is 2–3 agents; swarms multiply cost/latency/debugging faster than capability

## One Sentence

Multi-agent systems specialize and parallelize agent work through supervisor coordination and structured handoffs — valuable exactly when context isolation, specialization, or separation of duties demands it, and expensive multiplication everywhere else.

## Knowledge Check

1. Compute reliability: five handoffs at 95% each — and your mitigation per hop.
2. Design generator+reviewer for PR summaries; what schema travels between them?
3. Which of the four justifications applies to your use case — if none, what's the honest architecture?

---

**← Previous:** [Memory](memory.md)
**Next:** [MCP](mcp.md) →
