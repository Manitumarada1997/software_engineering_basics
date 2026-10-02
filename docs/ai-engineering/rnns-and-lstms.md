# RNNs & LSTMs — Sequence Models

## What is it?

**RNN (Recurrent Neural Network)**: a network that processes a sequence *one step at a time*, carrying a running summary (a "hidden state") forward:

    read "the"  -> update memory
    read "cat"  -> update memory
    read "sat"  -> update memory   (memory now holds: subject=cat, verb=sat)
    predict next word from memory

Unlike an n-gram, the running summary is *compressed context* — "cat" and "feline" reach it as similar embeddings (previous page), so it generalizes. Unlike a fixed window, the summary travels arbitrarily far. **LSTM (1997)** improved the memory cell with gates that decide what to keep and what to forget — making memories survive hundreds of steps instead of dozens.

For a decade (2014–2018) LSTMs *were* language AI: translation, speech, early chatbots.

## Why does it exist?

Embeddings solved word meaning but not *order and context*: "dog bites man" vs "man bites dog" — same words, different meanings. The RNN's contribution: a mechanism for sequence — process in order, accumulate state, let each step be influenced by all previous steps.

## The two fatal limitations (memorize these — they are why Transformers exist)

**1. The vanishing memory.** Each step overwrites the running summary a little. Information from step 1 passes through hundreds of rewrites by step 300 — like a rumor through a chain of people: technically transmitted, practically gone. The technical name is *vanishing gradients* (backpropagation's blame signal shrinks with distance — the same arithmetic). Consequence: **long-range dependencies fail**. The model handling "The trophy didn't fit because *it* was too big" can lose track of the trophy if the sentence got longer.

**2. No parallelism.** Step N needs step N−1's output. Sequences process *serially* — word by word. GPUs, which are parallel machines (Deep Learning page), sit mostly idle. Training on internet-scale text would take months of wall-clock time that no one could afford.

| LSTM limit | Consequence | The fix that comes |
|---|---|---|
| Vanishing memory | long-range relations lost | attention (next page) |
| Serial processing | untrainable at internet scale | transformer's parallelism |

## The DevOps mapping

| RNN/LSTM | Your world |
|---|---|
| Hidden state | a running accumulator — like a pipeline's shared context object |
| Vanishing memory | telemetry through lossy middleware — arrives, degraded |
| Serial bottleneck | a pipeline that can't parallelize — the GPU fleet idling |
| LSTM gates | retention policy — what to keep, compress, drop |

## What came next

Two fixes were needed and both arrived in one idea: a way to reach *directly* back to any earlier word without chains of rewrites (**attention**), and a way to process all words *simultaneously* (the transformer's architecture). Attention is next — arguably the single most important invention in this entire track.

## Remember This

1. RNN = step-by-step processing with a running summary; LSTM = memory with keep/forget gates
2. Vanishing memory (vanishing gradients): long-range information technically passes, practically dies
3. Serial dependency makes GPU-scale training unaffordable — the economic wall
4. These two limits, and nothing else, are why attention + transformers were invented
5. LSTMs still live where data is truly streaming and small — the idea isn't dead, just outscaled

## One Sentence

RNNs gave machines a running memory for sequences, but their memories faded over distance and their step-by-step nature made internet-scale training unaffordable — two walls that attention and transformers were built to break.

## Knowledge Check

1. Explain vanishing memory as "a rumor through a chain of people" — where exactly is the loss?
2. Why does serial processing waste GPUs specifically?
3. Which LSTM limitation is *scientific* and which is *economic*? Why does the distinction matter?

---

**← Previous:** [Word Embeddings](word-embeddings.md)
**Next:** [Attention](attention.md) →
