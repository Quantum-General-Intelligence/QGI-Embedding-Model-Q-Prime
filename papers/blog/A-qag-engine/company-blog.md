---
title: "Introducing Quantum-Augmented Generation: A Reasoning-First Memory Infrastructure for Regulated AI and AI Agents"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
slug: "introducing-quantum-augmented-generation"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"   # replace with arXiv URL after submission
published: "2026-04-21"
category: "Engineering"
tags: ["QAG", "Q-Prime", "RAG", "regulated AI", "agent memory", "announcement"]
reading_time_min: 9
---

Today we are releasing the full technical preprint for **Quantum-Augmented Generation (QAG)** — the reasoning-first memory infrastructure we have been building at Quantum General Intelligence for the last two years. The paper is [QGI-TR-2026-01]. This post is the plain-English companion for engineering leaders, platform teams, and anyone who has been watching the RAG stack fail quietly on regulations, policies, contracts, and long-running agent memory.

## The retrieve-then-stuff primitive is broken for rule-bearing content

Retrieval-Augmented Generation was the best thing to happen to grounded AI in years. It made a whole class of applications tractable, from customer support to documentation search. And it works, right up until the content it is grounded in stops looking like prose and starts looking like *rules*.

A regulation is not a paragraph. It is a structured conditional with a trigger, conditions, an obligation, an exception, and typically half a dozen cross-references that modify its effect. Two rules can be almost lexically identical and say the opposite of each other — "the broker **must** disclose material conflicts" and "the broker **must not** disclose material conflicts". In every production embedding model we have tested, those two sentences are retrieved as near-duplicates. The generator downstream, told to synthesise, produces a fluent, confident, and wrong answer. The audit report that results is a liability.

The same pathology breaks AI agents. An agent accumulates memory — preferences, tool outputs, prior conclusions — and retrieves memory items by similarity. Similar is not the same as *consistent*. Yesterday's "I prefer Thai" and today's "I'm avoiding Thai" are retrieved together, and the agent picks one by attention bias.

This is not a tuning problem. It is a primitive problem. The fix is not a bigger model. The fix is a different primitive.

## What QAG does instead

QAG replaces `chunk → embed → retrieve → stuff` with a pipeline whose every stage is either deterministic, advisory, or auditable:

> `parse → normalise → hypergraph → Hilbert-space signals → hybrid search → grounded answer`

Four architectural choices define it.

**A deterministic parser (QNR2).** Ingestion is algorithmic, not LLM-based. The same input produces the same AST every time. There is no temperature, no sampling, no silent drift between runs.

**An immutable, versioned hypergraph (QHG).** Rules live as nodes; relationships — including *dependency* and *conflict* — live as hyperedges that can join any number of rules at once. The graph is immutable: updates produce a new version, and the state in force on any past date can always be recovered.

**A purpose-built embedding model (Q-Prime).** Q-Prime preserves polarity, scope, obligation strength, and cross-rule dependency as separable directions in a 1,536-dimensional Hilbert space. Q-Prime's representation finds the entangled superpositions that rule-bearing text actually contains, rather than averaging them out.

**Real quantum formalism on classical GPUs.** The word *quantum* in QAG refers to the mathematics — Hilbert spaces, the Born rule, superposition, signed interference — executed on ordinary GPU-accelerated compute. No quantum processing unit is required. The formalism is genuine; the substrate is classical because the state vectors are well within the range where classical simulation is exact.

On top of this sits **Hilbert Space Compacting (HSC)**, our intelligence layer. HSC projects the high-dimensional state of a rule set onto seven named subspaces and publishes seven named signals: Relevance, Conflict, Overlap, Redundancy, Coverage, Coherence, Topology. Each signal is a typed answer, not a metric — an Overlap answer is shaped differently from a Conflict answer.

## The F1 = 0 vs F1 = 1 gap

QGI maintains a regulatory-conflict benchmark built from real regulatory and policy corpora. The benchmark measures whether a system can correctly identify pairs of rules that materially conflict — the single task compliance teams cannot afford to get wrong.

Classical cosine similarity across five widely used embedding models from four organisations: **F1 = 0.000**.

QAG interference signal driven by Q-Prime: **F1 = 1.000**.

The gap is not incremental. It is categorical. The signal the task requires is simply absent from classical representations; the signal is present in Q-Prime by design, and the decision is binary on the sign.

Full methodology — corpora, embedding backbones, extractor-agnostic validation, out-of-domain generalisation, GPU throughput — is held for a forthcoming companion evaluation paper.

## Trust as a first-class property

The engineering properties that separate QAG from generative-first pipelines summarise to one equation we put on most of our slides:

> **Trust = Determinism + Traceability + Human authority.**

None of the three is optional. Remove determinism, reproducibility dies. Remove traceability, audit dies. Remove human authority, accountability dies. Every QAG answer ties to a specific rule in a specific section of a specific document, and every pipeline step is recorded and replayable.

Concretely, QAG publishes *signals*, not *decisions*. Whether a loan should be originated, a trade is compliant, a patient's prior authorisation is granted, a news item may be published — those decisions remain with the qualified human reviewer or a separately-certified automated pipeline. QAG is the AI component of such a pipeline, not a substitute for it.

## Beyond regulated industry

The same primitives that address compliance solve problems in AI-agent memory and context engineering that no current system addresses:

- **Memory consolidation** — the same interference signal that detects regulatory conflict detects contradiction between memory items. Agents using QAG-backed memory no longer believe yesterday's superseded preference.
- **Context curation** — the Born-rule relevance classifier gives a calibrated score for which memory items belong in the next LLM call.
- **Conflict-aware context packing** — given a fixed context budget, QAG picks the subset of items with the highest aggregate relevance that contains no internal contradiction. Empirically, contradictory context is a strong predictor of hallucination.
- **Multi-agent coordination** — signed interference separates "agree", "unrelated", and "contradict". Cosine similarity cannot.
- **Research and due-diligence agents** — contradictory sources are surfaced as a graph-level event, not buried under an averaged rank.

The QAG engine you license for compliance becomes the memory layer for your copilots. You deploy once.

## How to try it

- **Read the preprint.** [QGI-TR-2026-01] — *Quantum-Augmented Generation: A Reasoning-First Memory Infrastructure for AI Agents and Regulated Systems*.
- **Request an evaluation token.** `contact@qgi.dev`. 90-day evaluation free under the QGI Commercial Model License v1.0.
- **Use Q-Prime via OpenRouter** as part of the public beta.
- **Enterprise engagement** (SLA, dedicated endpoints, audit support): `contact@qgi.dev`.

Q-Prime is in progressive beta today; the full QAG engine hits general availability on **21 June 2026**.

## A note on what this is not

This paper is a vendor preprint. I am the CTO and Founder of QGI and have a commercial interest in Q-Prime and QAG. The numerical results above are from QGI's internal benchmark, released under evaluation agreement. Peer-reviewed numbers are forthcoming in a companion evaluation paper. Criticism, pointers to missed prior work, and counter-positioning are all welcome at `research@qgi.dev`.

If you build systems that need to tell "must" apart from "must not", this paper is for you.
