# Hardware for AI

## What is it?

The resource model for running models: **VRAM capacity** (can it fit?), **VRAM bandwidth** (how fast does it generate?), and the **CPU/GPU/NPU** tiers — with the arithmetic to size any deployment.

## The two numbers that decide everything

**1. Capacity — does the model fit in memory?**
Weights must be resident (Model page: it's a file of numbers) *plus* the KV cache (previous page):

    VRAM needed ≈ model size + KV cache + working overhead
    model size (bytes) ≈ parameters × bytes-per-parameter
      7B × 2 bytes (fp16)  ≈ 14 GB
      70B × 2              ≈ 140 GB   (multi-GPU territory)
      7B × 0.5 (4-bit)     ≈  4 GB    (quantization — next page)

This is why "7B runs on a laptop, 70B runs on a server, 400B runs on a cluster" — capacity arithmetic, not marketing.

**2. Bandwidth — how fast is generation?**
Decode streams the weights per token (previous page):

    tokens/sec ≈ bandwidth ÷ model size
    e.g. 350 GB/s card ÷ 14 GB model ≈ 25 tok/s (reading speed)

Which is why **VRAM bandwidth**, not FLOPS, dominates consumer-GPU generation speed — and why "VRAM" (not RAM) is the spec that matters: GPUs read their own memory at TB/s-class rates that system RAM cannot match.

## The tiers (and what each is honestly for)

| Tier | Where the model lives | Reality |
|---|---|---|
| **CPU** | system RAM | works (llama.cpp), 3–10× slower — fine for batch/offline, not chat |
| **GPU (discrete)** | VRAM | the standard; the VRAM size *is* the model menu |
| **Multi-GPU / cluster** | sharded across cards | 70B+; tensor parallelism — weights split across cards with cross-talk |
| **NPU / phone SoC** | unified memory | on-device AI — small quantized models, private by default |

**VRAM vs RAM is the rookie mistake**: buying a fast GPU with 8GB VRAM caps you at small models regardless of the 64GB of system RAM sitting unused. The menu is: 8GB → 7B quantized · 16–24GB → 13–30B · 48GB+ → 70B quantized · multi-GPU beyond.

## The economics (your FinOps instincts, ported)

- GPU-hours are the unit: cloud GPU ≈ $1–4/hour; a model serving 24/7 needs utilization to justify it
- Batch throughput (previous page) is the lever: tokens/GPU/hour is the cost line
- Right-size by *measured* tokens/sec at your SLO — the same rightsizing law as CPU/memory requests on K8s, transplanted

## Remember This

1. Capacity: params × bytes/param + KV cache — the fit/no-fit arithmetic
2. Bandwidth: tokens/sec ≈ bandwidth ÷ model size — decode's law
3. VRAM (not FLOPS, not system RAM) is the spec; the VRAM size is the model menu
4. CPU inference works for batch; GPUs for interactive; NPUs for on-device
5. GPU-hours × utilization is the FinOps line; batch is the lever

## One Sentence

Running models is governed by two numbers — whether the weights fit (capacity) and how fast memory can stream them (bandwidth) — which makes VRAM the currency and arithmetic the whole procurement process.

## Knowledge Check

1. Will a 13B fp16 model fit a 24GB card with an 8GB KV cache? Show the arithmetic.
2. Why does 4-bit quantization roughly double generation speed on the same card?
3. Your team buys "a fast GPU, 8GB VRAM" for a 30B model. Diagnose.

---

**← Previous:** [How Inference Works](how-inference-works.md)
**Next:** [Quantization](quantization.md) →
