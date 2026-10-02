# Probability & Prediction

## What is this about?

Before defining what a model *is*, we need the language models speak internally. Every AI system — from spam filters to ChatGPT — produces **probabilities**, not certainties. This page makes that word concrete with a dice game, because you cannot understand LLMs while "probability" feels like a fog.

## Probability, in one simple idea

**Probability is a number between 0 and 1 that measures how likely something is.**

0 = impossible. 1 = certain. 0.5 = a coin flip.

Roll a normal die. Each face has probability 1/6 ≈ 0.167. The six probabilities *together* form a **distribution** — a complete list of "how likely is each outcome", always summing to 1. Think of it as slicing one whole pizza among all possible outcomes.

## The weather example (read it twice — this is the shape of every AI system)

A weather service doesn't say "it will rain." It says something like:

```text
P(rain)     = 0.70
P(cloudy)   = 0.20
P(sunny)    = 0.10
```

That's a probability distribution over tomorrow's outcomes. Notice what the service is really doing: **based on evidence (today's measurements), it scores every possible future and picks the most likely.**

!!! info "Prediction — the one-sentence version"
    **A prediction is choosing the most probable outcome from a distribution.**

Here is the sentence to memorize, because it will explain ChatGPT in a few pages:

> **An LLM is a machine that, given some text, produces a probability for every possible next piece of text — and then picks one.**

Everything else — tokens, weights, training — is machinery for doing this one job well.

## Why not just... be certain?

Because language (and the world) is genuinely uncertain. Consider predicting the next word:

```text
"I'm going to the ___"
    bank      0.30
    store     0.25
    gym       0.10
    moon      0.02
    ...       (thousands more options, tiny probabilities)
```

There is no *correct* answer — there are better and worse guesses. So the machine's honest output is a score for every option, not a single verdict. When an LLM answers you, it is playing this game maybe once per token, hundreds of times per paragraph.

## The engineer's view — the parts you'll actually use

**1. Sampling — how the "pick" happens.** You don't always take the top option; you usually *sample* — roll a weighted die built from the distribution. This is why the same prompt can produce different answers: the machine is drawing from a distribution, and dice have no memory of the last roll. (Later vocabulary: `temperature` — a dial controlling how adventurous the sampling is.)

**2. Confidence ≠ correctness.** A probability of 0.99 means "internally consistent with my patterns", not "true". A model can be confidently wrong — this is exactly the mechanism behind **hallucination**, and why "it sounded sure" is not evidence. File this now; it pays off on the limits page.

**3. Distributions are the output format of AI.** Spam filter: P(spam)=0.97. Weather: P(rain)=0.7. LLM: P(next token = "bank")=0.3. Different machines, same output *type*. When you build AI features, you are consuming distributions and deciding thresholds — an alerting problem (your SRE instincts apply directly: thresholds, precision vs. recall tradeoffs).

**4. The calibration idea (one paragraph, useful forever):** a good probability system is *calibrated* — when it says 70%, it's right about 70% of the time. You can measure this, plot it, alert on its decay. Uncalibrated confidence is how teams get burned by AI features.

## The bridge to the next page

We said the machine "produces probabilities from evidence." How? What machine, what evidence, and what does it *adjust* to get better? That machine is the **model** — the subject of the next page — and the adjusting is what "learning" means.

## Remember This

1. Probability: 0–1 likelihood; a distribution scores *all* outcomes and sums to 1
2. Prediction = picking from a distribution; LLMs do this per token, hundreds of times per answer
3. Sampling explains non-determinism; temperature is its dial (preview)
4. Confidence ≠ correctness — internally consistent guesses can be wrong (hallucination's mechanism)
5. AI outputs are distributions; consuming them = thresholds and tradeoffs (alert-thinking)

## One Sentence

Probability is a scoring of all possible outcomes that sums to certainty, and prediction is choosing from that score — which is the entire job description of every AI model, LLMs included.

## Knowledge Check

1. Why is "the model was confident" not evidence of correctness? Use the calibration idea.
2. Same prompt, different answers twice — explain via sampling.
3. Your spam filter outputs P(spam). Where have you handled "scores + threshold" professionally?
4. Why is "I'm going to the ___" a probability problem and "2+2" not?

---

**← Previous:** [From Calculators to Learning Machines](from-calculators-to-learning-machines.md)
**Next:** [What Is a Model?](what-is-a-model.md) →
