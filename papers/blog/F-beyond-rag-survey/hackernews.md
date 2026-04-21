---
title: "Show HN: Beyond RAG — a 2026 review of six families of alternatives graded on seven criteria"
venue: "Hacker News (Show HN)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~250
---

We surveyed published work from 2024–2026 in and around RAG for rule-bearing content. Six families of systems:

1. RAG variants (HyDE, RAFT, Self-RAG, RAGAS, HippoRAG, GraphRAG-MS).
2. KG-grounded generation (LightRAG, GraphRAG-Neo4j, StructGPT).
3. Agent memory (MemGPT, Letta, mem0, Zep, LangGraph memory).
4. Long context (Gemini 1M, Claude long-context, LongLoRA).
5. Specialised embeddings (CodeBERT, legal-BERT, Instructor).
6. Compositional/quantum NLP (DisCoCat family).

Seven criteria any system for rule-bearing content must pass: polarity, scope, deterministic parsing, conflict as first-class event, immutable provenance, graceful aging (time-travel on regulations), multi-party relationships.

Result (detailed in preprint): none of the six families satisfies all seven. QAG — the coherent composition of deterministic parser + versioned hypergraph + polarity-sensitive embedding + Born-rule probability — is the first stack (as of April 2026, to our knowledge) that does.

Preprint: [QGI-TR-2026-06], *Beyond RAG: A Landscape of Retrieval-Augmented and Reasoning-First Alternatives for AI Systems*. ~45 references, gap-analysis table, detailed per-family critique, open research questions.

Five companion preprints: A (QAG engine), B (embedding), C (agent memory), D (Born-rule classifier), E (QHG).

Interested in disagreement. If your work is in one of the six families and my characterisation seems unfair, I want to know.

`research@qgi.dev`.
