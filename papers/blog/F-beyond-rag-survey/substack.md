---
title: "Beyond RAG: A Gap Analysis Against Seven Criteria for Rule-Bearing Content"
byline: "Dr. Sam Sammane"
venue: "Substack (QGI Research Notes)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1200
tags: ["RAG", "review", "knowledge graphs", "agent memory", "benchmark criteria"]
---

This post summarises preprint [QGI-TR-2026-06], *Beyond RAG: A Landscape of Retrieval-Augmented and Reasoning-First Alternatives for AI Systems*. It is a research-facing review; the company-blog companion is more approachable.

## Motivation

Retrieval-Augmented Generation (RAG) has been the dominant paradigm for document-grounded generation since Lewis et al. (2020). It has benefited from steady engineering: better embeddings, better chunking, better reranking, better evaluation. But the training objective of the underlying embedding — contrastive topical similarity — is ill-suited to rule-bearing content, where polarity, scope, obligation strength, and cross-rule dependency are first-order structural features.

A growing set of research lines attempts to address this. This review maps the six most active families and grades them against seven criteria that rule-bearing content requires.

## The six families

1. **Classical RAG variants.** Includes HyDE (Gao et al., 2022), RAFT (Zhang et al., 2023), RQ-RAG, Self-RAG, RAGAS, HippoRAG, SubgraphRAG. Each adds preprocessing, reranking, self-critique, or lightweight graph structure, while preserving the retrieve-and-stuff pattern. The retrieval primitive remains vector similarity.

2. **Knowledge-graph-grounded generation.** Microsoft GraphRAG, LightRAG, Knowledge Graph RAG, SubgraphRAG, StructGPT. These use a graph alongside vector retrieval. Most use property-graph models.

3. **Agent memory architectures.** MemGPT (Packer et al., 2023), Letta (Tan et al., 2024), Generative Agents (Park et al., 2023), mem0, Zep temporal memory, LangGraph memory modules, OpenAI Assistants memory. These address capacity and hierarchy.

4. **Long-context generation.** Gemini (Google, 2024+), Claude Opus 3/4 long context, RoPE-extended models, LongLoRA. These bypass retrieval by enlarging the context window.

5. **Embedding model specialisation.** Task- or domain-specific embeddings (CodeBERT, GraphCodeBERT, E5-code, medical-BERT, legal-BERT, task-instruction embeddings like Instructor). Focus on domain-specific vocabulary, rarely on polarity.

6. **Compositional / quantum natural language.** DisCoCat (Coecke, Clark), QDisCoCat, DisCoCirc, Frame-DisCoCat. Categorical and quantum-formalism approaches to compositional sentence semantics.

Each family has active research output in 2025-2026. The review cites ~45 papers; the preprint has the full list.

## The seven criteria

After extensive interviews with regulated-AI practitioners and review of the failure modes we've observed across >20 deployed pipelines, we converged on seven criteria that any system for rule-bearing content must satisfy:

1. **Polarity sensitivity** — distinguishes proposition from its negation.
2. **Scope and quantifier preservation** — distinguishes universal / existential / specific.
3. **Deterministic parsing** — same input, same structured output.
4. **Conflict as first-class event** — contradictions surfaced, not inferred.
5. **Immutable provenance** — answers trace back by schema.
6. **Graceful aging** — time-travel queries are primitive.
7. **Multi-party relationships** — rules with 3+ entities held atomically.

The criteria are derived from production failure modes, not from theoretical desiderata. Each corresponds to a specific thing that has gone wrong for us or for a customer.

## Gap analysis

Summary table:

| Family | Polar. | Scope | Deter. | Confl. | Prov. | Age | MP |
|---|---|---|---|---|---|---|---|
| RAG variants | ✗ | ✗ | ~ | ✗ | ~ | ✗ | ✗ |
| KG-grounded | ✗ | ~ | ~ | ~ | ✓ | ~ | ~ |
| Agent memory | ✗ | ~ | ✗ | ✗ | ~ | ✓ | ✗ |
| Long context | ✗ | ✗ | ✗ | ✗ | ~ | ✗ | ✗ |
| Specialised emb. | ~ | ~ | N/A | ✗ | N/A | N/A | N/A |
| QNLP | ~ | ✓ | ~ | ~ | ✗ | ✗ | ~ |
| QAG (ours) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

(✓ = satisfied, ~ = partial, ✗ = not satisfied, N/A = not applicable.)

### Detailed notes

- **RAG variants.** None of HyDE / RAFT / Self-RAG / HippoRAG change the retrieval primitive. HippoRAG's cognitively-motivated graph is a genuine addition, but polarity remains unsupervised.

- **KG-grounded.** Microsoft GraphRAG's community-detection-based summarisation is impressive. The underlying graph is a property graph; multi-party relationships are reified as nodes. Conflict is not a first-class edge type.

- **Agent memory.** Letta / MemGPT hierarchical memory is excellent for capacity. The retrieval primitive is cosine similarity on a general-purpose embedding. Consistency is not modelled.

- **Long context.** Gemini 1M is transformative for some use cases. The "lost in the middle" and related phenomena (Liu et al., 2023) indicate attention does not uniformly weight the full context. Extension to paired-rule-negation reliability is unknown (our internal tests: disappointing).

- **Specialised embeddings.** Domain specialisation closes some of the topical-similarity gap but rarely addresses polarity. We tested legal-BERT on rule/negation pairs; the gap is similar to general-purpose embeddings.

- **QNLP.** DisCoCat and successors have principled compositional semantics. They struggle on enterprise-scale corpora (millions of sentences). Production deployment is minimal.

## Where QAG fits

QAG combines elements from all six families:

- Graph structure and provenance from KG-grounded (upgraded to hypergraph).
- Memory operations from agent memory (with consistency added).
- Formal primitives from QNLP (Hilbert space, Born rule).
- Specialisation from embedding literature (Q-Prime, specifically for polarity, scope, obligation).
- Deployment pragmatism from classical RAG (hosted API, standard GPUs, integration with existing frameworks).
- A deterministic parser (QNR2) as the ingestion step.

The contribution is coherent composition, not radical invention. Each layer has precedent; the stack is unusual.

## Open research questions

1. **Multi-modal rule content.** Contracts with tables; guidelines with decision trees. Current QAG is text-only. A natural extension would pair QHG's typed hyperedges with multi-modal source spans.

2. **Open regulation graphs.** LEGIT, OpenRegs, and related community efforts. How does a community-curated graph interact with a hypergraph schema? Provenance maintenance across crowd-curation is non-trivial.

3. **N-agent reconciliation.** Multi-agent systems with N=10+ agents, each with separate memory. Signed interference gives pairwise reconciliation; transitive reconciliation across N agents needs further theory (likely fits into QHG naturally — conflict hyperedges with 3+ participants).

4. **Formal verification.** Moving from "the system answers correctly" to "the system provably applies the right rule". Deterministic parse + immutable graph + Born-rule probabilities are a substrate; proving compliance is an open research agenda.

5. **Crossover with SQL-ish query languages.** Policy queries are structurally queries over a hypergraph with probabilities. Unifying this with a SQL-like interface (cf. SPARQL for RDF, Cypher for property graphs) is pragmatic future work.

## Closing

The preprint has ~45 references. It argues that no single family currently satisfies all seven criteria, and that QAG's contribution is the coherent combination rather than any one component.

I expect disagreement, and I expect some of the disagreement to be justified. If your work falls into one of the six families and my characterisation is unfair, I want to know. `research@qgi.dev`.

## Access

- Preprint: [arXiv URL].
- Companion preprints A–E on the specific QAG components.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
