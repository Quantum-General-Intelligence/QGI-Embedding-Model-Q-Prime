---
title: "Show HN: Q-Prime, a 1536-D embedding model where polarity is a separable axis"
venue: "Hacker News (Show HN)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~260
---

Releasing Q-Prime today. 1,536-D embedding model trained with multi-objective supervision where polarity, scope, obligation strength, and cross-rule dependency are separable directions.

**The ten-line reproduction that motivated it.** Every widely deployed general-purpose embedding (OpenAI `text-embedding-3-large`, Cohere `embed-v3`, Voyage `voyage-3`, BGE-M3, E5-large, GTE-large) gives cosine similarity > 0.95 between a rule and its literal negation. These are treated as near-duplicates by downstream retrieval.

**Why it happens.** InfoNCE on open-web data rewards topical similarity. Negation is a rare event relative to topical variation. The gradient never learns to separate "must" from "must not".

**Why scaling won't fix it.** The *ratio* of topical to polarity displacement doesn't change with model size. Tested empirically.

**What Q-Prime does.** Multi-objective loss: polarity-flip contrastive term, scope projection supervision, ordinal regression on deontic strength, auxiliary head for pairwise dependency. Topical similarity is a by-product, not the target.

**Result on a regulatory-conflict benchmark.** Classical cosine across six open-domain embeddings: F1 = 0.000. Q-Prime + Born-rule classifier: F1 = 1.000.

**Born rule.** Because the space is a proper Hilbert space, probability comes from $|\langle c | \psi \rangle|^2$. Calibrated, training-free classification.

**Access.** API via OpenRouter (public beta). Enterprise: `contact@qgi.dev`.

**Preprint:** *Embedding Rule-Bearing Text: The Case for a Purpose-Built Model* — [arXiv URL].

Interested in critique, especially from QNLP/compositional-semantics researchers and from people who've mined hard-negative negation pairs in retrieval fine-tuning. I'm `sam@qgi.dev`.
