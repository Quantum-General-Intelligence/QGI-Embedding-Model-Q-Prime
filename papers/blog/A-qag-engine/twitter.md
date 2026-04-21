---
title: "Twitter/X thread — Introducing QAG"
byline: "@sam_sammane (placeholder) / @qgidev"
venue: "Twitter / X"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
---

# Thread (12 tweets)

---

**1/**
In every production embedding model I've tested:

`cosine("must disclose", "must NOT disclose")` > 0.95.

Your RAG pipeline is merging contradictions. For regulated content and for agent memory, this is the single failure mode that matters.

We're releasing the fix today. 🧵

---

**2/**
The fix: Quantum-Augmented Generation (QAG).

Paper: [arXiv URL]
Author: me (@sam_sammane)
TL;DR: replace cosine similarity with **signed interference**, which can be positive, negative, or zero.

Positive = agreement. Negative = contradiction. Cosine can't say "negative".

---

**3/**
The math: real quantum formalism, executed on classical GPUs.

Hilbert space — embedding space, already there.
Born rule — P(outcome|ψ) = |⟨outcome|ψ⟩|², gives calibrated probability.
Superposition — a rule asserting obligation+exception+sanction is a genuine linear combination.

---

**4/**
On our regulatory-conflict benchmark (a real corpus of real regulation pairs):

Classical cosine across 5 widely used embedding models (4 orgs):
→ **F1 = 0.000**

QAG interference signal driven by Q-Prime:
→ **F1 = 1.000**

This is categorical, not incremental.

---

**5/**
Four architectural choices in QAG:

① Deterministic rule parser (QNR2, no LLM in ingestion)
② Immutable versioned hypergraph (QHG) with *conflict* as first-class edge
③ Purpose-built embedding (Q-Prime) preserving polarity, scope, obligation
④ Real quantum formalism, classical GPUs

---

**6/**
On top sits HSC — Hilbert Space Compacting.

Seven named signals:
• Relevance
• Conflict (signed!)
• Overlap
• Redundancy
• Coverage
• Coherence
• Topology

Each is a **typed answer**, not a metric. Overlap and Conflict have different shapes because the human actions differ.

---

**7/**
Trust = Determinism + Traceability + Human authority.

Every QAG answer cites a specific rule in a specific section.
Every pipeline step is replayable.
QAG publishes signals, NOT decisions — the human decides.

This is the property regulators ask for. Every one of them.

---

**8/**
The agent-memory angle, which I'm personally most excited about:

MemGPT, Letta, Zep, mem0, LangGraph memory — all manage memory *capacity*. None model *consistency*.

Yesterday's "prefers Thai" + today's "avoiding Thai" are both retrieved. The agent picks one by attention bias.

---

**9/**
QAG-backed agent memory detects the contradiction as a first-class event.

Implementation: on every new memory item, check signed interference with top-k recent. Threshold on the negative half → contradiction event → agent's consolidation policy decides.

Separate paper: QGI-TR-2026-03.

---

**10/**
We're releasing 6 preprints today:

A — The canonical QAG engine paper
B — Why general-purpose embeddings fail
C — Conflict-aware agent memory
D — Born-rule classifier (zero-shot, calibrated)
E — Quantum HyperGraph (QHG) data model
F — Beyond-RAG landscape review

---

**11/**
Full evaluation paper (numerical benchmarks across embedding families, out-of-domain corpora, extractor-agnostic validation, production GPU throughput) is forthcoming — paper G.

That one is released once the evaluation agreement clears the numbers.

---

**12/**
Access:
• Preprints: [arXiv URL]
• Q-Prime via OpenRouter (public beta)
• Evaluation tokens: contact@qgi.dev
• QAG engine GA: 21 June 2026

Criticism welcome, especially from QNLP / compositional semantics folks. `sam@qgi.dev` / `research@qgi.dev`.

Thread ends. /fin
