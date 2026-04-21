---
title: "Show HN: Three-line Born-rule classifier that gives 3-4× better calibration than softmax-cosine"
venue: "Hacker News (Show HN)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~240
---

Short method note from our research group. The standard zero-shot classifier recipe —

```
scores = cosine(doc, class_embeds)
p = softmax(scores / temperature)
```

— gives you numbers that are **not probabilities**. There is no principled calibration, and the temperature is a free parameter you guess.

The Born rule from quantum mechanics gives a principled replacement with no free parameter:

```python
amplitudes = class_embeds @ doc       # inner product
p = (amplitudes ** 2) / (amplitudes ** 2).sum()
```

Embedding spaces are literally Hilbert spaces. The Born rule applies directly: $P(c|\psi) = |\langle c|\psi\rangle|^2$.

Empirical results on three public zero-shot benchmarks and three public embedding models:

- Expected Calibration Error drops 3-4× (0.15 → 0.04).
- Top-1 accuracy is unchanged (within ±0.3%).
- No temperature tuning, no training.

Preprint: [QGI-TR-2026-04], *The Born-Rule Classifier: Calibrated Zero-Shot Categorization from Hilbert-Space Embeddings*. ~10 pages, self-contained.

On Q-Prime (our purpose-built embedding where class axes are orthogonal by training), ECE < 0.01 on classification axes it's trained for.

This is a drop-in replacement for every softmax-cosine zero-shot classifier you run. If you're gating downstream decisions on the probability ("flag if P(toxic) > 0.8", "auto-reply if P > 0.95"), the Born rule gives you a gate that actually means what it says.

Feedback welcome. `research@qgi.dev`.
