---
title: "Zero-Shot Classification That Is Actually Calibrated: The Born-Rule Classifier"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "QGI Engineering Blog"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
slug: "born-rule-classifier-zero-shot-calibrated"
published: "2026-04-21"
length_words: ~1400
tags: ["classification", "zero-shot", "calibration", "Born rule", "embeddings"]
---

If you've deployed zero-shot classification with embeddings, you've run this code:

```python
embeds = model.encode([doc] + class_prompts)
scores = cosine(embeds[0], embeds[1:])
predicted = class_prompts[argmax(scores)]
confidence = softmax(scores / temperature)[argmax]
```

You pick a temperature, squint at the confidence numbers, and hope they mean something. They don't. The softmax output of a cosine-based zero-shot classifier is *not a probability*. It has no claim to calibration. It is a convenient summary of a ranking.

In the preprint we released today — **[QGI-TR-2026-04]**, *The Born-Rule Classifier: Calibrated Zero-Shot Categorization from Hilbert-Space Embeddings* — we describe a classifier that produces *actually calibrated* probabilities, zero-shot, with no training, in the same computational budget.

This post explains what the Born-rule classifier is, why it's calibrated where cosine-softmax is not, and when to use it.

## 1. The problem with cosine-softmax zero-shot

The modern zero-shot classification recipe — associated with CLIP, SentenceTransformers, and every embedding vendor's examples — is:

1. Embed the input document $d$ → $\mathbf{u}_d$.
2. Embed each class label (or a prompted description of each class) → $\mathbf{v}_1, \ldots, \mathbf{v}_C$.
3. Compute cosine similarities $s_c = \cos(\mathbf{u}_d, \mathbf{v}_c)$.
4. Convert to a "probability" via softmax: $p_c = \mathrm{softmax}(s_c / \tau)$.

This recipe has one empirical pathology and one theoretical pathology:

- **Empirical.** The output $p_c$ is almost always overconfident. A typical temperature ($\tau = 0.1$) gives $p_{\mathrm{top}} > 0.99$ on classification boundaries where the model is actually guessing. Lowering $\tau$ makes the output *more* uniform but doesn't restore calibration.

- **Theoretical.** Cosine is a measure of angular similarity; there is no principled justification for exponentiating it and calling the result a probability. The procedure is motivated by an analogy with logistic regression over learned logits, but cosine is not a logit.

## 2. The Born rule

Quantum mechanics has a specific, principled rule for converting an inner product in a Hilbert space into a probability. It's called the Born rule:

$$
P(\text{outcome } | \psi) = |\langle\text{outcome}\,|\,\psi\rangle|^2
$$

Here $|\psi\rangle$ is the state vector of the system, and $\langle\text{outcome}|$ is a projection defining the measurement outcome. The squared magnitude of the inner product — $|\langle\text{outcome}|\psi\rangle|^2$ — is the probability that outcome occurs.

An embedding space is a Hilbert space. Embeddings are state vectors. We can apply the Born rule without modification:

$$
P(c | \mathbf{u}_d) = \frac{|\langle \mathbf{v}_c | \mathbf{u}_d \rangle|^2}{\sum_{c'} |\langle \mathbf{v}_{c'} | \mathbf{u}_d \rangle|^2}
$$

This is the **Born-rule classifier**. It is a zero-shot classifier. It requires no training. It requires no temperature. It produces a proper probability distribution.

## 3. Why the Born-rule classifier is calibrated

Two answers, at different levels.

**The short answer.** Squaring the inner product converts a signed quantity (dot product) into a non-negative quantity (probability amplitude magnitude squared). The normalisation over class outcomes is exactly the right one: the total probability over a complete measurement basis sums to 1.

**The longer answer.** Cosine-softmax is a re-calibration of a sum of cosines using a free parameter $\tau$. The re-calibration is *data-independent* and *post-hoc*: we picked $\tau$ ourselves. The Born rule is a *principled* probability law derived from the geometry of the Hilbert space — no free parameter, no post-hoc calibration.

Empirically, we find that the Born-rule classifier produces distributions where the top probability genuinely tracks accuracy: if the classifier says "0.8 confidence", it's right about 80% of the time on held-out data. Cosine-softmax gives you "0.8 confidence" when it's right ~50% of the time, because the temperature was chosen without reference to calibration.

## 4. Does this work on arbitrary embeddings?

Partially. The Born-rule classifier requires the embedding space to be a Hilbert space with meaningful inner products. Every L2-normalised embedding space satisfies this geometrically. But **calibration depends on the embedding's training objective**.

- **General-purpose embeddings** (OpenAI, Cohere, BGE, E5) are trained with contrastive objectives that optimise for topical similarity. The Born-rule classifier on these still gives valid probabilities — they sum to 1, they are in $[0, 1]$ — but the calibration quality depends on how well the topical manifold approximates the classification manifold.

