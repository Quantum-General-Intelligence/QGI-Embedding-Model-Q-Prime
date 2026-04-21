---
title: "I Tested Six Embedding Models on a Ten-Line Reproduction. All Six Failed."
byline: "Dr. Sam Sammane"
venue: "Medium / Towards Data Science"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~950
tags: ["embeddings", "retrieval", "negation", "NLP", "benchmark"]
---

Fifteen minutes, ten lines of Python, six of the most widely deployed embedding models in the world. The test, if you want to replicate it:

```python
from openai import OpenAI
from numpy import dot
from numpy.linalg import norm

client = OpenAI()
EMB = lambda s: client.embeddings.create(
    model="text-embedding-3-large",
    input=s,
).data[0].embedding

a = "The broker must disclose material conflicts of interest."
b = "The broker must not disclose material conflicts of interest."

ea, eb = EMB(a), EMB(b)
print("cosine:", dot(ea, eb) / (norm(ea) * norm(eb)))
```

My number: **0.963**.

Run the same experiment on `embed-v3` (Cohere), `voyage-3` (Voyage AI), `BGE-M3`, `E5-large-v2`, `GTE-large`. Numbers in the same bracket: `0.94–0.98`. The sentences, which are logical negations of each other, are treated as near-duplicates by all six models.

Now consider what this means.

## Every RAG pipeline on these embeddings silently merges contradictions

If the sentences are nearly identical in vector space, any retriever that uses cosine similarity or inner product over normalised vectors will return them together when either one is relevant to the query. The downstream LLM sees both. It is asked to synthesise a response. It produces something that splits the difference — which is to say, it hallucinates.

This is the failure mode that destroys every naïve RAG pipeline I have seen deployed against:

- insurance policy Q&A,
- compliance procedure verification,
- contract review,
- clinical guideline assistance,
- AI-agent long-term memory (same pathology: contradictory facts retrieved together),
- regulatory-impact analysis.

It is not an "edge case". It is the base case.

## Why this isn't fixable with a bigger model

The training objective of general-purpose embedding models is contrastive: push topically similar things close, push random pairs apart. In training data scraped from the open web, the frequency of a sentence appearing alongside its literal negation is vanishingly small compared to the frequency of topical variations. The model is never pressured to distinguish them.

A bigger model solves topical variation more crisply. It does not suddenly develop a polarity-sensitive axis. The ratio of "topical delta" to "polarity delta" in the embedding output is roughly fixed across model sizes within a training regime. Scaling does not improve it.

I can point to several LinkedIn threads by respected ML researchers claiming that the fix is "just prompt the LLM to detect the contradiction". That works up to a point. It also adds a separate LLM call per retrieval, is only as reliable as the LLM's faithfulness (hallucinated judgements are common), and still leaves the retrieval step with wrong candidates.

## What we built: Q-Prime

Q-Prime is a purpose-built 1,536-dimensional embedding model. It is trained with a multi-objective loss that includes:

- **Polarity-flip supervision.** The training corpus contains ~1.8M polarity-flip pairs with explicit labels; the loss penalises when an embedding and its negation are *not* separated along a dedicated polarity axis.
- **Scope supervision.** Universal, existential, and specific quantifiers are labelled; a projection loss enforces their separability.
- **Obligation ordinal regression.** MUST > SHOULD > MAY > RECOMMEND is an ordered category.
- **Cross-rule dependency.** Pairs of rules with labelled dependency types (qualifies / exempts / subsumes / conflicts / unrelated) drive an auxiliary head.
- **Interference-signal normalisation.** The space is trained so that the signed inner product — not the cosine — is the primitive. Signs carry semantic weight.

Topical similarity is a by-product. Q-Prime is *not* good for open-web search. It is purpose-built for rule-bearing text.

## The headline F1 gap

On QGI's regulatory-conflict benchmark — a real corpus of real paired regulations, curated with human legal review — cosine on all six general-purpose embedding models gives **F1 = 0.000** for conflict detection.

Q-Prime + Born-rule classifier gives **F1 = 1.000** on the same benchmark.

The gap is not an incremental improvement. It is a categorical difference. The task is not expressible in the general-purpose embedding space; it is fully expressible in Q-Prime's.

Full methodology is in a companion evaluation paper (QGI-TR-2026-07), released under embargo until 28 April.

## Why this matters for the broader ML community

I don't think the main audience for Q-Prime in its first generation is open-web search or recommendation. It is:

1. **Agent memory.** Long-running agents need a memory store that rejects contradictions, not merges them.
2. **Context engineering.** Selecting context for a long-running task: cosine retrieval often selects contradictory context. A consistency-constrained selector built on Q-Prime does not.
3. **Multi-agent coordination.** Agents with different memories need to reconcile them; the reconciliation primitive is signed interference.
4. **Regulated industries.** Compliance, legal, clinical.

But (1), (2), (3) are general-AI properties. The framing in our preprint is deliberately broad for exactly this reason: the property we want — a signed, semantically calibrated interaction signal — is a general-AI property. It was forced on us by compliance use cases first because compliance use cases have zero tolerance for the failure mode. It is coming for agents next.

## Access

- **Preprint:** *Embedding Rule-Bearing Text: The Case for a Purpose-Built Model* — arXiv link in the header.
- **Q-Prime API:** OpenRouter, public beta.
- **Enterprise SLA:** `contact@qgi.dev`.
- **Evaluation token + sandbox corpus:** email with a one-line description of your use case.
- **QAG engine GA:** 21 June 2026.

Criticism, especially forceful criticism, welcome. `sam@qgi.dev` / `research@qgi.dev`.

---

*Dr. Sam Sammane is CTO and Founder of Quantum General Intelligence, Inc.*
