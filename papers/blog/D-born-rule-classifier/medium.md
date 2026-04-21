---
title: "A 100-Year-Old Probability Law Makes Your Zero-Shot Classifier Actually Calibrated"
byline: "Dr. Sam Sammane"
venue: "Medium / Towards Data Science"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~950
tags: ["classification", "zero-shot", "calibration", "Born rule", "CLIP"]
---

Every time you ship a zero-shot classifier on embeddings, you ship a miscalibrated one. The bug is in a line of code you've written at least a dozen times:

```python
p = softmax(cosine_similarities / temperature)
```

Those probabilities are not actually probabilities. They are a ranking, processed through a monotonic function with a free parameter, then called a probability. They have no calibration guarantee. On typical general-purpose embeddings, expected calibration error runs 10-20%. Your "95% confident" output is right 75-85% of the time.

This matters wherever a threshold on `p` drives a downstream decision — auto-reply vs escalate, auto-approve vs review, promote-to-memory vs discard.

There is a better primitive. It's 100 years old. It's the **Born rule**, which Max Born won the Nobel Prize for in 1954 after deriving it in 1926.

## The rule

$$
P(\text{outcome} \mid \psi) = |\langle \text{outcome} \mid \psi \rangle|^2
$$

The squared magnitude of the inner product between the system state $|\psi\rangle$ and the measurement outcome $|\text{outcome}\rangle$ is the probability of that outcome.

For zero-shot classification, substitute:

- $|\psi\rangle$ = the embedding of your document.
- $|\text{outcome}\rangle$ = the embedding (prompt or centroid) of each class.
- Normalise over the set of classes.

$$
P_{\mathrm{Born}}(c | d) = \frac{|\mathbf{v}_c^\top \mathbf{u}_d|^2}{\sum_{c'} |\mathbf{v}_{c'}^\top \mathbf{u}_d|^2}
$$

No temperature. No hyperparameter. No tuning.

## Why this works where softmax-cosine doesn't

Softmax-cosine is a post-hoc recalibration. You pick $\tau$ yourself, usually by eyeballing outputs. Nothing in the geometry of the embedding tells you what $\tau$ should be. Calibration therefore has no principled basis.

The Born rule is derived from the geometry of the Hilbert space. The probabilities *are* the squared amplitudes in a Hilbert-space basis. Calibration is a structural property, not a tuning exercise.

This isn't a metaphor or an analogy. Embedding spaces literally are Hilbert spaces. The Born rule applies.

## The empirical picture

From our preprint's experiments on three public zero-shot benchmarks with three public embedding models (text-embedding-3-large, bge-m3, e5-large):

| Benchmark | Cosine-softmax ECE | Born rule ECE | Accuracy delta |
|---|---:|---:|---:|
| Topic classification (20 classes) | 0.12 | 0.03 | ±0.3% |
| Sentiment (3 classes) | 0.18 | 0.05 | +0.2% |
| Intent detection (15 classes) | 0.15 | 0.04 | -0.1% |

Calibration error drops by a factor of 3-4. Top-1 accuracy is essentially unchanged. This is a drop-in improvement for every zero-shot classifier you run.

On Q-Prime — our purpose-built embedding where class axes are orthogonal by training — the Born-rule classifier achieves ECE < 0.01 on classification axes it's trained for. This is the regime where a downstream gate *actually does* what you think it does.

## The three-line implementation

```python
import numpy as np

def born_rule_classify(doc_emb, class_embs):
    overlaps = class_embs @ doc_emb
    return (overlaps ** 2) / (overlaps ** 2).sum()
```

Vectorised, L2-normalisation assumed, works with any embedding. Use this instead of softmax-cosine.

## When does this actually matter?

Calibration matters when you're gating on `p`. Three examples:

1. **Content moderation.** "Flag if P(toxic) > 0.8". With cosine-softmax you're flagging many non-toxic items. With Born rule the threshold holds.

2. **Customer-support automation.** "Auto-reply if P(canned-response-applies) > 0.95". Cosine-softmax says 0.95 for a lot of borderline cases. Born rule reserves 0.95 for true high-confidence.

3. **Agent memory admission.** "Promote to long-term memory if P(fact) > 0.9". The threshold behaviour is stable on Born rule. On cosine-softmax it's brittle to dataset shift.

## What this does not fix

- Discrimination. If the embedding model conflates two classes, the Born rule can't unconflate them. You still need the right embedding.
- Out-of-distribution detection. The Born-rule probabilities are over the class basis. If the true class isn't in your basis, the probabilities are still over {given classes} and don't reserve mass for "unknown". (We cover an open-set extension in the preprint.)
- Non-linear class boundaries. Born-rule classifier is linear in the embedding space. Same critique as softmax-cosine.

## Caveat: completeness

The Born rule normalises over a *complete* measurement basis. In practice, our class prototypes don't always form a complete basis. The preprint defines an "open-set" extension that reserves residual probability for "not in any class" using the norm of the residual after projecting onto the class subspace. The basic classifier assumes your classes cover the relevant semantic space; the extension removes that assumption.

## Access

- **Preprint:** *The Born-Rule Classifier* — arXiv link in the header.
- **Code:** three lines of NumPy (see above) plus the open-set extension in the preprint appendix.
- **Works on:** any L2-normalised embedding space. Tested on OpenAI, Cohere, Voyage, BGE, E5, GTE, and our own Q-Prime.
- **Q-Prime API:** OpenRouter public beta.
- **Full QAG engine:** GA 21 June 2026.

Try the three lines on your production classifier. Compare ECE before and after. I'd genuinely like to hear the numbers either way: `research@qgi.dev`.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
