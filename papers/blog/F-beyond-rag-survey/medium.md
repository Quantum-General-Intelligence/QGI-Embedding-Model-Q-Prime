---
title: "Is Your RAG Pipeline Broken? A 2026 Landscape Review of What Fixes It."
byline: "Dr. Sam Sammane"
venue: "Medium / Towards Data Science"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1000
tags: ["RAG", "survey", "knowledge graphs", "agent memory", "review"]
---

RAG turned five this year. For open-web Q&A, it works. For regulations, contracts, clinical guidelines, and AI-agent memory, it visibly fails, and the research community has spent the last two years generating alternatives. This post is a map.

(It's the short form of our 2026 review preprint, [QGI-TR-2026-06].)

## The six families of work

1. **Classical RAG variants.** HyDE, RAFT, Self-RAG, RAGAS, HippoRAG, GraphRAG Microsoft. Better preprocessing, reranking, self-critique.

2. **KG-grounded generation.** LightRAG, GraphRAG Neo4j, StructGPT. Knowledge graph + vector retrieval.

3. **Agent memory architectures.** MemGPT, Letta, mem0, Zep, LangGraph memory.

4. **Long context.** Gemini 1M, Claude Opus long context, LongLoRA.

5. **Specialised embeddings.** CodeBERT, legal-BERT, task-instruction embeddings.

6. **Compositional/quantum NLP.** DisCoCat family.

If you're building AI for regulated content or agent memory, odds are one of these feels like it should work. Each of them fixes *something*. None of them fix everything.

## The seven-criterion test

A system for rule-bearing content needs all seven of:

1. Polarity sensitivity — tells "must" from "must not".
2. Scope preservation — tells "all" from "some".
3. Deterministic parsing — same input → same output.
4. Conflict as first-class event — surfaces contradictions as events.
5. Immutable provenance — schema-enforced trace.
6. Graceful aging — time-travel on regulations.
7. Multi-party relationships — 3+ entities held atomically.

Here's how the six families score (our gap analysis):

| Family | Polar. | Scope | Deter. | Confl. | Prov. | Age | MP |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| RAG variants | ✗ | ✗ | ~ | ✗ | ~ | ✗ | ✗ |
| KG-grounded | ✗ | ~ | ~ | ~ | ✓ | ~ | ~ |
| Agent memory | ✗ | ~ | ✗ | ✗ | ~ | ✓ | ✗ |
| Long context | ✗ | ✗ | ✗ | ✗ | ~ | ✗ | ✗ |
| Specialised emb. | ~ | ~ | N/A | ✗ | N/A | N/A | N/A |
| QNLP | ~ | ✓ | ~ | ~ | ✗ | ✗ | ~ |
| QAG (ours) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

## The four common vendor failure modes

Compressing all of the above into vendor patterns I see in the wild:

1. **"We have a KG."** Usually a property graph. Forces pairwise edges; multi-party rules are reified as nodes; atomicity is lost.

2. **"We use long context."** Works for small corpora. Doesn't address polarity. Economically infeasible for enterprise-scale rule corpora.

3. **"We use state-of-the-art embeddings."** General-purpose embedding + cosine similarity. Fails the polarity test in ten lines of Python.

4. **"Our LLM is smart enough."** No deterministic parsing, no auditable trace. Good demo; fails compliance.

Each of these is a partial answer. None is a complete one.

## What would a complete stack look like

1. **Deterministic parser** at ingestion (no LLM in ingestion).
2. **Versioned hypergraph** for storage (multi-party, signed, conflict-first-class).
3. **Polarity-sensitive embedding** for the retrieval primitive.
4. **Calibrated probability** (Born rule, not cosine-softmax).
5. **Intelligence layer** that exposes typed signals (relevance, conflict, overlap, coverage), not just scalars.

Our QAG engine is one realisation of this. There could be others. The point of the review is to make the four/five-layer shape legible so the market can evaluate vendors against it.

## The research story

Each of the six families has more to contribute. A partial research agenda:

- **RAG variants:** train a polarity-contrastive reranker on top of existing retrieval. Might close the polarity gap without changing embeddings.
- **KG-grounded:** upgrade from property graphs to typed hypergraphs with signed edges.
- **Agent memory:** add a consistency layer to existing capacity management.
- **Long context:** characterise polarity performance in the "lost in the middle" regime.
- **Specialised embeddings:** add polarity as a supervision signal.
- **QNLP:** scale from research-scale corpora (thousands of sentences) to enterprise-scale (millions).

None of these require starting from scratch. All of them meaningfully narrow the gap.

## The deeper point

RAG is not a failed paradigm. It is a limited paradigm whose limits we are discovering. The thing that replaces it will be a small constellation of primitives — polarity-sensitive embeddings, hypergraph-structured memory, principled probability, deterministic parsing — each of which has precedent in the literature. The integration is the contribution.

Once that integration exists, the question shifts from "can AI answer questions from our documents?" (a capability question) to "can AI answer questions from our documents *consistently, auditably, and correctly*?" (a trust question). The second question is more important.

## Access

- **Preprint:** *Beyond RAG: A Landscape of Retrieval-Augmented and Reasoning-First Alternatives for AI Systems* — arXiv link in the header.
- **Five companion preprints:** on QAG engine, Q-Prime embedding, agent memory, Born-rule classifier, QHG data model.
- **Research conversations:** `research@qgi.dev`. Especially welcome: disagreement with the gap analysis.

---

*Dr. Sam Sammane is CTO and Founder of Quantum General Intelligence, Inc.*
