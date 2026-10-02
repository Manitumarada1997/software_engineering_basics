# Evals

## What is it?

**Evals** are the testing discipline for non-deterministic systems: graded test suites over model outputs, run on every change (prompt, model, retrieval, temperature). Classic unit tests assert exact equality; equality is *never* the right standard for sampled text — so the machinery differs, and everything else (gates, CI, regression detection) is your existing discipline transplanted.

## Why normal testing is insufficient (the mechanical reason)

Same input → same output is the contract unit tests rest on (the course's first lesson: determinism). Generation *samples from a distribution* (Probability page) — same prompt, different valid answers. Asserting one exact string rewards luck and punishes valid phrasings. Evals grade *properties* instead:

| Property | Graded how |
|---|---|
| Correctness (facts, decisions) | expected answer match / rubric scoring |
| Faithfulness/groundedness | does the answer follow from the provided context? |
| Format compliance | schema validation (Structured Outputs page) |
| Safety/refusal behavior | red-team cases that must refuse; benign cases that must not |
| Latency/cost | SLO thresholds per test case |

## The eval stack (three layers)

**1. Golden set (offline, versioned):** 50–500 curated input→expected-behavior cases *from your real traffic*. The heart. Graded automatically where possible:

- **Exact/regex/contains** — for formats, IDs, enums
- **Programmatic checks** — "did the JSON parse? is severity in the enum? does the cited chunk exist?"
- **LLM-as-judge** — a model grades answer quality against a rubric. Scalable, imperfect (judge biases: verbosity, self-preference) — calibrate against human graders and sample-audit forever

**2. CI gate:** every prompt/model/retrieval change runs the suite; score thresholds block merges — the ratchet (baseline + no regressions) rather than unreachable perfection.

**3. Online evals (production):** sampled real traffic scored by judges + user signals (thumbs, escalation rates, edit distance on drafts) — drift detection for quality. A/B for model/prompt changes: the canary's non-deterministic cousin.

## The metric trap (precision/recall, AI edition)

A retrieval step needs **recall** (did the right chunk get retrieved?); a safety filter has **false-positive cost** (over-refusal) and **false-negative cost** (harm) — *asymmetrically*. Setting thresholds is a business decision disguised as a technical one — the alerting page's discipline, replayed exactly.

## The workflow that makes it real

    prompt/prompt/model change → run golden suite → score + diff vs baseline
    → regression? block or justify → promote → online evals watch quality drift
    → worst failures become new golden cases (the suite grows from incidents — the postmortem loop)

## The DevOps mapping

| Evals | Your world |
|---|---|
| Golden set | the regression suite — built from real incidents |
| CI gate with thresholds | quality gates in pipelines — SonarQube's model, new domain |
| LLM-as-judge | a flaky-but-calibratable probe — monitored, sampled-audited |
| Online evals | production canary + drift detection |

## Remember This

1. Non-determinism breaks equality-assertions; evals grade properties instead
2. Golden set from real traffic is the heart; grow it from incidents (postmortem law)
3. Grading: deterministic checks first, LLM-judge scaled with calibration + audits
4. Eval gates in CI — ratchet, not perfection; online evals catch drift
5. Thresholds are business decisions (asymmetric error costs) — alert-thinking

## One Sentence

Evals are the CI/CD of non-determinism — versioned golden sets graded by deterministic checks and calibrated model-judges, gating every change and watching production for quality drift.

## Knowledge Check

1. Why does "assert equals" fail here, precisely? What replaces it?
2. Design the gate: prompt change scores 84% vs 86% baseline. Block or pass? What do you need to know?
3. Which failure classes belong in the golden set from day one?

---

**← Previous:** [RAG](rag.md)
**Next:** [Guardrails & Safety](guardrails.md) →
