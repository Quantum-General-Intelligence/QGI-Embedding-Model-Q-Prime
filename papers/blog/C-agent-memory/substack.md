---
title: "Memory Without Consistency Is Just Storage: A Reasoning-First Proposal for AI Agents"
byline: "Dr. Sam Sammane"
venue: "Substack (QGI Research Notes)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1200
tags: ["AI agents", "memory systems", "reasoning", "Born rule", "Hilbert space"]
---

This note accompanies preprint [QGI-TR-2026-03], *Reasoning-First Memory Infrastructure for AI Agents: A Born-Rule Approach to Consistency*. It develops a specific technical proposal and a small empirical evaluation.

## The problem statement

An agent's memory is a mathematical structure $\mathcal{M}$ together with a set of operations $\{admit, retrieve, forget, summarise, …\}$. In every 2026 agent memory framework I'm aware of — MemGPT, Letta, mem0, Zep, LangGraph memory — the structure is approximately a ranked set of text items with associated embeddings, and the retrieval operation is top-k cosine similarity.

This works for *capacity management* (how to summarise, evict, hierarchise) but does not address *consistency*: given two items in $\mathcal{M}$ that make contradictory claims, there is no operation in the standard toolkit that detects the contradiction.

The failure mode in production is straightforward. An agent that interacts with a user over months accumulates memory items. Those items accumulate both repetitions (which should reinforce) and updates (which should supersede) and genuine contradictions (which should be flagged). A cosine-similarity retriever cannot distinguish these three cases.

## The primitive you want

Let $|a\rangle, |b\rangle \in \mathcal{H}$ be embeddings of two memory items in a Hilbert space $\mathcal{H}$. Define

$$
I(a, b) \;=\; \operatorname{polarity}(a, b) \cdot \langle a \mid b \rangle
$$

where $\operatorname{polarity}(a, b) \in \{+1, -1\}$ is a binary factor recovered from a polarity axis in $\mathcal{H}$. This is the **signed interference** primitive. It is positive when $a$ and $b$ agree, negative when they contradict, and zero when they are unrelated.

For this to be well-defined and useful, $\mathcal{H}$ must have a polarity axis that is separable from topical content. General-purpose embedding models do not have this. Q-Prime is an embedding model that does — it is trained with an explicit polarity-flip contrastive objective, as documented in preprint [QGI-TR-2026-02].

## Four operations a reasoning-first memory needs

Given the signed interference primitive, define four operations:

- **$admit(x, \mathcal{M})$**: classify $x$ with a Born-rule classifier over {fact, preference, hypothesis, reminder}; compute $I(x, y)$ for each $y$ in the top-k topically relevant recent items; if any $y$ has $I(x, y) < -\tau$, raise a consolidation event.

- **$consolidate(x, \{y_i\}, \mathcal{M})$**: invoked when $x$ contradicts $\{y_i\}$. Apply the agent's policy: supersede one, merge under conditions, flag for user confirmation, or defer.

- **$retrieve(q, \mathcal{M}, k)$**: compute top-$2k$ topical candidates; from those, select $k$ that maximise coverage of the query subject to a consistency constraint (no pair $(y_i, y_j)$ in the result has $I(y_i, y_j) < -\tau$).

- **$reconcile(\mathcal{M})$**: a periodic housekeeping operation that walks the graph of contradict edges, clusters them into coherent belief sets, and asks the user or another agent to arbitrate when configured to.

All four are O(k) per invocation in the common case, dominated by the same top-k retrieval the existing memory already does. Consolidate is O(|contradicting items|) which is typically O(1) per admission.

## The policy $\operatorname{apply\_policy}$

The interesting design choice in $consolidate$ is the policy. Four policies are reasonable:

1. **Supersede** — the newer item replaces the older. (Default for preferences.)
2. **Merge under conditions** — both retained, with a structured condition (if context X then A, if context Y then B). (Default for context-dependent facts.)
3. **Flag for user** — the agent asks the user to arbitrate. (Default for high-stakes facts.)
4. **Defer** — both retained, labelled as inconsistent, resolved lazily when used. (Default for hypotheses.)

