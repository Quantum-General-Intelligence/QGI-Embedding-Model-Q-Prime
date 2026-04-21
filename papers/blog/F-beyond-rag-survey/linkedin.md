---
title: "Beyond RAG: Six Families of Systems That Try to Replace It, and What They Miss"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "LinkedIn Pulse"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~750
---

If you are running enterprise AI initiatives in 2026, you've probably had this conversation at least twice.

- Year 1: "We need RAG for our policy documents. Let's set up a vector DB."
- Year 2: "The RAG doesn't work on our regulations. Let's add reranking."
- Year 3 (now): "The reranking doesn't fix contradictions. What do we actually need?"

The third conversation is the interesting one. If you're having it, here's what's happening in the research and vendor landscape.

## Six families of systems

Published work in the last 24 months has converged on six broad families that propose to augment or replace classical RAG:

1. **RAG variants** — HyDE, RAFT, Self-RAG, RAGAS evaluation, HippoRAG, GraphRAG Microsoft. Add preprocessing, reranking, or light graph structure.

2. **Knowledge-graph-grounded generation** — LightRAG, GraphRAG Neo4j, Knowledge Graph RAG. Ground generation in a KG alongside vector similarity.

3. **Agent memory architectures** — MemGPT, Letta, mem0, Zep, LangGraph memory. Build structured memory for long-running agents.

4. **Long-context generation** — Gemini 1M+ context, Claude Opus long context. Skip retrieval by stuffing everything.

5. **Specialised embeddings** — task- or domain-specific embeddings (code, legal, medical).

6. **Compositional / quantum NLP** — DisCoCat, QDisCoCat, categorical semantics.

Each family makes progress. None of them, individually, solves the regulated-content problem. A review paper we released today ([QGI-TR-2026-06]) maps the gaps against seven criteria.

## Seven criteria for rule-bearing content

A system must pass all seven. Any "partial" or "no" is disqualifying.

| Criterion | What it means |
|---|---|
| **Polarity sensitivity** | Distinguishes "must" from "must not" |
| **Scope preservation** | Distinguishes "all" from "some" |
| **Deterministic parsing** | Same input → same structured output |
| **Conflict as first-class event** | Contradictions surfaced, not inferred |
| **Immutable provenance** | Answers trace back by schema |
| **Graceful aging** | "What did it say on date D?" is a primitive |
| **Multi-party relationships** | Rules with 3+ entities held atomically |

Running the six families against these seven (abbreviated):

| Family | Polarity | Scope | Deterministic | Conflict | Provenance | Aging | Multi-party |
|---|---|---|---|---|---|---|---|
| RAG variants | ✗ | ✗ | ~ | ✗ | ~ | ✗ | ✗ |
| KG-grounded | ✗ | ~ | ~ | ~ | ✓ | ~ | ~ |
| Agent memory | ✗ | ~ | ✗ | ✗ | ~ | ✓ | ✗ |
| Long context | ✗ | ✗ | ✗ | ✗ | ~ | ✗ | ✗ |
| Specialised embeddings | ~ | ~ | N/A | ✗ | N/A | N/A | N/A |
| QNLP | ~ | ✓ | ~ | ~ | ✗ | ✗ | ~ |
| QAG (ours) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

The table is in the preprint. It's a call to action for each family — there's room to improve along any criterion. It's also, transparently, a positioning of what QAG contributes.

## What this means for your procurement process

If a vendor is pitching you AI for regulated content, ask them which of the seven they satisfy. Make them be specific. If their answer is "our LLM is very smart and will figure it out", that's a no on "deterministic" and usually a no on "polarity".

The three most common vendor failure modes I see:

1. "**We have a knowledge graph**" — usually a property graph that forces pairwise edges. No on multi-party. Often no on conflict.

2. "**We have a long-context window**" — eliminates retrieval bottleneck, does nothing for polarity. No on polarity, scope, deterministic.

3. "**We use a state-of-the-art embedding**" — general-purpose embedding with cosine similarity. No on polarity, scope, conflict, multi-party.

## What a complete stack looks like

The preprint's conclusion is that a complete stack needs four layers:

1. **Deterministic parser** (QNR2, in our case): no LLM in ingestion.
2. **Typed versioned hypergraph** (QHG, in our case): multi-party, signed, conflict-first-class.
3. **Purpose-built embedding** (Q-Prime, in our case): polarity, scope, obligation separable.
4. **Intelligence layer** (HSC, in our case): Born-rule probability over the graph.

Any of these four can be built independently by a sufficiently determined vendor. What nobody but us is shipping (as of April 2026) is all four, integrated, with enterprise audit properties.

## Access

- **Preprint:** arXiv link in header.
- **Five companion preprints** on each layer of QAG: A (engine), B (embedding), C (agent memory), D (Born-rule classifier), E (QHG data model).
- **Technical review** of your current stack against the seven criteria: `contact@qgi.dev`.

---

**Dr. Sam Sammane** is CTO and Founder of Quantum General Intelligence, Inc.
