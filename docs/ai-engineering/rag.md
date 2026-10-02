# RAG — Retrieval-Augmented Generation

## What is it?

**RAG** answers questions from *your documents, at question time*: find the relevant chunks, put them in the context window, and have the model answer *from that context*. The model stops being the knower and becomes the *reader*.

## The problem (the Limits page, monetized)

Organizations needed answers from data the model cannot have: private runbooks, fresh tickets, product docs — knowledge that is **private** (never trained on) and **fresh** (changed since training). Fine-tuning fails both: expensive, knowledge smears poorly (Model page), and goes stale daily. The insight: *don't put knowledge in the weights — put it in the window.*

## The pipeline (every RAG system is this shape)

**At ingestion (offline, per document):**

    documents → chunking → embeddings (one vector per chunk) → vector DB index

**At question time (online, per query):**

    question → embedding → similarity search (top-k chunks) → [optionally rerank]
    → chunks into context + "answer only from this context" → generation → cited answer

Every piece is a page you've already read: embeddings and cosine similarity (Embeddings page), the context window (Context page), answer-from-context (Limits page's grounding remedy).

## The components, each with its craft

**Chunking** — split docs into ~200–800-token pieces. Craft: respect structure (headers, paragraphs), overlap edges, and match chunk size to the *questions* asked. Bad chunking splits a table from its header and retrieval finds orphaned halves — garbage into the window.

**Vector DB** — an index for nearest-vector queries (pgvector, Qdrant, Pinecone, OpenSearch): the "find the nearest arrows" engine the Embeddings page promised. Filters (namespace, team, freshness) ride along — tenancy matters from day one.

**Reranking** — first pass retrieves top-50 by vector similarity; a *reranker* model scores query-chunk relevance precisely, keeping the best 5. Cheap accuracy multiplier.

**Citations** — the answer references its chunks. Not decoration: the audit trail that lets users (and evals) verify groundedness.

## The failure modes (why RAG systems disappoint, and the fixes)

| Symptom | Cause | Fix |
|---|---|---|
| Wrong answer, right doc missing | retrieval missed it | reranking, better chunking, hybrid (keyword+vector) search |
| Right docs in window, wrong answer | generation ignored context | "answer only from context" + citations + evals |
| Stale answers | index not refreshed | ingestion pipeline tied to doc changes (CI for knowledge) |
| Cross-document questions | facts split across chunks | bigger-chunk/graph retrieval/summary layers |
| Slow/expensive | oversized context per query | top-k tuning, reranking, cache |

The law: **retrieval quality is the ceiling of answer quality.** The model cannot use a document it wasn't given — debug RAG failures at the retrieval layer first.

## The DevOps mapping (this is why you can build RAG well)

| RAG | Your world |
|---|---|
| Ingestion pipeline | CI for knowledge: docs commit → index update, versioned |
| Vector DB | a specialized index/cache — tenancy, retention, backup |
| Freshness lag | drift — detected and reconciled |
| Retrieval evals | canary + SLO: measure recall on golden questions |
| Citation audit | traceability — the forensic spine |

## What came next

RAG gives knowledge; tool calling gives action. Combine them in a loop that pursues a goal, and you have an **agent** — the next section.

## Remember This

1. RAG = retrieval into context: private + fresh knowledge without retraining
2. Pipeline: chunk → embed → index; then query → search → rerank → ground → cite
3. Retrieval quality is the ceiling: debug retrieval first
4. Ingestion is a pipeline — version it, tie it to doc changes, eval recall
5. Citations are the audit trail; chunking is the silent killer

## One Sentence

RAG answers questions by retrieving relevant chunks of your documents and having the model read them in context — trading the impossible (private, fresh knowledge in weights) for the engineerable (a retrieval pipeline whose quality bounds the answers).

## Knowledge Check

1. Why does fine-tuning fail at "fresh knowledge" where RAG succeeds?
2. Trace "user asks about yesterday's incident" through both pipelines (ingest + query).
3. Your RAG cites a doc that says the opposite of the answer. Which layer failed?

---

**← Previous:** [Tool Calling](tool-calling.md)
**Next:** [Evals](evals.md) →
