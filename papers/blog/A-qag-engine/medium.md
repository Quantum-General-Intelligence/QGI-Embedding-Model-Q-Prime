---
title: "RAG Is Not the Right Primitive for Rule-Bearing Text. Here's What Is."
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "Medium / Towards Data Science"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1000
tags: ["RAG", "embedding models", "regulated AI", "agent memory", "Born rule"]
---

If you're an ML engineer who has built a RAG pipeline, you've probably noticed something awkward. The pipeline is great on FAQs. It is great on documentation search. It is great on customer support tickets. Then one day a product manager asks you to make it work on the company's compliance documents, or its contracts, or a long-running agent's memory. And the wheels come off.

The wheels don't come off because the model is small. They don't come off because the chunks are the wrong size. They come off because **cosine similarity is the wrong primitive for rule-bearing content**.

I want to explain why, and what we've been building at QGI to replace it.

## The failure mode in one sentence

In every production embedding model I've tested — OpenAI `text-embedding-3`, Cohere `embed-v3`, BGE, E5, GTE — the cosine similarity between "the broker must disclose material conflicts" and "the broker must not disclose material conflicts" is **above 0.95**. Try it yourself; it takes ten lines of code. These are treated as near-duplicates by every downstream retrieval system. The generator trained to synthesise does exactly that: it synthesises. Confidently. Wrongly.

This is not a tuning problem. The training objective of a general-purpose embedding model rewards "similar things close, dissimilar things far". Negation is a rare event relative to topical variation in open-web text. The model learns to treat "not" as a small perturbation.

Scaling the model larger does not help — it makes the space bigger, leaves the negation delta proportionally the same size. Fine-tuning helps a bit. Writing cleverer prompts helps a bit. Neither closes the gap to the level a compliance team can sign off on.

## The primitive you actually want

What you want is a **signed** signal between two passages. Positive when they agree. Negative when they contradict. Small magnitude when they are unrelated. Cosine similarity, by construction, does not have a sign; it returns only magnitude.

Quantum mechanics has the exact operation you're looking for: the *inner product* of two states in a Hilbert space, which can be positive, negative, or zero. This isn't a metaphor. An embedding space is a Hilbert space. The operation is the dot product before you take the cosine. The quantum-mechanical probability law on top of it — the **Born rule**, $P(\text{outcome}|\psi) = |\langle\text{outcome}|\psi\rangle|^2$ — is how you turn inner products into calibrated probabilities.

The catch: for the signed inner product to be meaningful, your embedding has to place polarity (and scope, and obligation strength) along separable directions in the space. General-purpose embeddings don't. Training objective doesn't reward it.

## Introducing Q-Prime and QAG

**Q-Prime** is a purpose-built embedding model that does. It is 1,536-dimensional, trained so that polarity, quantifier scope, obligation strength, and cross-rule dependency are each separable directions. It's accessed today as a managed API (OpenRouter, enterprise SLA); weights are not distributed.

**QAG** (Quantum-Augmented Generation) is the pipeline that sits on top of Q-Prime:

```
parse → normalise → hypergraph → Hilbert-space signals → hybrid search → grounded answer
```

Every stage has a property ordinary RAG doesn't:

- **`parse`** is a deterministic rule extractor (QNR2). No LLM. Same input, same AST.
- **`normalise`** produces a canonical form of the rule in either CNL (controlled natural language) or DSL.
- **`hypergraph`** stores rules as nodes in an immutable, versioned graph where *conflict* and *dependency* are first-class edge types.
- **`Hilbert-space signals`** is the intelligence layer (HSC). It publishes seven named signals: Relevance, Conflict, Overlap, Redundancy, Coverage, Coherence, Topology.
- **`hybrid search`** fuses symbolic exact match, vector similarity, and HSC re-ranking.
- **`grounded answer`** produces text that traces to a specific rule in a specific section.

And all of it is classical. Runs on ordinary GPUs. The *word* "quantum" refers to the mathematics — Hilbert space, Born rule, superposition, interference — not to a QPU.

## The F1 = 0 vs F1 = 1 result

On QGI's regulatory-conflict benchmark:

- Classical cosine similarity (five models, four organisations): **F1 = 0.000**.
- QAG interference signal driven by Q-Prime: **F1 = 1.000**.

The gap is categorical. The signal the task requires is simply absent from cosine-based retrieval; it is present in Q-Prime's space by design.

Full methodology is held for a forthcoming companion evaluation paper. The numbers above are real but are released under evaluation agreement.

## Beyond compliance: agents

If you build AI agents, the same pathology breaks your memory store. Yesterday's "user prefers Thai" and today's "user is avoiding Thai" are similar in vector space, retrieved together, and the agent picks one by attention bias. Vector retrieval returns *similar* memories; it has no primitive for *consistent* memory.

The signed interference signal is that primitive. When a new memory item is admitted, check it against recent memories — positive interference means reinforce; negative means contradict and trigger consolidation. The companion preprint ([QGI-TR-2026-03]) develops this.

## Try it

- Preprint: [QGI-TR-2026-01] on arXiv (URL in the header).
- Evaluation token: `contact@qgi.dev`.
- Q-Prime via OpenRouter: public beta.
- QAG engine GA: **21 June 2026**.

Criticism welcome. I'm at `sam@qgi.dev`.
