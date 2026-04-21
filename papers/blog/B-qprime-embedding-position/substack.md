---
title: "The Case for a Purpose-Built Embedding: Six Properties General-Purpose Models Do Not Preserve"
byline: "Dr. Sam Sammane"
venue: "Substack (QGI Research Notes)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1100
tags: ["embeddings", "contrastive learning", "retrieval", "rule-bearing text"]
---

This note accompanies preprint [QGI-TR-2026-02], *Embedding Rule-Bearing Text: The Case for a Purpose-Built Model*. It is a position paper with an empirical spine, aimed at researchers working on embedding models, retrieval, and their intersection with reasoning.

## The claim

General-purpose embedding models trained with contrastive objectives cannot be patched into a reliable substrate for rule-bearing content. The gap is not a matter of scale, data, or fine-tuning. It is structural: the training objective rewards a different thing than what the downstream task requires.

## The six properties

A rule-bearing embedding must preserve six properties. Four of these are linguistically well-studied (polarity, scope, structured conditions, obligation strength). The fifth (cross-rule dependency) is less studied. The sixth (signed interference) is a property of the representation space itself rather than the individual embeddings, and is central to what Q-Prime exists for.

1. **Polarity.** Formally: the assertion sign of a proposition is a binary feature that must be recoverable as a linear functional of the embedding. Contrastive objectives that use in-batch negatives rarely include polarity-flip pairs, so the direction is not supervised.

2. **Quantifier scope.** Formally: the scope operator (∀, ∃, "at least k", etc.) modifies the truth condition of the underlying proposition. Natural-language embeddings conflate surface phrases ("all / every / any") without a consistent semantic basis.

3. **Structured conditions.** Formally: a rule of the form *if C then A* is a guarded assertion. The guard C must be retrievable as a structured sub-representation, not reduced to topical features of the surface string.

4. **Obligation strength.** The deontic category (MUST / SHOULD / MAY / RECOMMEND) determines the downstream enforcement action. Collapsing these into near-synonyms is the single most damaging representational choice for compliance.

5. **Cross-rule correlation.** Regulations are not isolated statements. A rule's embedding should encode a signal about how it relates to nearby rules: qualifies, exempts, subsumes, contradicts. General-purpose embeddings don't carry this; they don't have a training signal for it.

6. **Signed interference.** This is the property of the space, not of the individual vector. The signed inner product $\langle a | b \rangle$ must be positive when rules agree, negative when they conflict. Cosine similarity cannot be negative in a semantically meaningful way.

## Why contrastive learning can't give you these for free

The standard InfoNCE-style objective writes:

$$
\mathcal{L} = -\log\frac{\exp(\langle a, a^+\rangle / \tau)}{\sum_{b}\exp(\langle a, b\rangle / \tau)}
$$

The positives are topically or lexically similar pairs. The negatives are random batch neighbours. The objective pushes positives together and negatives apart.

Two things are absent from this signal: (i) systematic polarity flips, and (ii) semantically adversarial pairs (A and ¬A). Absent both, the model learns to treat a polarity flip as a small perturbation of the underlying proposition.

This is an empirical claim, not speculation. It reproduces across six embedding families tested in our work.

## What a fit-for-purpose objective looks like

For polarity, the simplest correct supervision is a dedicated polarity-flip contrastive term:

$$
\mathcal{L}_{\text{pol}} = \| \mathbf{u} + \mathbf{u}_{\neg} \|^2 - \lambda \| \mathbf{u} - \mathbf{u}_{\neg} \|^2
$$

where $\mathbf{u}$ and $\mathbf{u}_{\neg}$ are embeddings of a proposition and its negation, and $\lambda$ controls how strongly the polarity direction dominates the topical direction. This is a schematic — the full objective carries scope and obligation terms in parallel.

For scope, we found that a projection-based supervision — requiring that a linear projection onto a designated scope axis recovers the quantifier — worked better than treating scope as a class label.

For obligation, the deontic category is trained as an ordinal regression, not as classification. MUST > SHOULD > MAY is an order; a categorical classifier loses the order.

For cross-rule dependency, we use an auxiliary pairwise head that predicts the dependency type (qualifies / exempts / subsumes / conflicts / unrelated) from the concatenation of the two embeddings. Gradient flows back into the encoder; the encoder learns a dependency-sensitive representation.

None of these are radical novelties. What is slightly unusual is that they are all composed simultaneously into a single multi-objective training loss, with the loss weighted so that topical similarity is a by-product, not the primary target.

## The Born-rule classifier

A consequence of building the embedding as a proper Hilbert space is that the Born rule applies directly:

$$
P(c | \psi) = |\langle c | \psi \rangle|^2
$$

where $|\psi\rangle$ is the embedding of a passage and $|c\rangle$ is the centroid of a class. Because $|c\rangle$ is normalised, the probabilities $\{P(c|\psi)\}_c$ sum to a well-defined quantity and can be re-normalised to a categorical distribution. This gives a calibrated zero-shot classifier without training.

Empirically: the Born-rule classifier over Q-Prime gives calibrated probabilities on a held-out rule corpus, whereas cosine-softmax over a general-purpose embedding gives distributions that are uniformly over-confident. Full details in preprint [QGI-TR-2026-04].

## Empirical anchor

The F1 gap on conflict detection:

| Model | Cosine F1 | Born-rule / Interference F1 |
|---|---:|---:|
| `text-embedding-3-large` | 0.000 | — |
| `embed-v3` | 0.000 | — |
| `voyage-3` | 0.000 | — |
| `BGE-M3` | 0.000 | — |
| `E5-large` | 0.000 | — |
| `Q-Prime` | 0.001 | **1.000** |

(The 0.001 with cosine on Q-Prime's vectors is the null-baseline noise floor; cosine is not the primitive the embedding is trained for.)

Full methodology and out-of-domain validation: forthcoming evaluation paper QGI-TR-2026-07. Released under NDA for enterprise reviewers, public release after embargo.

## What this does not claim

- It does not claim Q-Prime is better than general-purpose embeddings for open-web search. It is not trained for that and will underperform.
- It does not claim negation is the only failure mode. It claims negation is the failure mode you can reproduce in ten lines, and the other five are equally real and equally structural.
- It does not claim scaling has been exhausted. It claims scaling does not close the *ratio* between topical and polarity displacements in the objective that these models are trained with. A different objective is required; Q-Prime is one instance.

## Criticism welcome

I am especially interested in critiques from the compositional-semantics community (QDisCoCat, quantum natural language) and from the retrieval-QA community who have implemented hard-negative mining on negation pairs. The easiest way to retire an embedding model is to publish it.

Research email: `research@qgi.dev`. Preprint on arXiv.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
