---
title: "Entanglement in Regulation: What Happens When You Take the Born Rule Seriously in NLP"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "Substack"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1200
tags: ["quantum NLP", "compliance AI", "embedding models", "agent memory", "research"]
---

I've spent the last two years building something I believe most of machine learning is going to have to take seriously in the next five: a production-grade NLP pipeline whose core primitive is the Born rule, not cosine similarity. Today my team at QGI released the technical preprint — QGI-TR-2026-01 — along with five companion papers. This is the researcher-oriented companion to the release. If you've followed the DisCoCat line, or Bruza/Busemeyer's quantum cognition, or CTO-Coecke's lambeq, this will be familiar ground. If not, I'll try to make the pitch accessible.

## The thesis

A standard dense-retrieval pipeline embeds every passage as a vector in some space and retrieves by cosine similarity. The pipeline has one property that is load-bearing and one property that is load-breaking. The load-bearing property is that it works on prose. The load-breaking property is that **cosine similarity is sign-indifferent**: it cannot tell agreement from contradiction, it cannot separate "all" from "some", and it represents obligation strength as a smear along the same direction as topical content.

For the open web, this doesn't matter very much. For rule-bearing content — regulations, policies, contracts, clinical guidelines, legal memos, the rule-like observations a long-running agent accumulates as its own memory — it is catastrophic. Two rules that disagree on polarity are retrieved as near-duplicates. A downstream generator trained to synthesise produces fluent contradiction-synthesis.

The interesting question is: *what probability law do you use instead?*

## The Born rule as a probability law for NLP

The embedding space of a general-purpose model is already a Hilbert space. It has vectors, an inner product, a norm. What it's missing is the full operator algebra — or rather, the full operator algebra is available but nobody is using it. In particular, nobody is computing the quantity

$$
P(\text{outcome}\,|\,\psi) \;=\; |\langle\text{outcome}\,|\,\psi\rangle|^2
$$

as the answer to "what is the probability of this classification / this conflict / this topic label". Instead, we compute similarities and apply softmax with an arbitrary temperature.

The Born rule gives you three things for free that softmax does not:

1. **Calibration is built in.** Squared amplitudes on an orthonormal basis sum to one. No temperature, no Platt scaling, no calibration network.
2. **Sign matters.** If you drop the squaring, you have a signed inner product that can be positive (reinforcement), negative (cancellation), or near-zero (orthogonality). This is the *interference signal*. It is the single quantity the regulatory-conflict task requires, and it is the quantity cosine similarity silently drops.
3. **Superposition is first-class.** A state $|\psi\rangle$ that is a linear combination of basis states is a genuine superposition, not an average. A regulatory clause that asserts an obligation *and* an exception *and* a sanction is handled by the same machinery that handles a quantum superposition of polarisation states.

The QGI stack treats these three properties as first-class primitives. The embedding model (Q-Prime) is trained so that polarity, quantifier scope, and obligation strength are separable directions; the inference layer (Hilbert Space Compacting) computes named projections onto each; the Born rule is applied directly.

## The empirical claim

On QGI's internal regulatory-conflict benchmark — a corpus built from real regulatory text where the task is to identify pairs of rules in material conflict — classical cosine similarity across five widely used embedding models scores F1 = 0.000. The QAG interference signal driven by Q-Prime scores F1 = 1.000.

Three clarifications worth stating directly because they matter to researchers:

First, the **interference effect** is a property of the language of regulation itself, not of any one model. We've replicated it across embedding dimensionalities from 384 to 3,072, across four model families, and in out-of-domain corpora (medical safety, educational curricula, engineering safety, research ethics). Q-Prime doesn't create the effect; it makes it production-grade.

Second, conflict is only one observable. The Born-rule classifier — argmax over squared amplitudes on class centroids — is a zero-shot categorisation primitive that is calibrated by construction. We're releasing it as a standalone method note [QGI-TR-2026-04]. The short version: it is a cleaner generalisation of prototype networks and nearest-centroid classifiers, and it competes with supervised heads on tasks where you can write class prototypes.

Third, **full methodology is held for a forthcoming companion evaluation paper**. The headline numbers are real; the detailed cross-backbone, cross-extractor, out-of-domain numbers are under evaluation agreement. We don't publish them yet.

## Why this matters for agents, not just compliance

The same machinery addresses an open problem in agent memory that no current system solves: the consistency of long-term memory. Existing agent-memory systems (MemGPT, Letta, Zep, mem0, LangGraph memory) manage capacity — tiered storage, summarisation, importance scoring, temporal decay. None of them model consistency. A vector-indexed memory retrieves similar items; it has no primitive for detecting that two items contradict each other. The signed interference signal is exactly that primitive.

Concretely: when a new memory item is admitted, compute its interference with the top-$k$ existing items. A threshold on the negative half of the interval defines a contradiction event. What the agent does with the event — supersede, keep both with provenance, ask the user — is a policy choice. The *detection* is what was missing.

The companion preprint [QGI-TR-2026-03] develops this.

## The full corpus

This week we are releasing six preprints plus an executive whitepaper:

- **A** (QGI-TR-2026-01) — The canonical engine paper. Architecture, trust, full related work, limitations.
- **B** (QGI-TR-2026-02) — Why general-purpose embeddings fail on rule-bearing content. The F1 = 0 argument in full.
- **C** (QGI-TR-2026-03) — Conflict-aware agent memory. Integration sketches for Letta, LangGraph, mem0, and MCP.
- **D** (QGI-TR-2026-04) — The Born-rule classifier as a zero-shot method note.
- **E** (QGI-TR-2026-05) — Quantum HyperGraph (QHG) as a first-class data model.
- **F** (QGI-TR-2026-06) — A landscape review: RAG variants, graph-grounded, agent memory, long context, embedding models, QNLP.

The companion evaluation paper — paper G — is held until the evaluation agreement clears the numbers.

## Invitation

If you work in QNLP, compositional semantics, or quantum cognition, I'd genuinely like your criticism. We are a vendor; I am the CTO and Founder and I have a commercial interest. The criteria and methodology are the load-bearing part; I would rather have them improved by the academy than defended in solitude.

Research contact: `research@qgi.dev`. Evaluation access: `contact@qgi.dev`.
