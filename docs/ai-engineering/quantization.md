# Quantization

## What is it?

**Quantization** stores each model weight in fewer bits — 16 → 8 → 4 → 3 bits — shrinking the file and often *speeding up* inference, at a small quality cost. It is *the* technique that makes local AI practical.

    7B model:  fp16 → 14 GB   ·  8-bit → 7 GB   ·  4-bit → 3.5 GB
    Same 7B parameters — fewer bytes each. A 70B fits a single 48GB card at 4-bit.

## Why it works (and what it costs)

Weights are decimal numbers; the naive formats use 16 or 32 bits each. Most of that precision is *wasted*: the model's behavior survives coarse rounding because knowledge is smeared across millions of weights (Model page) — no single weight's fourth decimal decides anything. Round them together, carefully, and the *ensemble* keeps behaving.

The cost curve, honestly:

| Format | Size (7B) | Quality | Use |
|---|---|---|---|
| fp16 | 14 GB | baseline | training; serving with VRAM to spare |
| 8-bit | 7 GB | ~indistinguishable | default serving choice |
| 4-bit | 3.5 GB | small measurable loss | consumer hardware, edge |
| 3/2-bit | ~2.6 GB | noticeable | experimentation; small models only |

The engineering law: **quantization degrades small models proportionally more** — a 4-bit 70B loses little; a 4-bit 1B loses a lot. And because decode speed ≈ bandwidth ÷ model size (Hardware page), halving bytes roughly doubles tokens/sec *on the same card* — quantization buys speed as well as space.

## GGUF, briefly

**GGUF** is the file format of the local-AI ecosystem (llama.cpp/Ollama): a single file containing quantized weights + tokenizer + metadata — a model as *one self-contained artifact*. The naming decodes directly: `Q4_K_M` = 4-bit, K-quant method, medium — you now read model filenames like you read Docker tags.

## The tradeoff decision (the page's practical output)

1. Start at the largest quantization your memory allows (8-bit if it fits)
2. Drop to 4-bit for capacity, not for taste — the quality delta is real but small
3. Evaluate on *your* task (Evals page, soon) — "benchmark quality" and "quality on your extraction task" diverge
4. Re-test when you change models *or* quantization — model file changes are dependency upgrades

## The DevOps mapping

| Quantization | Your world |
|---|---|
| Fewer bits per weight | compression with loss — like image codecs for production |
| GGUF file | OCI image — self-contained, versioned, checksummed artifact |
| "Q4_K_M" | a tag with meaning — like `alpine-3.19-slim` |
| Quality vs size curve | the right-sizing curve — measured on *your* workload |

## What came next

You can now read requirements, formats, and tradeoffs. Next page: actually running models locally — the tools, the commands, and the operational checklist.

## Remember This

1. Quantization = fewer bits per weight: smaller file, faster decode, small quality cost
2. It works because knowledge is smeared — ensembles survive rounding
3. 8-bit near-lossless; 4-bit the practical edge; degradation hits small models hardest
4. Speed bonus: half the bytes ≈ double the tokens/sec on the same card
5. GGUF = the self-contained artifact; Q4_K_M is a readable tag; re-eval on change

## One Sentence

Quantization shrinks models by storing weights in fewer bits — trading a little ensemble quality for large gains in memory, speed, and hardware accessibility, with GGUF as the artifact format of the local ecosystem.

## Knowledge Check

1. Why does 4-bit hurt a 1B model more than a 70B model?
2. Your 24GB card must run a 30B model. Choose a format and defend it.
3. Why does quantization *speed up* generation, not just shrink files?

---

**← Previous:** [Hardware for AI](hardware-for-ai.md)
**Next:** [Running Models Locally](running-models-locally.md) →
