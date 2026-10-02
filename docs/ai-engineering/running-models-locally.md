# Running Models Locally (Hands On)

## What is it?

The practical page: install a local runner, download a model, serve it, wire your first client — plus the operational checklist. Everything mechanical from the last three pages becomes commands.

## The tools (the landscape, decoded)

| Tool | What it is | Use |
|---|---|---|
| **llama.cpp** | the engine: C++ inference for GGUF on CPU/GPU, every quant | the foundation; embedded/special cases |
| **Ollama** | llama.cpp + model management + OpenAI-compatible API, one command | the default local stack |
| **LM Studio** | GUI desktop app (chat + server) | non-terminal users, quick exploration |
| **vLLM / TGI** | GPU serving engines (batching, throughput) | the server fleet — next page |

## The five-minute start (Ollama)

    ollama pull llama3.1:8b            # downloads the GGUF artifact (~5GB)
    ollama run llama3.1:8b             # chat interactively — tokens/sec on display

    # Serve an OpenAI-compatible API for your own code:
    curl http://localhost:11434/v1/chat/completions \
      -d '{"model":"llama3.1:8b","messages":[{"role":"user","content":"hi"}]}'

That last line is the significant one: **local models speak the same API shape as OpenAI** — your application code is provider-agnostic by default. Swapping local ↔ cloud is a base-URL change (the routing pattern the Cost page builds on).

## What to verify when it "doesn't work" (the triage tree)

    Model won't load?
      → capacity: model size vs VRAM/RAM (Hardware page arithmetic)
    Painfully slow?
      → is it on GPU at all? (ollama ps shows the placement)
      → model too big for the card → spilled to RAM → bandwidth death
      → 4-bit it (Quantization page)
    Quality weird?
      → quantization too aggressive for a small model; or the task needs a bigger model
    Context errors?
      → KV cache + weights must BOTH fit; long contexts need headroom

## The operational checklist (your instincts, applied)

- **Pin versions**: model name + quant + digest, like an image tag — reproducibility
- **Govern artifacts**: models are files — checksum, store in a registry, scan provenance (supply-chain page's rules apply; model files are executable-adjacent trust objects)
- **Resource limits**: tokens/sec and context have real cost — set max context and concurrency deliberately
- **Egress-free privacy is the point**: confirm nothing calls home (audit the tool; local means local)
- **Evaluate**: a small golden-set (Evals page) run per model change — model upgrades are dependency upgrades

## Why this matters beyond the laptop

Every enterprise "AI gateway" decision (self-host vs API) is this page at scale: capacity arithmetic + artifacts + serving economics + privacy posture. You have now personally operated every layer of that decision.

## What came next

One user on one machine is done. Serving a team — throughput, batching engines, SLOs, multi-model fleets — is the next page.

## Remember This

1. Ollama = one-command local stack; llama.cpp the engine underneath; both speak OpenAI-shape APIs
2. Local/cloud swapping = base-URL change — build provider-agnostic from day one
3. Triage: fit (capacity) → placement (GPU?) → format (quant) → quality (eval)
4. Models are pinned, checksummed, governed artifacts
5. "Local means local" — audit, don't assume

## One Sentence

Running models locally has collapsed to downloading a GGUF artifact and one command — and the real work is the operations around it: capacity arithmetic, placement, versioned artifacts, and measured quality.

## Knowledge Check

1. Your Ollama model runs at 2 tokens/s. Walk the triage tree.
2. Why does OpenAI-API compatibility make your app architecture future-proof?
3. Which supply-chain rules from the main track apply to model files?

---

**← Previous:** [Quantization](quantization.md)
**Next:** [Serving at Scale](serving-at-scale.md) →
