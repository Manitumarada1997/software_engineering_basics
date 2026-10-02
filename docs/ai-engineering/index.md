# AI Engineering — From Zero to Advanced

A complete track that starts where AI actually started — with the limits of rule-based computing — and walks every evolutionary step to modern LLMs, local inference, RAG systems, agents, and production AI engineering.

!!! quote "The promise"
    By the end you will know **why LLMs exist**, **how they actually work**, **how to run one on your own machine**, and **how to build production AI systems** — explained as one continuous engineering story, from absolute zero.

## How this track teaches

Every page follows the same evolutionary pattern:

```text
PROBLEM → OLD SOLUTION → LIMITATION → NEW IDEA → NEW SOLUTION → NEW LIMITATION → NEXT
```

Every technical word is introduced, defined in one sentence, given an analogy, and only then used. No assumed terminology. Two layers everywhere: **simple** (analogies) and **engineer** (internals, tradeoffs, production reality) — with each AI idea connected back to the DevOps knowledge you already own.

## Section 1 — From Calculation to Learning

How computers stopped following rules and started learning from data.

| Concept | Focus |
|---|---|
| [What Is Intelligence?](what-is-intelligence.md) | Information, knowledge, learning — working definitions |
| [From Calculators to Learning Machines](from-calculators-to-learning-machines.md) | The rule-writer's wall |
| [Probability & Prediction](probability-and-prediction.md) | Distributions — the output format of all AI |
| [What Is a Model?](what-is-a-model.md) | Parameters and weights, without mystique |
| [What Is Learning?](what-is-learning.md) | Loss, gradient descent, overfitting |
| [Neural Networks](neural-networks.md) | Neurons, layers, backpropagation |
| [The Deep Learning Era](deep-learning-era.md) | Data + GPUs + depth; what still failed |

## Section 2 — Language & the Road to LLMs

Every failed approach to language, and why each was necessary.

| Concept | Focus |
|---|---|
| [Teaching Machines Language](teaching-machines-language.md) | Rules, n-grams, and the wall |
| [Word Embeddings](word-embeddings.md) | Meaning as geometry |
| [RNNs & LSTMs](rnns-and-lstms.md) | Sequence memory and its two fatal limits |
| [Attention](attention.md) | Query/Key/Value — the key invention |
| [Transformers](transformers.md) | The architecture every LLM is built from |
| [Scaling Laws](scaling-laws.md) | Capability as a budget line |

## Section 3 — LLM Fundamentals

Inside the machine you use every day.

| Concept | Focus |
|---|---|
| [Tokens](tokens.md) | What an LLM actually reads |
| [How LLMs Generate Text](how-llms-generate.md) | The full loop, step by step |
| [Training an LLM](training-an-llm.md) | Pretraining, cost, data |
| [Alignment](alignment.md) | From predictor to assistant |
| [The Limits of LLMs](limits-of-llms.md) | Hallucination, context, reasoning |
| [Open & Closed Models](open-and-closed-models.md) | The model landscape |

## Section 4 — Running Models Locally

How a file on your laptop becomes a conversation.

| Concept | Focus |
|---|---|
| [How Inference Works](how-inference-works.md) | Forward pass, KV cache, batching |
| [Hardware for AI](hardware-for-ai.md) | VRAM, memory bandwidth |
| [Quantization](quantization.md) | Smaller numbers, smaller files |
| [Running Models Locally](running-models-locally.md) | Ollama, llama.cpp — hands on |
| [Serving at Scale](serving-at-scale.md) | Throughput, latency SLOs |

## Section 5 — AI Engineering Core

Building reliable software on top of non-deterministic models.

| Concept | Focus |
|---|---|
| [Prompt Engineering](prompt-engineering.md) | Instructions that steer |
| [Context Engineering](context-engineering.md) | What goes in the window |
| [Structured Outputs](structured-outputs.md) | JSON, schemas |
| [Tool Calling](tool-calling.md) | The LLM uses your APIs |
| [RAG](rag.md) | Retrieval-augmented generation |
| [Evals](evals.md) | Testing non-determinism |
| [Guardrails & Safety](guardrails.md) | Injection, PII, moderation |

## Section 6 — Agents & Agentic Systems

Systems that plan, use tools, and act.

| Concept | Focus |
|---|---|
| [Agents](agents.md) | The loop: goal → act → observe |
| [Agentic Systems](agentic-systems.md) | Workflows, autonomy, risks |
| [Agent Loops](agent-loops.md) | Termination, retries, control |
| [Graphs](graphs.md) | State machines for agents |
| [Harnesses](harnesses.md) | The system around the model |
| [Memory](memory.md) | Short-term, long-term, poisoned |
| [Multi-Agent](multi-agent.md) | Teams of specialists |
| [MCP](mcp.md) | Standardized tool access |
| [Coding Agents](coding-agents.md) | The evolution of code assistants |

## Section 7 — Production AI Engineering

| Concept | Focus |
|---|---|
| [AI Architecture](ai-architecture.md) | The full production stack |
| [AI Observability](ai-observability.md) | Traces, tokens, quality |
| [AI Security](ai-security.md) | Injection to agent hijacking |
| [AI Cost](ai-cost.md) | Token economics, routing |
| [AI Reliability](ai-reliability.md) | Fallbacks, degradation |
| [Fine-tuning](fine-tuning.md) | LoRA, RLHF, DPO, distillation |
| [LLMOps](llmops.md) | The delivery pipeline for AI |
| [AI Workflow](ai-workflow.md) | Idea → production → improve |
| [AI + DevOps](ai-for-devops.md) | AI in your day job |

## Capstone

[The ShopEasy Assistant](capstone-shopea
sy-assistant.md) — build a production AI system end to end.

---

**← Back:** [Home](../index.md) · **Full track:** [Roadmap](roadmap.md) · **Start:** [What Is Intelligence?](what-is-intelligence.md) →
