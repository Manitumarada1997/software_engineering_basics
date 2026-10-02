# Transformers

## What is it?

The **Transformer** (paper: *Attention Is All You Need*, Google, 2017) packaged attention into a complete machine — and, crucially, a **parallel** one. Every LLM you have ever used — GPT, Claude, Llama, Gemini, Mistral — is a stack of Transformer layers.

## The two walls it broke (recap from the RNN page)

| Wall | Transformer's answer |
|---|---|
| Vanishing memory | attention: direct connections, any distance |
| Serial processing | process all tokens simultaneously — the whole input at once, on GPUs |

The second made LLMs possible *economically*: internet-scale training became affordable because GPUs stopped idling. Attention was the scientific fix; parallelism the industrial one. Both were needed.

## The architecture, as a pipeline (you read pipelines fluently)

    token embeddings (+ position information)
       |
       v   each of N layers:
    1. SELF-ATTENTION   -- every token consults every token (previous page)
    2. FEED-FORWARD     -- a small per-token network "thinking about" what was gathered
    3. RESIDUAL + NORM  -- add the input back (skip connection) and normalize
       |
       v
    probability distribution over the next token (Probability page)

The pieces you haven't met, one sentence each:

- **Positional encoding**: attention is order-blind, so each token's position is added into its embedding — order becomes part of the signal (the RNN page's open problem, closed)
- **Feed-forward layers**: attention *gathers*; feed-forward *processes* — the per-token thinking step
- **Residual connections**: each stage adds output to input (refinement, not replacement) — this is what lets stacks go 100+ layers deep without training collapse
- **Layer normalization**: keeps numbers in a healthy range across depth — stabilization

**Encoder vs decoder** (vocabulary you will meet): encoder stacks *read* (build representations — search, understanding); decoder stacks *write* with a **causal mask** — each token attends only backward, so no future leaks into predicting the next token. GPT-style LLMs are decoder-only: pure next-token predictors. (BERT was encoder-only: understanding tasks.)

## Why Transformers changed everything

1. **Parallel training** — internet-scale data became affordable; capability exploded (next page: scaling laws)
2. **One architecture for everything** — text, images (as patches), audio, code, protein structures: anything expressible as token sequences. The 2018 model zoo (one CNN for vision, one LSTM for translation...) collapsed into a single reusable design
3. **Predictable scaling** — bigger model + more data + more compute gives better results, on a curve you can plan around like capacity planning

## The DevOps mapping

| Transformer | Your world |
|---|---|
| Layers | pipeline stages; residuals = each stage refines, never rewrites |
| Causal mask | a security boundary — no future leakage |
| Parallel tokens | fan-out workers; GPU utilization finally earned |
| One architecture everywhere | platform consolidation — snowflakes to platform |

## What problem did this create?

Transformers made enormous models *possible* — the question became *how enormous, and what happens as you scale?* That empirical answer — smooth improvement with emergent surprises — is the Scaling Laws page, the last step before "Large Language Model" finally gets said properly.

## Remember This

1. Transformer = attention + feed-forward + residual/norm, stacked; positional info restores order
2. Its superpower is parallelism — the economic fix behind LLM-scale training
3. Decoder-only (GPT-style) = masked next-token prediction; encoders read, decoders write
4. Residual connections are why depth scales to 100+ layers
5. One token-based architecture ate every domain

## One Sentence

The Transformer packaged attention into a parallel, stackable architecture — turning sequence processing from a serial, memory-fading craft into a scalable industrial pipeline that every modern LLM is built from.

## Knowledge Check

1. Which sub-stage gathers information and which processes it?
2. Why do residual connections permit depth where naive stacking collapses?
3. Explain the causal mask as a security boundary.
4. Why did "one architecture everywhere" matter commercially, not just scientifically?

---

**← Previous:** [Attention](attention.md)
**Next:** [Scaling Laws](scaling-laws.md) →