- **Purpose-built embeddings** (Q-Prime, but others would work too) can have *task-aligned axes*. If the embedding is trained so that classes are orthogonal directions, the Born-rule classifier is exact — and you get an actually-calibrated zero-shot classifier.

In our experiments (see the preprint), the Born-rule classifier:

- Improves calibration (Expected Calibration Error drops ~3-4×) over cosine-softmax on general-purpose embeddings.
- Gives near-perfect calibration on Q-Prime embeddings for the classification axes Q-Prime is trained on.

## 5. The centroid question

For each class, we need a class vector $\mathbf{v}_c$. Options:

1. **Prompt the class.** Embed a textual description of the class. This is the CLIP-style approach. Works, but sensitive to prompt wording.

2. **Centroid.** If we have a few labelled examples, embed each and take the (normalised) centroid. This is the few-shot variant.

3. **Prompt ensemble.** Embed multiple descriptions of the class, take the centroid of the embeddings. Reduces prompt-wording sensitivity.

4. **Trained class vectors.** If we have a lot of labelled data, train $\mathbf{v}_c$ directly against the Born-rule classifier objective. This is the supervised variant.

For zero-shot, (1) and (3) are the usual choices. For few-shot, (2) or (4). The Born rule itself is orthogonal to this choice; it applies to the inner product regardless of how $\mathbf{v}_c$ was constructed.

## 6. Concrete recipes

### Basic zero-shot

```python
import numpy as np

def born_rule_classify(doc_embedding, class_embeddings):
    doc = doc_embedding / np.linalg.norm(doc_embedding)
    classes = class_embeddings / np.linalg.norm(
        class_embeddings, axis=1, keepdims=True
    )
    overlaps = classes @ doc
    probabilities = overlaps ** 2
    return probabilities / probabilities.sum()
```

That's the whole classifier. Three lines in vectorised NumPy.

### Few-shot with centroid

```python
def build_class_centroid(examples, embed):
    vs = np.stack([embed(e) for e in examples])
    centroid = vs.mean(axis=0)
    return centroid / np.linalg.norm(centroid)
```

### Calibration evaluation

```python
def expected_calibration_error(probs, labels, n_bins=10):
    confidences = probs.max(axis=1)
    predictions = probs.argmax(axis=1)
    accuracies = (predictions == labels).astype(float)
    # Standard ECE computation
    ...
```

In our experiments, ECE drops from ~0.15 (cosine-softmax) to ~0.04 (Born rule) on a held-out general-domain classification task with OpenAI's `text-embedding-3-large`.

## 7. Two production applications

### Policy classification

A QAG pipeline needs to classify each extracted rule into one of several dozen policy categories (disclosure / suitability / KYC / AML / …). Cosine-softmax gets the top-1 right ~82% of the time but is badly miscalibrated — "0.9 confidence" means ~72% accuracy. The Born-rule classifier gets the top-1 right ~81% of the time — essentially the same — but "0.9 confidence" means ~88% accuracy. For downstream deferred-review gating, the second is dramatically more useful.

### Agent memory admission

For agent memory (see Paper C), we classify each incoming memory item into {fact, preference, hypothesis, reminder}. The Born-rule classifier's calibration means the agent can set a sensible threshold ("admit as fact only if P > 0.85") and have the threshold mean what it says. Cosine-softmax thresholds are unstable across corpora; the Born-rule thresholds are stable.

## 8. Limitations

- **Still a linear classifier in the embedding space.** If your classes are non-linearly separable in the embedding, Born-rule classifier won't save you. (You'd be in trouble with cosine-softmax too.)

- **Depends on the embedding.** Calibration is conditioned on the embedding's geometry. A bad embedding gives a bad Born-rule classifier, just not quite as bad as a bad cosine-softmax classifier.

- **Complete basis assumption.** The normalisation $\sum_c |\langle v_c | u \rangle|^2$ is only a probability distribution over a *complete* measurement basis. If your class vectors $\{v_c\}$ don't span the relevant subspace, the "not in any class" outcome is hidden. We address this in the preprint with an open-set extension.

- **No miracle on polarity.** If the embedding conflates polarity (i.e., a general-purpose embedding), the Born-rule classifier still won't distinguish "must X" from "must not X". Calibration is not the same as discrimination. You still need the right embedding. For rule-bearing text, that's Q-Prime.

## 9. Access

- **Preprint:** *The Born-Rule Classifier: Calibrated Zero-Shot Categorization from Hilbert-Space Embeddings* — arXiv link in the header.
- **Reference implementation:** included in the preprint's supplementary code.
- **Q-Prime API** for the embedding primitive: OpenRouter (public beta).
- **Evaluation collaborators:** `research@qgi.dev`.

The preprint is a method note; it is short (~10 pages) and deliberately self-contained. If you run zero-shot classification in production, I think you'll get something out of swapping in the Born-rule classifier for cosine-softmax regardless of which embedding you use.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