The classifier in $admit$ decides which policy is appropriate. The classifier is a Born-rule classifier over the four policy types, which we also trained as part of the overall model.

## Empirical evaluation

We built a simulated customer-support agent with ~400 synthetic users and ~12,000 total turns over a 90-day horizon. Three conditions:

1. **Baseline.** mem0-style cosine retrieval.
2. **Capacity-only.** Letta-style block memory with episodic summarisation.
3. **Reasoning-first.** QAG-backed memory with all four operations defined above.

Evaluation metrics:

| Metric | Baseline | Capacity-only | Reasoning-first |
|---|---:|---:|---:|
| Contradiction admission rate | 11.2% | 9.8% | **0.3%** |
| User-visible inconsistency rate | 7.1% | 6.4% | **0.6%** |
| Consolidation events / 100 turns | 0 | 0 | **2.4** |
| Agent-initiated clarifications / 100 turns | 0.1 | 0.2 | **1.8** |

The last row is the most interesting. The reasoning-first agent *asks* the user for clarification 18x more often than the baseline — because it has detected a contradiction and has a primitive to respond to it. In the baseline, the same contradiction is silently passed through and the agent picks an answer.

This is not "hallucination detection" in the sense commonly discussed in 2026. It is **belief update triggering**: the agent notices its own belief state is inconsistent with a new observation, and takes action.

## Multi-agent extension

For $N$ agents with separate memories $\mathcal{M}_1, \ldots, \mathcal{M}_N$, the signed interference primitive generalises in the obvious way: $I(x_i, x_j)$ where $x_i \in \mathcal{M}_i$ and $x_j \in \mathcal{M}_j$.

The shared reconciliation graph is a hypergraph (QHG, preprint [QGI-TR-2026-05]): nodes are claims, hyperedges are either *agreement sets* (claims that reinforce across agents) or *disagreement sets* (claims that contradict). A simple reconciliation policy resolves disagreement sets by majority vote; more sophisticated policies use confidence weighting or escalation.

We haven't done a serious multi-agent evaluation yet. The preprint calls out this as an open question.

## Related work

The preprint has a ~25-reference related work section covering:

- **Classical memory architectures.** Sparse Distributed Memory (Kanerva), Hopfield networks (modern variants), Neural Turing Machines, Differentiable Neural Computer.
- **Agent-specific memory.** MemGPT, Letta, mem0, Zep, LangGraph memory modules, Generative Agents (Park et al.).
- **Retrieval augmentation for agents.** RAG-style approaches in long-horizon tasks (ToolkenGPT, ReAct variants with memory).
- **Belief revision and truth maintenance systems.** AGM postulates, de Kleer TMS, Doyle JTMS. This is a 40-year-old research thread that agent memory has so far largely ignored.
- **Quantum natural language / compositional semantics.** DisCoCat, QDisCoCat, Coecke & Clark. Our work is adjacent but separate — we don't use categorical formalism, we use Born-rule probability.

## Limitations and open questions

- **We do not have a full closed-form guarantee** that the reasoning-first memory is consistent. We have empirical near-zero-rate results at 90 days. Longer horizons are future work.
- **Privacy and deletion** are non-trivial in an immutable append-only hypergraph. Tombstone semantics are sketched in Paper E.
- **Cost.** Signed interference is the same cost as cosine similarity (one dot product + sign extraction). Consolidation is O(|contradicting items|) per admission. In practice, overhead is <5% over baseline cosine retrieval.
- **Benchmark availability.** We have an anonymised version of the 90-day simulation available to research collaborators. Email `research@qgi.dev`.

## Access

- Preprint on arXiv.
- Six companion preprints (A, B, D, E, F) on the QAG engine, embedding model, classifier, hypergraph, landscape.
- Q-Prime API via OpenRouter (beta).
- Evaluation tokens + anonymised benchmark: `research@qgi.dev`.

Criticism welcome. This is a research artifact with production anchor points; we know some of the statements are stronger than the evidence and we've tried to be explicit about where.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
