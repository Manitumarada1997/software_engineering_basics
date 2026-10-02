# Scaling Laws

## What is it?

The empirical discovery (Kaplan et al. 2020; refined by Chinchilla 2022) that model capability improves **smoothly and predictably** as you increase three things together: **parameters** (model size), **data** (training tokens), and **compute**. Log-scale, near-straight-line relationships — like a capacity-planning curve for intelligence.

    10x compute  ->  reliably better loss  ->  reliably better predictions
    "Loss" = the training wrongness score (Learning page) — lower = better at next-token

## Why does it matter?

Because it turned LLM development from science experiment into **engineering program**. Before scaling laws, doubling model size might or might not help; after them, capability became a *budget line*: $X of compute buys roughly Y of quality. Companies could rationally decide to spend $100M on training *because the curve promised what they'd get back*. The entire LLM industry — the models you use daily — exists because the curve held.

## The three laws, plainly

1. **Bigger is better, smoothly**: loss falls predictably with parameters, data, and compute — no cliffs, no magic threshold where intelligence switches on
2. **Balance matters (Chinchilla)**: for a fixed compute budget, model size and data must grow *together* (~20 tokens of data per parameter). Starving a big model of data wastes it — a 70B model trained on too little data loses to a smaller, better-fed one
3. **Emergent abilities** (the caveat): some skills — multi-step arithmetic, instruction-following, in-context learning — appear *weak or absent* in small models and strong in large ones, without being explicitly trained. Smooth averages hide step-changes in specific capabilities. (Whether truly "emergent" or just hard-to-measure is an active debate — hold it loosely.)

## What the laws made possible — the LLM, properly introduced

Scaling laws said: *take the Transformer, train it on a large fraction of the internet's text, with enough parameters, and next-token prediction gets good enough to look like understanding*. That machine — Transformer + web-scale data + compute budget — is the **Large Language Model**. "Large" is not marketing: it's the variable the laws told us to turn up. The next section finally opens it up: tokens, generation, training, alignment.

## The DevOps mapping

| Scaling law | Your world |
|---|---|
| Compute → quality curve | capacity planning: spend predicts capability |
| Chinchilla balance | no point provisioning 64-core nodes for a single-threaded app — balance the fleet |
| Emergent abilities | threshold behaviors under scale (the architecture that "worked" at 10 rps failing at 10k) |
| $/quality as budget line | FinOps before it was cool — unit economics of capability |

## What problem did this create?

Scaling works — but it costs tens of millions, produces a machine that predicts text *beautifully but has no desire to be helpful*, and concentrates power in whoever can pay. Those three problems produce the rest of this track: efficiency research (quantization, distillation — later pages), alignment (the section after next), and open-weight models.

## Remember This

1. Loss falls predictably with parameters + data + compute — capability as budget line
2. Chinchilla: size and data must balance (~20 tokens/parameter) — starved giants lose
3. Emergent abilities: smooth curves, step-changes in specific skills — held loosely
4. LLM = Transformer + web-scale data + compute, justified by the curve
5. The costs it created: money, indifference, concentration — driving the next sections

## One Sentence

Scaling laws showed that model quality rises predictably with parameters, data, and compute in balance — turning "how smart?" into a budget line and making the LLM industry a rational bet.

## Knowledge Check

1. Why did scaling laws change investment decisions rather than just predictions?
2. Your team trains a 70B model on 200B tokens (Chinchilla says ~1.4T). What happens?
3. Give a threshold-at-scale behavior from systems you operate; compare with emergence.

---

**← Previous:** [Transformers](transformers.md)
**Next:** [Tokens](tokens.md) →
