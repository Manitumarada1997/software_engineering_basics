# Graphs — Structured Agent Workflows

## What is it?

Instead of one free-form loop, define the workflow as a **directed graph**: nodes (steps: an LLM call, a tool, a human gate, code) and edges (transitions, possibly conditional). The model still *reasons within nodes*; the graph controls *flow between them*. This is the state machine / workflow-engine idea you've operated forever (CI stages, Argo Workflows, Airflow), with LLM steps as nodes.

## Why loops alone became insufficient

Free loops are maximally flexible and minimally controllable: no guaranteed verification step, no easy "these two things in parallel", no clean place for a human gate, and behavior that differs run-to-run even when the task type is known. But most production tasks have a *known shape* — understand request → (need search? branch) → generate → validate → deliver. When the shape is known, **encode the shape**; leave judgment to the model *inside* steps and *at branch points*.

## A graph, concretely

```text
START
  ↓
[Understand] — LLM classifies: question? incident? request?
  ↓                    ↓                      ↓
 question            incident               request
  ↓                    ↓                      ↓
[Search RAG]      [Fetch metrics]        [Check perms]
  ↓                    ↓                      ↓
 └────────→ [Generate] ←─────────────────────┘
  ↓            (validation failed? → loop back, max 2)
[Validate] — schema + faithfulness check
  ↓ pass                    ↓ fail×2
[Deliver]                [Escalate human]
```

Note what the graph buys: a *guaranteed* validation node; branching by content (LLM decides the edge, code enforces it); a bounded repair loop (generate↔validate, capped); an explicit human escalation node; and **parallelism** where branches are independent.

## The control comparison (graph vs loop)

| | Free loop | Graph |
|---|---|---|
| Predictability of flow | low | high (shape fixed) |
| Where model judgment lives | everywhere | within nodes + at branch decisions |
| Human gates | ad hoc | first-class nodes |
| Verification | optional/hoped | structural |
| Parallel branches | awkward | natural |
| Recovery/resume | manual | checkpoint per node, re-run from node |
| Best for | open exploration | production, known task shapes |

This is *literally* the CI/CD lesson: pipelines (graphs) won for the same reasons. Agents didn't repeal it — loops are for the unknown-shaped minority of tasks; graphs for the engineered majority. Most mature frameworks (LangGraph et al.) converge here: **graphs whose nodes may contain agent loops**.

## The DevOps mapping (it's your existing mental model)

| Graph concept | Your world |
|---|---|
| Nodes + conditional edges | pipeline stages + conditions |
| Bounded repair loop | retry stage / canary-rollback loop |
| Human gate node | approval gate in CD |
| Checkpoint per node | resumable workflow engines (Argo/Airflow) |
| Parallel branches | fan-out stages |

## Remember This

1. Graphs fix flow control; models keep judgment — inside nodes and at branch decisions
2. Structural guarantees: validation nodes, capped repair loops, human gates, parallelism
3. Graphs = pipelines with LLM nodes; loops live *inside* nodes when needed
4. Known task shape → graph; open exploration → loop; most production is graphs
5. Checkpoint per node → resume/replay/debug at step granularity

## One Sentence

Graph-based agent workflows apply pipeline discipline to agentic systems — encoding known task shapes as nodes and conditional edges so verification, approval gates, parallelism, and recovery are structural properties rather than loop-side hopes.

## Knowledge Check

1. Convert the naive "investigate incident" loop into a graph; where do human gates land?
2. Why is a bounded generate↔validate cycle safer in a graph than inside a free loop?
3. Which of your workflows is secretly already this graph?

---

**← Previous:** [Agent Loops](agent-loops.md)
**Next:** [Harnesses](harnesses.md) →
