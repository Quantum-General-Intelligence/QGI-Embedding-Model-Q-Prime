---
title: "QAG: A reasoning-first replacement for RAG, running real quantum formalism on classical GPUs"
venue: "Hacker News (Show HN)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~250
---

We just released the preprint for **Quantum-Augmented Generation (QAG)** — the pipeline we've been building at QGI to replace RAG for rule-bearing content (regulations, policies, contracts, clinical guidelines, and the memory of long-running AI agents).

The pitch: cosine similarity is a **sign-indifferent** similarity. It can't tell "must" from "must not", or agreement from contradiction. For regulated content that's a non-starter. On our regulatory-conflict benchmark, classical cosine across five widely deployed embedding models scores F1 = 0.000; the QAG interference signal driven by our purpose-built embedding (Q-Prime) scores F1 = 1.000.

What QAG does differently:

1. Deterministic rule parser (no LLM in ingestion).
2. Immutable, versioned hypergraph with *conflict* and *dependency* as first-class edge types.
3. Purpose-built embedding (Q-Prime) that preserves polarity, scope, obligation strength as separable directions.
4. Real quantum formalism — Hilbert space, Born rule, superposition, signed interference — executed on ordinary GPUs. No QPU.

Seven named signals (Relevance, Conflict, Overlap, Redundancy, Coverage, Coherence, Topology), every answer traces to a specific rule in a specific document, full audit replay.

Same primitives address AI-agent memory consistency: MemGPT, Letta, Zep, mem0 all manage *capacity*; none of them model *consistency*. The signed interference signal is that missing primitive.

Preprint (and five companion preprints): [arXiv URL].

Evaluation tokens: `contact@qgi.dev`. Q-Prime on OpenRouter now; QAG engine GA 21 June 2026.

Happy to answer technical questions here or at `sam@qgi.dev`.
