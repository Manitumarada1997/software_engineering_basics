# LLMOps — The Delivery Pipeline for AI

## What is it?

Your entire CI/CD + platform discipline, re-instantiated for a new artifact class: prompts, retrieval configs, adapters, and harnesses — versioned, gated, deployed, observed. LLMOps isn't a new profession; it's DevOps where the artifact contains non-determinism and the test suite grades instead of asserts.

## What's versioned (the artifact inventory)

| Artifact | Notes |
|---|---|
| Prompts & templates | the config layer — diffs matter (Prompting page) |
| Retrieval configs | chunking, indexes, rerankers — changes alter answers silently |
| LoRA adapters / models | pinned by digest — dependency upgrades, with regression evals |
| Harness code | graphs, tools, guardrails — regular code, regular discipline |
| Eval suites | the golden set grows from incidents — itself versioned |

## The pipeline (your pipeline, with eval gates)

```text
PR: prompt/model/harness change
  → unit tests (harness logic — deterministic, classic)
  → eval suite on golden set (graded thresholds — the gate)
  → policy checks (security, cost ceilings)
  → deploy: canary cohort (A/B) with online evals watching quality
  → full rollout — or auto-rollback on quality drift
  → production: traces, cost telemetry, drift alerts
```

Every stage is a page you've read: the ratchet from Evals, canary from CD, rollback from Reliability, tracing from Observability. The one genuinely new muscle: **changes that pass all tests can still degrade quality** — hence evals in CI *and* online, both gating.

## ModelOps vs LLMOps (the boundary)

- **ModelOps** (training-side): datasets, training runs, adapters — ML engineers' pipeline
- **LLMOps** (serving-side): prompts, retrieval, harness, evals, cost, security — *your* pipeline
They share the registry (adapters/models as versioned artifacts, with provenance — the supply-chain rules apply: checksums, signed models, SBOM-like manifests of data lineage).

## The org reality (Platform page, applied)

The harness-as-platform pattern: one golden-path AI template (gateway client, eval harness, tracing, guardrails wired) that every AI feature inherits — because 40 teams hand-rolling their own LLM plumbing is the snowflake-pipeline disease you already cured once. The AI platform team's metrics: adoption, time-to-first-eval, cost per feature under budget.

## The DevOps mapping (it's not even a metaphor anymore)

| LLMOps | Your pipeline |
|---|---|
| Eval gate in CI | SonarQube quality gate — graded not binary |
| Canary with quality watch | canary with SLO analysis (Argo Rollouts) |
| Prompt/model version pinning | image digests, lockfiles |
| Adapter registry | artifact repository + promotion |
| Drift alerts → freeze | error budget policy — verbatim |
| AI template golden path | platform golden paths |

## Remember This

1. LLMOps = CI/CD for prompts, retrieval, models, and harnesses — evals replace assertions
2. Eval gates in CI (ratchet) AND online (drift) — passing tests ≠ stable quality
3. ModelOps trains; LLMOps serves; they share a provenance-bearing registry
4. Harness-as-platform: one AI golden path; features inherit correctness
5. Your pipeline instincts transfer wholesale — the gates grade instead of assert

## One Sentence

LLMOps is the DevOps pipeline re-instantiated for AI artifacts — prompts, retrieval configs, models, and harnesses versioned and promoted through graded eval gates and quality-watching canaries, delivered by a platform so every feature inherits the discipline.

## Knowledge Check

1. Map your current pipeline's stages onto the LLMOps pipeline; which gate is new?
2. A model upgrade passes the golden set; complaints rise within 48h. What was missing?
3. What belongs in the AI golden-path template? (Nine harness components — which ship by default?)

---

**← Previous:** [Fine-tuning](fine-tuning.md)
**Next:** [AI Workflow](ai-workflow.md) →
