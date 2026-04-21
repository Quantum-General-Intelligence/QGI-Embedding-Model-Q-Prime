---
title: "Beyond RAG: A Landscape Review of Reasoning-First Alternatives in 2026"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "QGI Engineering Blog"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
slug: "beyond-rag-landscape-review-2026"
published: "2026-04-21"
length_words: ~1700
tags: ["RAG", "retrieval", "knowledge graphs", "agent memory", "review", "landscape"]
---

RAG turns five this year. The pattern — retrieve chunks by vector similarity, stuff them in a prompt, let an LLM synthesise — has become the default for every enterprise that wants an AI to answer from its own documents. It is also the pattern that fails, specifically and reproducibly, on the documents that matter most: rules, regulations, contracts, policies, and the long-term memory of AI agents.

This post is a high-level landscape review — the short version of our preprint [QGI-TR-2026-06], *Beyond RAG: A Landscape of Retrieval-Augmented and Reasoning-First Alternatives for AI Systems*.

I want to give you a map. Not a sales pitch.

## Six families of systems in 2026

After surveying the last two years of published work in RAG and its neighbours, I've found six broad families of systems that propose to replace or augment RAG:

1. **Classical RAG variants** (HyDE, RAFT, RQ-RAG, Self-RAG, RAGAS evaluation, SelfCheck, HippoRAG, GraphRAG). These stay within the retrieve-and-stuff paradigm but add preprocessing, reranking, self-critique, or lightweight graph structure.

2. **Knowledge-graph-grounded generation** (LightRAG, GraphRAG Neo4j, Knowledge Graph RAG, SubgraphRAG). These ground generation in a knowledge graph, using graph structure alongside vector similarity.

3. **Agent memory architectures** (MemGPT, Letta, mem0, Zep, LangGraph memory, Generative Agents). These build structured memory systems for long-running agents.

4. **Long-context / context-engineering** (Gemini 1M+ context, Claude Opus long context, RoPE extensions, LongLoRA). These bypass retrieval by putting everything in the context window.

5. **Embedding model specialisation** (task-specific embeddings, instruction embeddings, code embeddings). These train embeddings for specific domains or tasks.

6. **Compositional / quantum natural language** (DisCoCat, QDisCoCat, DisCoCirc, categorical compositional semantics). These use categorical or quantum-inspired formalisms for sentence meaning.

Each of these families makes real progress against specific failure modes of vanilla RAG. None of them, individually, solves the full problem of rule-bearing content. The preprint gives a detailed gap analysis against seven criteria. I'll summarise the criteria and the gap below.

## Seven criteria that rule-bearing content demands

A system for rule-bearing content must pass seven tests. A grade of "partial" or "no" on any one is disqualifying for regulatory use and severely limiting for agent memory.

1. **Polarity sensitivity.** Distinguish "must" from "must not". Is a negation detected as different from the assertion?

2. **Scope and quantifier preservation.** Distinguish "all customers" from "some customers". Is quantifier scope recoverable from the representation?

3. **Deterministic parsing.** For the same input, always produce the same structured output. No LLM-in-ingestion.

4. **Conflict as first-class event.** Two rules contradict; the system surfaces this as an event, not an inferred side channel.

5. **Immutable provenance.** Every answer traces back to a specific source, by schema (not by convention in code).

6. **Graceful aging.** Versioning. "What did the rule say on date D?" is a first-class query.

7. **Multi-party relationships.** Rules involve more than two entities. The data model supports this atomically.

Running the six families of systems against these seven criteria:

| Family | Polarity | Scope | Deterministic | Conflict | Provenance | Aging | Multi-party |
|---|---|---|---|---|---|---|---|
| RAG variants | no | no | partial | no | partial | no | no |
| KG-grounded | no | partial | partial | partial | yes | partial | partial |
| Agent memory | no | partial | no | no | partial | yes | no |
| Long context | no | no | no | no | partial | no | no |
| Specialised embeddings | partial | partial | N/A | no | N/A | N/A | N/A |
| QNLP | partial | yes | partial | partial | no | no | partial |
| QAG (ours) | yes | yes | yes | yes | yes | yes | yes |

This table is from the preprint. It's meant to be a call to action for each of the six families — there's room to improve along any of the criteria. It's also, transparently, a positioning of what QAG contributes.

## Gap analysis

Let me spend a moment on each family's strengths and the specific criterion it currently misses.

### 1. Classical RAG variants

- **Strength:** Easy to deploy, compatible with existing infrastructure, material improvements on open-domain QA.
- **Gap:** The retrieval primitive is still vector similarity. Polarity and scope are still conflated. HyDE, RAFT, and self-RAG don't change the primitive.
- **Example:** HippoRAG introduces a cognitively-inspired graph, which helps structural retrieval. But it doesn't add polarity-sensitive primitives.

### 2. Knowledge-graph-grounded generation

