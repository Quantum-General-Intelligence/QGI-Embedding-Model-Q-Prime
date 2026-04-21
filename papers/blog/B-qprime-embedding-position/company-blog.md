---
title: "Why Your Embedding Model Can't Tell 'Must' From 'Must Not' — And Why It Matters"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "QGI Engineering Blog"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
slug: "qprime-embedding-position-why-general-purpose-fails"
published: "2026-04-21"
length_words: ~1600
tags: ["embeddings", "Q-Prime", "retrieval", "regulated AI", "agent memory", "ML"]
---

Every general-purpose embedding model in production today has a specific, reproducible failure mode: it cannot reliably distinguish a rule from its logical negation.

This is not a marketing argument. It is a property of the training objective. In this post I want to explain precisely what the failure is, why it cannot be fixed by scaling, and why QGI built a separate, purpose-built embedding model — Q-Prime — to close the gap for rule-bearing content.

This is a non-mathematical tour of our companion preprint, **"Embedding Rule-Bearing Text: The Case for a Purpose-Built Model"** ([QGI-TR-2026-02]).

## 1. The failure you can reproduce in ten lines

Take any widely deployed embedding model — OpenAI `text-embedding-3-large`, Cohere `embed-v3`, Voyage `voyage-3`, BGE-M3, E5-large, GTE-large. Run it on this pair:

- *A:* "The broker must disclose material conflicts of interest to the client before executing the trade."
- *B:* "The broker must **not** disclose material conflicts of interest to the client before executing the trade."

In all six models we tested, the cosine similarity is above 0.95. In several, it exceeds 0.98. The models treat these as near-duplicates.

Now try:

- *A:* "The broker must disclose material conflicts of interest."
- *C:* "The weather in Lisbon is mild this week."

The cosine similarity drops to roughly 0.1–0.2, as you would expect.

The ratio of the *semantic delta* you get from an unrelated topic to the *semantic delta* you get from logical negation is on the order of 10:1 or worse in open-domain embeddings. That means any retrieval system driven by cosine similarity will reliably retrieve negations as close matches.

In a FAQ application, this is harmless. In a contracts application, a compliance application, a clinical-guidelines application, or an AI agent's memory store, it is the entire failure mode of the system.

## 2. Six properties a rule-bearing embedding must preserve

The preprint enumerates six concrete properties that any embedding that is fit for rule-bearing content must preserve. General-purpose embeddings preserve, at best, one or two.

1. **Polarity.** The sign of a rule — whether it asserts or denies — must live in a separable direction in the embedding space. If the polarity direction is entangled with topic or style, negations will be embedded as near-duplicates.

2. **Quantifier scope.** "All customers" is not "some customers"; "at least one" is not "exactly one". Scope changes the set of affected instances. Embeddings must distinguish them.

3. **Structured conditions.** "If the transaction exceeds $10,000" is part of the rule. A model that ignores the `if` — or treats it as topical noise — cannot reconstruct when the rule applies.

4. **Obligation strength.** "Must", "should", "may", "is recommended to" — the deontic category of the rule determines the downstream enforcement action. These must be graded, not collapsed.

5. **Cross-rule correlation.** Rule A qualifies Rule B; Rule C exempts Rule D under conditions. The embedding of A ought to carry a signal about how it relates to B and D. In a general-purpose embedding, it does not.

6. **Signed interference.** When two rules *agree*, their interaction should be positive; when they *disagree*, their interaction should be negative. Cosine similarity cannot produce a negative value in any semantically meaningful way.

None of these six is a preference. Each one is a hard requirement in at least one production application we ship.

## 3. Why scaling doesn't help

A reasonable question: surely a larger embedding model trained on more data will eventually get this right?

No. The reason is the training objective.

General-purpose embedding models are trained with contrastive objectives — InfoNCE, triplet loss, or similar. The objective is "move semantically similar pairs close, move unrelated pairs far". The "similar" signal is overwhelmingly topical. "Discloses conflicts of interest" and "does not disclose conflicts of interest" share every topical word; the training objective will push them together unless negation is *explicitly* over-weighted.

Going bigger makes the space bigger. The absolute displacement from negation does grow slightly, but the displacement from topic change grows faster. The *ratio* does not improve.

Fine-tuning helps. A compliance-tuned embedding model can close the polarity gap partially on the domains it saw. It does not close it on new domains, and it does not generalise to the other five properties.

