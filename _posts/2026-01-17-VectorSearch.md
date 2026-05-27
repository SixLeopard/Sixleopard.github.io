---
title: Vector Search
date: 2026-05-30 22:00:00 +1000
categories: [Machine Learning, Embeddings]
tags: [machine learning, llm, RAG, vector search, embeddings, ANN, databases]
---

# 🚧 UNDER CONSTRUCTION 🚧

Will cover:
- Embeddings
- What is Vector Search and why
- HNSW
- IVF-PQ
- Vector Databases

<!--# How Vector Search Actually Works: From Embeddings to HNSW and IVF-PQ

*[Optional subtitle or one-line hook — what will the reader walk away knowing?]*

---

## TL;DR

> [2–4 sentences. State the problem (exact nearest-neighbor search doesn't scale), name the two main approaches you'll cover (graph-based: HNSW; quantization-based: IVF-PQ), and tease the tradeoff between them.]

---

## Introduction: why vector search?

[Open with a concrete scenario — semantic search, recommendation, RAG retrieval, image similarity, dedup. Make the reader feel the problem before introducing the math.]

- The shift from keyword matching to **semantic matching**
- Why brute-force k-NN is O(N · d) per query and breaks at scale
- What "approximate nearest neighbor" (ANN) buys you and what it costs

> *Aside:* mention the recall/latency/memory trilemma early — every later section comes back to it.

---

## Part 1 — Embeddings: turning meaning into geometry

### What an embedding is

[Define an embedding as a learned mapping from input → ℝᵈ where semantic similarity ≈ geometric proximity. Keep it short — assume the reader has heard the term.]

### How they're produced

- Text: sentence-transformers, OpenAI `text-embedding-3-*`, Cohere, BGE, E5, etc.
- Images: CLIP, DINOv2
- Multimodal: SigLIP, CLIP variants
- [Mention domain-specific encoders if relevant to your audience]

### Distance metrics (and why the choice matters)

| Metric | Formula sketch | When to use |
|---|---|---|
| Cosine | `1 − (a·b)/(‖a‖‖b‖)` | Text embeddings, when magnitude is noise |
| Dot product | `a·b` | When magnitude encodes importance (e.g., some recsys models) |
| Euclidean (L2) | `‖a − b‖₂` | Image embeddings, when models are trained with L2 |

> **Gotcha to mention:** if vectors are L2-normalized, cosine ≡ dot product ≡ monotonic in L2. Many "cosine" indexes are actually dot-product indexes under the hood.

### Code snippet idea

```python
# Show generating an embedding and computing similarity in ~10 lines
# e.g., sentence-transformers + numpy cosine
```

---

## Part 2 — Similarity search: the naive baseline

[Explain flat / brute-force search. Show that it's the ground truth ANN is approximating against.]

- Pseudocode: loop over corpus, compute distance, top-k heap
- Complexity: O(Nd) per query, O(N) memory
- When flat is actually fine (small N — under ~100k, or where 100% recall is non-negotiable)

> *Set up the next two sections:* "To go beyond flat, you either **prune the search space** (IVF) or **navigate it cleverly** (HNSW). Often you also **compress the vectors** (PQ). Real systems combine these."

---

## Part 3 — HNSW: navigating a small-world graph

### Intuition

[Use the "skip list in graph form" analogy. Higher layers are sparse highways, lower layers are dense local streets. You enter at the top, greedy-walk toward the query, descend a layer, repeat.]

### How it's built

- Each new vector is inserted at a random max layer (geometric distribution)
- At each layer, connect to `M` nearest existing nodes (with a heuristic for diversity)
- Lower layers get denser; layer 0 contains every vector

### How a query runs

1. Start at the entry point on the top layer
2. Greedy search: hop to the neighbor closest to the query
3. When no neighbor improves, drop down a layer
4. At layer 0, do a beam search of width `ef_search` and return top-k

### The knobs

| Parameter | What it controls | Tradeoff |
|---|---|---|
| `M` | Max neighbors per node | Higher = better recall, more memory |
| `ef_construction` | Build-time search width | Higher = better graph, slower build |
| `ef_search` | Query-time search width | Higher = better recall, slower queries |

### Strengths and weaknesses

- ✅ Excellent recall at low latency, no training step
- ✅ Supports incremental inserts well
- ❌ **Memory hungry** — stores the full graph + full-precision vectors
- ❌ Deletes are awkward (usually tombstoned)
- ❌ Harder to shard than IVF

[Maybe insert a diagram here — layered graph with a query path traced through it.]

---

## Part 4 — IVF-PQ: partitioning + compression

This is really two ideas stapled together. Cover them in order.

### IVF (Inverted File index)

- Run k-means on the corpus to get `nlist` centroids
- Assign each vector to its nearest centroid → that's its "cell"
- At query time, find the `nprobe` nearest centroids and only search those cells

> Key parameter: `nprobe`. Small `nprobe` = fast but lossy. Large `nprobe` → brute force.

### PQ (Product Quantization)

[The compression trick.]

- Split each d-dim vector into `m` subvectors of dim `d/m`
- Run k-means **per subspace** with `k* = 256` centroids → each subvector is now 1 byte (its centroid id)
- A 768-dim float32 vector (3072 bytes) becomes `m` bytes — often a **30–60× compression**
- Distances are computed via precomputed lookup tables (ADC — asymmetric distance computation)

### Putting IVF + PQ together

- IVF prunes *which* vectors you look at
- PQ shrinks *how big* each vector is and *how fast* distance comparisons run
- Result: indexes that fit billions of vectors in RAM that wouldn't otherwise

### The knobs

| Parameter | What it controls |
|---|---|
| `nlist` | Number of IVF cells (rule of thumb: ~√N to 4·√N) |
| `nprobe` | Cells visited at query time |
| `m` | PQ subvectors (must divide `d`) |
| `nbits` | Bits per PQ code (usually 8) |

### Strengths and weaknesses

- ✅ Massive memory savings — billion-scale on a single box
- ✅ Tunable accuracy/speed via `nprobe`
- ❌ **Requires training** on a representative sample
- ❌ PQ introduces quantization error → lower recall than HNSW at the same memory budget
- ❌ Often paired with a re-ranking step using full vectors

[Optional sub-section: **OPQ** — a rotation learned before PQ that improves recall noticeably for ~free.]

---

## Part 5 — HNSW vs IVF-PQ: when to use which

| | HNSW | IVF-PQ |
|---|---|---|
| Recall at low latency | ★★★★★ | ★★★ |
| Memory efficiency | ★★ | ★★★★★ |
| Build time | Medium | Slow (k-means training) |
| Incremental updates | Easy | Awkward |
| Scales to billions | Painful | Designed for it |
| Best for | < 10–100M vectors, recall-critical | 100M+ vectors, memory-bound |

> Real systems often combine them: **HNSW over IVF centroids**, or **IVF-PQ for the coarse pass + full-precision rerank** on a small candidate set.

---

## Part 6 — Vector databases in practice

[This is where you ground the theory. Pick 2–3 to discuss in depth rather than listing everything.]

Candidates to mention:

- **pgvector** — Postgres extension, supports HNSW and IVFFlat. Best when you already live in Postgres.
- **Qdrant** — HNSW-based, strong filtering, written in Rust.
- **Weaviate** — HNSW + hybrid search.
- **Milvus** — multiple index types including IVF-PQ, DiskANN; built for scale.
- **FAISS** — Facebook's library, not a database, but the reference implementation for most of these algorithms.
- **LanceDB / Chroma / Pinecone / Vespa** — [pick what's relevant to your audience]

### What to actually evaluate when choosing

1. **Filtered search** — can you combine vector similarity with metadata predicates without killing recall?
2. **Hybrid search** — BM25 + dense, with proper score fusion (RRF, weighted)
3. **Update patterns** — read-heavy vs write-heavy, batch vs streaming
4. **Operational story** — replication, snapshots, multi-tenancy
5. **Index type availability** — does it actually support the algorithm you need?

---

## Part 7 — [Optional] Worked example

[Show, don't just tell. A short end-to-end snippet — embed a small corpus, build an HNSW or IVF-PQ index with FAISS, query it, compare recall vs flat search.]

```python
# import faiss, numpy as np
# build flat + HNSW + IVFPQ over the same data
# measure recall@10 and latency for each
```

---

## Closing thoughts

[Pull the threads together in 2–3 paragraphs:]

- Vector search isn't magic — it's geometry plus clever data structures
- The interesting work is in the **tradeoffs**, not the algorithms in isolation
- Where the field is heading: learned indexes, on-disk ANN (DiskANN, SPANN), better quantization (RaBitQ, scalar quant), tighter filter integration

---

## Further reading

- Malkov & Yashunin, *Efficient and robust approximate nearest neighbor search using HNSW graphs* (2016)
- Jégou, Douze, Schmid, *Product Quantization for Nearest Neighbor Search* (2011)
- FAISS documentation and wiki
- [Any blog posts or talks that shaped your thinking — credit them]

---
-->