- **Strength:** Graph structure captures multi-party relationships. Provenance is first-class.
- **Gap:** Most implementations use property graphs, which force pairwise edges. Polarity is stored as attribute, not as signed semantics. Conflict discovery is ad-hoc.
- **Example:** Microsoft's GraphRAG is excellent for document topology. It's not designed for regulation, and conflict is not a first-class primitive.

### 3. Agent memory architectures

- **Strength:** Sophisticated capacity management (hierarchical summarisation, episodic memory, temporal memory).
- **Gap:** None model consistency. Retrieval is cosine-similarity on general-purpose embeddings, which silently admit contradictions.
- **Example:** Zep's temporal memory is impressive for capacity, but polarity and conflict are still emergent, not primitive.

### 4. Long context

- **Strength:** Eliminates retrieval as a bottleneck for small-to-medium document sets.
- **Gap:** Does not eliminate the embedding-to-attention pathway, which has its own polarity conflation (see the "lost in the middle" literature and its extensions to negation). Also: cost.
- **Example:** Gemini 1M context is game-changing for some use cases, but for a 10,000-rule corpus with a 100-turn agent, it's economically infeasible.

### 5. Embedding model specialisation

- **Strength:** Domain-specific embeddings are well-studied; the community knows how to train them.
- **Gap:** Polarity-sensitive training is rare even in specialised embeddings. Specialisation usually targets domain vocabulary, not the six properties in Paper B.
- **Example:** Code embeddings (CodeBERT, GraphCodeBERT, E5-code) are excellent for code search. They do not solve the rule-negation problem in natural-language rules.

### 6. Compositional / quantum NLP

- **Strength:** Principled compositional semantics; some of it directly uses quantum-formalism machinery.
- **Gap:** Industrial maturity. DisCoCat and descendants are research artifacts; there's no production deployment at enterprise scale, and the pipelines are not designed for the ingestion rate or corpus size of enterprise compliance.
- **Example:** DisCoCat + classical parser gives a polarity-sensitive representation. It scales to tens of thousands of sentences, not tens of millions.

## Where QAG sits

QAG tries to take the best of each family:

- From **KG-grounded generation**: first-class graph structure, with provenance.
- From **agent memory**: operational treatment of memory as a running structure that admits, consolidates, retrieves.
- From **QNLP**: the mathematical primitives (Hilbert space, Born rule, superposition, interference).
- From **specialised embeddings**: a purpose-built embedding (Q-Prime) trained for polarity, scope, obligation, cross-rule dependency.
- From **classical RAG**: deployment pragmatism — it ships as a hosted API, runs on standard GPUs, integrates with existing agent frameworks.

What we add that the literature doesn't currently combine:

- **Deterministic parsing** (QNR2).
- **Immutable versioned hypergraph** (QHG) with signed conflict as first-class edge.
- **Signed interference** as a retrieval primitive.
- **Born-rule classifier** for calibrated zero-shot categorisation.
- **Hilbert Space Compacting** (HSC) as an intelligence layer over the graph.

None of these are radical inventions. The embedding community has the pieces. The KG community has the pieces. The QNLP community has the pieces. QAG is a coherent stack that puts them together, with enterprise-grade ingestion and audit properties.

## Open questions

1. **Multi-modal rule content.** Contracts with embedded tables, clinical guidelines with decision trees, regulations with images. The current QAG stack is text-only. Each of the six families has varying modal support; none addresses rule-bearing multi-modal content.

2. **Crowd-curated regulation graphs.** The legal community is increasingly interested in open regulation graphs (e.g., LEGIT, OpenRegs). How do these connect to a hypergraph data model? How is provenance preserved across crowd-curation?

3. **Agent-to-agent reconciliation at scale.** Multi-agent systems with N=10, N=100 agents. Each has separate memory. How does reconciliation scale?

4. **Formal verification of rule systems.** Moving from "the LLM answers correctly" to "the system provably applies the right rule" is a frontier. Some of the KG-grounded work points in this direction. QAG is a natural substrate (deterministic parse + immutable graph + Born-rule probabilities).

## Why publish a review

Two reasons.

First, **to calibrate expectations**. Reviewers and enterprise-AI decision-makers often treat RAG as monolithic. The families above are different. Some of them address real failure modes. Some of them don't address the ones you need. A map helps.

Second, **to make QAG's positioning legible**. We don't claim to invent the idea that RAG is insufficient for regulated content — that has been said by many researchers. We claim to build the first coherent stack that addresses all seven criteria at once. A landscape review is the honest way to stake that claim.

## Access

- **Preprint** on arXiv: *Beyond RAG: A Landscape of Retrieval-Augmented and Reasoning-First Alternatives*. Includes the full gap analysis, ~45 references, and evaluation methodology.
- **Five companion preprints** on the specific contributions: QAG engine (A), Q-Prime embedding (B), agent memory (C), Born-rule classifier (D), QHG data model (E).
- **Research correspondence:** `research@qgi.dev`. Especially welcome: disagreements with the gap analysis.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
