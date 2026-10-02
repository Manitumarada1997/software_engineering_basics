# Fine-tuning, LoRA & Distillation

## What is it?

Training adjustments to an *existing* model — cheap, targeted, post-pretraining. You meet the vocabulary everywhere; this page makes each term one sentence plus a tradeoff.

## The family, one sentence each

| Technique | What it does | Cost | Use when |
|---|---|---|---|
| **Supervised fine-tuning (SFT)** | continue training on your examples (input→ideal output) | moderate | style, format, domain tone |
| **RLHF / DPO** | train on preferences (chosen vs rejected answers) | high | behavior alignment (Alignment page) |
| **LoRA** (PEFT family) | freeze the giant weights; train *tiny adapter matrices* beside them | ~0.1–1% of full tuning | the default customization method |
| **Distillation** | train a small model on a big model's outputs | one-time | capability into cheap hardware |
| **Synthetic data** | generate training examples with models, curate, train | cheap | scarce real data (quality-audit!)

**LoRA** is the workhorse to actually understand: instead of updating 70B weights, attach small low-rank matrices to each layer ("adapters", megabytes not gigabytes) and train only those. The base model stays frozen — adapters are swappable modules: one base + one adapter per task/domain/customer, hot-swappable at serve time, versionable as small artifacts (your CI/CD instincts apply directly).

**Distillation** is how the open ecosystem gets small-but-capable models: the big model generates (or grades) answers; the small model trains on them — teacher-student, compressing behavior into hardware you can afford. (Its mention on the Open/Closed page, now precise.)

## Fine-tuning vs RAG vs prompting (the decision table)

| Need | Answer |
|---|---|
| new/fresh/private **facts** | **RAG** — always. Knowledge smears (Model page); facts change daily |
| consistent **style/format/behavior** | fine-tune (LoRA) — often prompting + few-shot suffices first |
| narrow task at high volume, cheap | LoRA-tuned small model |
| edge/offline capability | distilled/quantized small model |
| "we have data, might as well tune" | no — evals first; prompting/RAG ceiling measured? |

The industry's hard lesson in one line: **teams reach for fine-tuning when they actually needed retrieval, and for RAG when they needed a better prompt.** Measure the ceiling of the cheaper layer before paying for the next.

## The workflow (it's your pipeline discipline)

```text
curated dataset (provenance!) → train LoRA → eval vs base (golden set — Evals page)
→ promote adapter (versioned artifact) → serve → monitor quality drift
→ retrain triggers like dependency upgrades
```

Data quality governs everything: synthetic data without curation bakes in the teacher's errors; biased datasets ship bias at scale. The dataset is the artifact to review hardest.

## The DevOps mapping

| Concept | Your world |
|---|---|
| LoRA adapters | small config layers over a base image — composable, swappable |
| Adapter versioning | artifact versioning — pinned, evaluated, promoted |
| Distillation | golden-image creation — bake once, run cheap forever |
| Dataset curation | code review, but for examples — provenance mandatory |

## Remember This

1. LoRA/PEFT: freeze the base, train tiny adapters — swappable, versionable customization
2. Distillation: big teacher → small student — capability into affordable hardware
3. Facts → RAG; behavior/format → fine-tune (after prompting's ceiling is measured)
4. The dataset is the artifact to review hardest — provenance, curation, bias
6. Evals gate every promotion, base or adapter — non-determinism never ships unmeasured

## One Sentence

Fine-tuning adapts frozen pretrained models cheaply — LoRA via tiny swappable adapters, distillation via teacher-to-student compression — and the engineering judgment is matching the technique to the need: retrieval for facts, tuning for behavior, and evals gating every artifact.

## Knowledge Check

1. Why does LoRA make per-customer customization operationally feasible?
2. Your team wants fine-tuning for "answers about our product". Push back with what, and why?
3. Trace a synthetic-data pipeline's failure modes from teacher bias to production behavior.

---

**← Previous:** [AI Reliability](ai-reliability.md)
**Next:** [LLMOps](llmops.md) →
