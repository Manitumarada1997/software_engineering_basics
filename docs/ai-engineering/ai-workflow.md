# The AI Engineering Workflow

## What is it?

The end-to-end lifecycle of an AI feature — the AI edition of the SDLC loop (Software Engineering phase) and the Workflow page's delivery loop, assembled from every page in this track.

```text
Idea → Problem Definition → Model Selection → Prompt/Context Design
  → RAG / Tools → Orchestration (workflow/graph/agent)
  → Evaluation → Security → Observability → Deployment
  → Monitoring → Feedback → Improvement ↺
```

## The stages, each one sentence of wisdom

1. **Problem definition**: is AI even needed? (a rule or a script beats a model — the Rules page's ghost haunting every AI project). Define the *job*, the NFRs (latency, quality threshold, cost ceiling), and the failure budget
2. **Model selection**: local vs hosted (Open/Closed table), tier by task — prototype on frontier, earn your way down to cheap
3. **Prompt/context design**: the five-component prompt (Prompting); the context budget (Context engineering) — designed, versioned, from day one
4. **RAG/tools**: knowledge via retrieval (fresh/private), actions via scoped tools — build the ingestion pipeline like CI for knowledge
5. **Orchestration**: lowest sufficient autonomy (Agentic page): LLM-step → workflow → graph → loop → multi-agent, in that order of reluctance
6. **Evaluation**: golden set *before* iteration begins — you cannot improve what you don't grade; CI gate + online drift (Evals)
7. **Security**: the threat walk (Security page) — channels, objectives, scopes, gates, audit
8. **Observability**: tracing from the first prototype — retro-fitting trace IDs is rewriting history
9. **Deployment**: canary with quality-watch; rollback = model/prompt pin (Reliability)
10. **Monitoring**: cost anomalies, quality drift, tool misuse — the three dashboards
11. **Feedback**: failures become golden-set cases (postmortems, verbatim); users' fixes become evals
12. **Improvement**: the loop's ratchet — each cycle raises the floor

## The anti-patterns (each a page's lesson inverted)

- "Start with the API" — no problem definition, no evals: the demo that couldn't
- "Fine-tune it" — before retrieval/prompting ceilings were measured (Fine-tuning page)
- "Make it an agent" — where a workflow with one LLM step sufficed (Agentic page)
- "Ship the prototype" — unversioned prompts, untraced calls, unbounded spend — the thin wrapper
- "Security later" — the injection surface shipped to production on day one

## Where the two tracks merge (the point of this whole site)

```text
SDLC (main track)  →  problem definition, reviews, versions
CI/CD              →  the LLMOps pipeline with eval gates
Platform Eng       →  harness-as-product, AI golden paths
SRE                →  quality SLOs, error budgets, degradation ladders
Security           →  the AI threat model on DevSecOps controls
AI Engineering     →  models, context, agents — this track
```

You are the intersection the industry is short of: engineers who can operate the harness *and* understand the model.

## Remember This

1. The lifecycle is the SDLC with eval gates and quality SLOs — familiar shape, graded gates
2. Order matters: problem → model → prompt/context → retrieval/tools → orchestration → evals → everything else
3. The ratchet: incidents feed golden sets; feedback raises the floor each cycle
4. The five anti-patterns are five skipped pages
5. The two tracks merge in *you* — the engineer fluent in both halves of the stack

## One Sentence

The AI engineering workflow is the classic delivery loop — problem, design, build, gate, deploy, monitor, improve — where every gate grades instead of asserts and every incident feeds the evaluation set that raises the floor.

## Knowledge Check

1. Walk a feature you'd like to build through all twelve stages in one page.
2. Which stage skipping produces the most expensive failure? Defend with a scenario.
3. Which two main-track disciplines does every AI feature silently depend on?

---

**← Previous:** [LLMOps](llmops.md)
**Next:** [AI + DevOps](ai-for-devops.md) →
