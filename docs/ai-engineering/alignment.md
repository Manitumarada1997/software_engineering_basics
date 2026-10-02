# Alignment

## What is it?

Pretraining builds a next-token predictor with no concept of a helpful assistant. **Alignment** is everything done *after* pretraining to turn that predictor into a product: helpful, honest, safe to hand to strangers.

## The problem, concretely

Ask a base (unaligned) model "Why is the sky blue?" and it may answer — or continue with "Why is the grass green? Why do birds sing?" — because on the internet, *lists of questions follow questions*. Base models complete text; they don't serve users. No amount of scale fixes this: it is not a capability gap, it is a *purpose* gap.

## The alignment pipeline (the industry's standard three steps)

**1. Supervised fine-tuning (SFT):** tens of thousands of high-quality example conversations (human-written): question + ideal assistant answer. Cheap (days, not months) — teaches the *format and role*: "you are the assistant; answer like this."

**2. RLHF (reinforcement learning from human feedback):** humans rank candidate answers; a small **reward model** learns to score answer quality; the LLM is nudged (gradient descent again) toward higher-reward outputs. This is where "helpful, harmless, honest" behavior actually comes from — curated human preference, distilled into weights.

**3. Safety fine-tuning & red-teaming:** adversarial prompting (jailbreaks, harmful requests), patched with targeted training data and refusal behavior — an ongoing arms race, not a completed chapter.

**DPO** (direct preference optimization) — the modern simplification of step 2: learn directly from preference pairs without a separate reward model. Same goal, fewer moving parts.

## What alignment costs (the honest ledger)

| Gain | Price |
|---|---|
| follows instructions, converses | trained on preferences, not truth — sycophancy risk |
| refuses harmful requests | over-refusal on legitimate asks |
| consistent persona | RLHF narrows diversity — the "assistant voice" sameness |
| safety guardrails | jailbreakable by construction — a tendency, not a boundary |

The engineering truth: **alignment is behavior shaping, not a guarantee** — which is why production systems add *external* guardrails rather than trusting the model's manners.

## The DevOps mapping

| Alignment | Your world |
|---|---|
| SFT examples | golden-path templates — "this is how we do it here" |
| Reward model | an SLO everyone optimizes toward (same Goodhart risk) |
| Red-teaming | chaos/pentest — adversarial verification |
| Trained refusal is not security | policy-as-code beats training — enforce outside the model |

## What came next

Aligned LLMs became assistants — and their *structural* limits became everyone's daily experience. Naming those limits precisely, and mapping each to its remedy, is the next page.

## Remember This

1. Base models complete text; alignment adds purpose: role, instruction-following, refusals
2. Pipeline: SFT (format) -> RLHF/DPO (preferences) -> safety tuning (arms race)
3. Human preference is the label for alignment, as the next token was for pretraining
4. Sycophancy and over-refusal are built-in costs, not bugs
5. Alignment is shaping, not security — external guardrails remain mandatory

## One Sentence

Alignment turns a text predictor into an assistant by fine-tuning on example conversations and human preferences — buying helpfulness and safety of manner at the price of sycophancy risk and the certainty that manners are not a security boundary.

## Knowledge Check

1. Why can't scale alone turn a base model into an assistant?
2. Why is the reward model "an SLO" — and what Goodhart failure does it share?
3. Why do production systems wrap guardrails around aligned models anyway?

---

**← Previous:** [Training an LLM](training-an-llm.md)
**Next:** [The Limits of LLMs](limits-of-llms.md) →
