---
title: "Twitter/X thread — Beyond RAG: 2026 landscape review"
byline: "@sam_sammane / @qgidev"
venue: "Twitter / X"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
---

# Thread (10 tweets)

---

**1/**
RAG is 5 years old this year.

For open-web Q&A, it works. For regulations, contracts, clinical guidelines, agent memory — it visibly breaks.

We surveyed 2024-2026 alternatives. 6 families, 7 criteria. Here's the map. 🧵

---

**2/**
The 6 families:

① RAG variants (HyDE, RAFT, Self-RAG, HippoRAG, MS GraphRAG)
② KG-grounded (LightRAG, KG-RAG, StructGPT)
③ Agent memory (MemGPT, Letta, mem0, Zep)
④ Long context (Gemini 1M, Claude long-context)
⑤ Specialised embeddings (legal/medical/code)
⑥ QNLP (DisCoCat family)

---

**3/**
The 7 criteria any rule-bearing system needs:

① Polarity (must vs must not)
② Scope (all vs some)
③ Deterministic parsing
④ Conflict as first-class event
⑤ Immutable provenance
⑥ Time-travel on regulations
⑦ Multi-party rules held atomically

"Partial" on any = disqualifying.

---

**4/**
Gap analysis (✓ = yes, ~ = partial, ✗ = no):

RAG variants: ✗✗~✗~✗✗
KG-grounded: ✗~~~✓~~
Agent memory: ✗~✗✗~✓✗
Long context: ✗✗✗✗~✗✗
Specialised: ~~N/A✗N/AN/AN/A
QNLP: ~✓~~✗✗~
QAG: ✓✓✓✓✓✓✓

---

**5/**
Notable: KG-grounded is the strongest on provenance, weakest on polarity.

Most industrial KG systems use property graphs → force pairwise edges → multi-party rules lose atomicity.

A hypergraph data model (our QHG) fixes this. Paper E.

---

**6/**
Notable: agent memory frameworks are excellent for CAPACITY (hierarchy, summarisation, temporal).

None model CONSISTENCY — they silently retrieve contradictory memories.

Polarity + signed interference is the missing primitive. Paper C.

---

**7/**
Notable: long-context bypass.

1M context is transformative for some use cases. For a 10K-rule corpus with 100-turn agent, it's economically infeasible AND doesn't solve polarity ("lost in the middle", extensions to negation reliability).

---

**8/**
Notable: QNLP.

DisCoCat family is the only branch with polarity and scope as principled compositional semantics. But production maturity: none. Scales to thousands of sentences, not millions.

QAG borrows formal primitives (Hilbert space, Born rule) with industrial engineering.

---

**9/**
What a complete stack looks like:

① Deterministic parser (no LLM in ingestion)
② Versioned hypergraph (multi-party, signed, conflict-first)
③ Polarity-sensitive embedding
④ Born-rule probability (not cosine-softmax)
⑤ Typed signals, not scalars

Each has precedent. Integration is the contribution.

---

**10/**
Preprint: [QGI-TR-2026-06]

~45 references, per-family critique, open research questions, migration paths for each family.

5 companion preprints (A-E) on each QAG layer.

If your work is in one of the 6 families and I've been unfair, I want to know.

research@qgi.dev.

/fin