What's needed is a **different training objective**. That is what Q-Prime is.

## 4. What Q-Prime does differently

Q-Prime is a 1,536-dimensional embedding model trained with a multi-objective loss that rewards separability along polarity, scope, obligation, and cross-rule dependency. It is deliberately not trained as an open-web similarity model. Topical similarity is a by-product, not a training target.

Five concrete architectural choices:

1. **Polarity is a supervised axis.** The training corpus contains explicit polarity-flip pairs, and the loss rewards orthogonalising them along a dedicated subspace.

2. **Scope is a sub-representation.** Quantifier scope (universal / existential / specific / general) is a separable sub-representation you can project out.

3. **Obligation is graded.** "Must / should / may / recommend" are trained as an ordinal category, not as synonyms.

4. **Cross-rule dependency is contrastive.** Pairs of rules that qualify, exempt, or subsume each other are trained with explicit dependency labels.

5. **The space is a proper Hilbert space.** The inner product — not the cosine — is the primitive. Signs matter. The Born rule applies without modification.

## 5. The headline empirical result

On QGI's regulatory-conflict benchmark:

- Six widely deployed open-domain embedding models (four organisations): **F1 = 0.000** on conflict detection.
- Q-Prime + QAG signed interference: **F1 = 1.000**.

This is not a percentage-point improvement. It is a categorical difference: the task is not expressible in the general-purpose embedding space, and is fully expressible in Q-Prime's.

The full methodology, ablations, and out-of-domain validation are held for a forthcoming companion evaluation paper (QGI-TR-2026-07). Enterprise reviewers receive full methodology under evaluation agreement.

## 6. "Why not just fine-tune an existing embedding?"

The three most common fine-tuning strategies we benchmarked against:

1. **LoRA on a frozen encoder.** Closes the polarity gap partially on seen domains. Does not generalise; breaks on the five other properties.
2. **Adapter layers for polarity.** Same pattern. Works until it doesn't.
3. **Task-specific re-ranking after retrieval.** Requires the retrieval stage to have surfaced the right candidates, which is exactly what the broken embedding space prevents.

Each of these is a patch. Q-Prime is a different training regime.

## 7. Why this matters outside regulated industry

The same six properties are what agent memory needs.

An AI agent that "remembers" your preferences, your goals, your recent actions cannot tolerate its memory store returning contradictory items as if they were reinforcement. "Avoiding Thai" and "prefers Thai" are both retrieved by a cosine-based memory store. The agent then picks one by attention bias.

In a memory store built on Q-Prime, the two memories produce a negative interference signal. The agent has a first-class event — *consistency breach* — to act on. This is an implementation of the reasoning-first architecture described in [QGI-TR-2026-03].

Context engineering has the same shape. A context window is a budget; you decide what to include. Cosine-similarity context selection frequently includes contradictory passages because each is "close to the query". A Born-rule classifier over Q-Prime can pick a *consistent* subset that maximises coverage of the user's intent under a consistency constraint.

## 8. What we're releasing

- **Q-Prime API.** Available via OpenRouter (public beta) and as a managed endpoint for enterprise customers with an SLA. No weights released in 2026.
- **QAG Engine.** GA 21 June 2026. Bundles Q-Prime with the deterministic parser, hypergraph, and signal layer.
- **Companion preprints.** Six preprints on the theory, the engine, the data model, the classifier, the landscape, and the agent-memory story. Full list in `papers/README.md`.

## 9. How to evaluate it

Three things you can do this week.

1. **Run the ten-line reproduction.** Pick your current embedding. Compute cosine on any rule and its negation. Email me the result. `sam@qgi.dev`.
2. **Request an evaluation token.** Email `contact@qgi.dev` with a one-page description of your use case. You get an API key, a sandbox corpus of anonymised rules, and a playbook.
3. **Read the preprints.** The preprint pack is on arXiv: [QGI-TR-2026-01] through [QGI-TR-2026-06]. The evaluation pack is QGI-TR-2026-07 (under embargo until 28 April).

---

Criticism is welcome, including the forceful kind. The easiest way to end a bad ML idea is to publish it and let the community break it.

*The author is Dr. Sam Sammane, CTO and Founder of Quantum General Intelligence, Inc. (sam@qgi.dev). This post summarises the companion preprint available at [arXiv URL].*
