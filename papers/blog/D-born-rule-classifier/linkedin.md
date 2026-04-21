---
title: "Stop Softmaxing Your Cosine Similarities. There's a Better Primitive, and It's 100 Years Old."
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "LinkedIn Pulse"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~650
---

Every ML team I talk to deploys zero-shot classification with the same recipe:

1. Embed the document and each class label.
2. Compute cosine similarities.
3. Softmax with a temperature.
4. Call the result a "confidence score".

That result is not a probability. It is a ranking masquerading as a probability. It has no calibration guarantees, and the temperature is chosen by guess-and-check.

When you then build a downstream gate — "auto-approve if confidence > 0.9, escalate otherwise" — the gate does not mean what you think it means. Calibration error on cosine-softmax confidence is typically 0.10 to 0.20. Your "95% confident" decisions are correct 75% of the time.

## The fix is a principled probability law, and it's 100 years old

In quantum mechanics, there is one and only one principled rule for converting inner products in a Hilbert space into probabilities. It's the **Born rule**:

$$
P(\text{class } c \mid \text{document } d) = |\langle \mathbf{v}_c | \mathbf{u}_d \rangle|^2
$$

— normalised over the complete set of classes.

Embeddings already live in a Hilbert space. The Born rule applies directly. No temperature parameter. No free hyperparameters. Calibrated by the geometry itself.

In our preprint released today — [QGI-TR-2026-04] — we show the Born-rule classifier:

- **Preserves top-1 accuracy** relative to cosine-softmax (within noise) on general-purpose embeddings.
- **Reduces expected calibration error by 3-4×** across three public benchmarks.
- **Gives actually-calibrated probabilities** when coupled with a purpose-built embedding like Q-Prime, so your downstream decision gates mean what you think they mean.

## Why this matters for AI procurement

If your enterprise runs a zero-shot classification gate somewhere in its AI pipeline — and almost every enterprise does — the calibration of that gate matters more than you think.

- **Compliance triage.** "Confidence > 0.9 auto-approve" means different things under different classifiers. With cosine-softmax, you're auto-approving many borderline cases. With Born rule, you're auto-approving only high-confidence cases.

- **Customer support deflection.** "Only respond automatically if 95% sure". Cosine-softmax says it's 95% sure ~60% more often than the Born rule does. That's 60% more risky auto-responses.

- **Agent memory admission.** "Only promote to long-term memory if 85% confident it's a fact". The threshold behaviour is stable with the Born rule, unstable with cosine-softmax.

Calibration is the difference between a gate that works as designed and a gate that lets edge cases through.

## This is low-effort to adopt

Replacing cosine-softmax with the Born-rule classifier in your pipeline is three lines of code:

```python
# Before
scores = cosine(doc, class_embeds)
p = softmax(scores / 0.1)

# After
amplitudes = class_embeds @ doc   # inner product, not cosine
p_unnormalised = amplitudes ** 2
p = p_unnormalised / p_unnormalised.sum()
```

Same embeddings. Same classes. Same compute. Better calibration.

## Why this is named after Born

Max Born, 1926. Nobel Prize in Physics. The rule specifies how to turn quantum amplitudes into probabilities for physical measurement outcomes. Embedding spaces are Hilbert spaces; embeddings are amplitudes. The mathematics transfers without modification.

This is not metaphor. It is not "quantum-inspired". It is the Born rule applied to a Hilbert space that happens to be an embedding model. The rule worked for physics; it works for ML.

## Access and next steps

- **Preprint:** arXiv link in the header.
- **Reference implementation:** 20 lines of NumPy, in the preprint's appendix.
- **Q-Prime API** (the purpose-built embedding where calibration is near-perfect): OpenRouter public beta.
- **Full QAG engine** GA on **21 June 2026**.

If you'd like a technical review of your current classification pipeline, I'm at `sam@qgi.dev`.

---

**Dr. Sam Sammane** is CTO and Founder of Quantum General Intelligence, Inc.
