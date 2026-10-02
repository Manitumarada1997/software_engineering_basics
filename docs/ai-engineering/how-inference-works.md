# How Inference Works

## What is it?

The mechanics of *serving* a model: what happens per token, why generation is memory-bound, and the three optimizations (KV cache, batching, speculative decoding) that make it affordable.

## What one token costs

Generating each token requires one forward pass (Generation page) — the whole Transformer run over the context so far. Two distinct phases, with very different economics:

| Phase | Work | Character |
|---|---|---|
| **Prefill** (read your prompt) | process ALL input tokens — *in parallel* | compute-heavy, fast in wall-clock |
| **Decode** (write the answer) | ONE token per forward pass, strictly serial | **memory-bandwidth-bound** |

Decode is the bottleneck of every LLM service: each step loads essentially *all* the model's weights from memory to produce one token. The speed limit is not FLOPS — it's **how fast memory can feed the processor**. This single fact explains GPU pricing, quantization's value, and "tokens per second" numbers.

    Rule of thumb: decode speed ≈ memory bandwidth ÷ model size
    (weights must stream through per token; bigger model = fewer tokens/second)

## The KV cache — the central optimization

Naive decode re-processes the entire conversation each token. But attention results for *past* tokens never change. So: cache each token's Key/Value vectors (the Attention page's K and V!) and each new step computes only the new token's contribution.

The cost: KV cache grows with context length × batch size — an *active memory* resident next to the weights. Long-context serving isn't limited by the model file but by weights + KV cache fitting in VRAM. (This is why context windows are precious and why serving engines obsess over cache management.)

## Batching — throughput from latency

One request leaves the GPU mostly idle during decode (waiting on memory). Serve *many* requests simultaneously: while request A streams from memory, B and C ride along. **Continuous batching** (requests join/leave the batch mid-flight) is what modern engines (vLLM et al.) built their speed on: 10–50× throughput per GPU. The tradeoff is batch-induced latency — a queueing-theory dial you already own.

## Speculative decoding — draft and verify

Serial decode is the wall. Trick: a *small* model drafts several tokens cheaply; the big model verifies them in one parallel pass; accepted drafts = multiple tokens for one big-model step. Same output quality (the big model still approves everything), less wall-clock. Symptom you'll notice: long matching answers (code, boilerplate) generate much faster than creative prose.

## The DevOps mapping (this whole section is your home turf)

| Inference | Your world |
|---|---|
| Prefill/decode split | cold start vs steady state |
| Memory-bound decode | disk-bound jobs — bandwidth, not CPU, is the ceiling |
| KV cache | per-connection state (like session memory) — sized, bounded |
| Continuous batching | connection pooling / multiplexing |
| Speculative decoding | read replicas serving, primary verifying |

## Remember This

1. Prefill (parallel, compute-bound) vs decode (serial, memory-bandwidth-bound)
2. Tokens/sec ≈ memory bandwidth ÷ model size — the arithmetic behind every serving number
3. KV cache avoids reprocessing history; it grows with context × batch — the real VRAM constraint
4. Continuous batching: throughput from multiplexing (10–50×/GPU)
5. Speculative decoding: small drafts, big verifies — same quality, faster wall-clock

## One Sentence

Inference serving is a memory-bandwidth problem — streaming weights per serially-generated token — attacked by caching attention history (KV cache), multiplexing requests (batching), and pre-drafting tokens (speculation).

## Knowledge Check

1. Why does a 70B model on a 1TB/s card produce roughly 15 tokens/s? Show the arithmetic.
2. Why does long context cost memory beyond the prompt itself?
3. Which optimization explains "code generates faster than poetry"?

---

**← Previous:** [Open & Closed Models](open-and-closed-models.md)
**Next:** [Hardware for AI](hardware-for-ai.md) →
