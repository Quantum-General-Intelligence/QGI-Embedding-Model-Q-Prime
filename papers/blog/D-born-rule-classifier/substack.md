---
title: "The Born-Rule Classifier: A Short Method Note on Calibrated Zero-Shot Categorization"
byline: "Dr. Sam Sammane"
venue: "Substack (QGI Research Notes)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1100
tags: ["method", "zero-shot classification", "calibration", "Born rule"]
---

This is a short-form summary of preprint [QGI-TR-2026-04], *The Born-Rule Classifier: Calibrated Zero-Shot Categorization from Hilbert-Space Embeddings*. The preprint is a method note (~10 pages); this post hits the same beats more quickly.

## Setup

Given

- $\mathbf{u}_d \in \mathcal{H}$: the L2-normalised embedding of a document $d$.
- $\{\mathbf{v}_c\}_{c \in C}$: L2-normalised class prototypes, one per class, in the same space.
- A target: a probability distribution $P(c | d)$ over classes.

The standard zero-shot recipe computes

$$
P_{\cos}(c | d) = \frac{\exp\left( \cos(\mathbf{u}_d, \mathbf{v}_c) / \tau \right)}{\sum_{c'} \exp\left( \cos(\mathbf{u}_d, \mathbf{v}_{c'}) / \tau \right)}
$$

with $\tau$ a free temperature.

## The method

The Born-rule classifier computes

$$
P_{\mathrm{Born}}(c | d) = \frac{|\langle \mathbf{v}_c | \mathbf{u}_d \rangle|^2}{\sum_{c'} |\langle \mathbf{v}_{c'} | \mathbf{u}_d \rangle|^2}
$$

where $\langle \cdot | \cdot \rangle$ is the Hilbert-space inner product. For real-valued embeddings, this reduces to $(\mathbf{v}_c^\top \mathbf{u}_d)^2$ up to normalisation.

No temperature. No free parameter.

## Observations

### 1. Algebraic properties

$P_{\mathrm{Born}}$ is a proper probability distribution:

- Each component is in $[0, 1]$.
- They sum to 1.

It is symmetric under unitary transformations of $\mathcal{H}$ (rotations of the basis):

$$
P_{\mathrm{Born}}(c | d; U \mathbf{v}_c, U \mathbf{u}_d) = P_{\mathrm{Born}}(c | d; \mathbf{v}_c, \mathbf{u}_d)
$$

so it's a geometric object of the embedding space, not of the representation choice.

Cosine-softmax is *not* unitarily invariant. It is rotation-invariant, which is weaker.

### 2. Completeness

The Born-rule probabilities sum to 1 *over the given set of class prototypes*. This is different from the physics setting, where the measurement basis is complete.

If the $\{\mathbf{v}_c\}$ do not span the relevant subspace — if the document's state has a component orthogonal to all classes — the Born-rule probabilities only cover the "in-basis" probability mass. The preprint develops an open-set extension that reserves probability for "none of the above" using the residual norm $\| \mathbf{u}_d - \sum_c \mathbf{v}_c \langle \mathbf{v}_c | \mathbf{u}_d \rangle \|^2$.

### 3. Calibration

On general-purpose embeddings (text-embedding-3-large, bge-m3, e5-large), we observe:

- $P_{\cos}$ with $\tau = 0.1$: ECE ≈ 0.15 on a multi-class classification benchmark.
- $P_{\mathrm{Born}}$: ECE ≈ 0.04 on the same benchmark.

The Born-rule classifier does not need temperature tuning to be calibrated; cosine-softmax cannot be made well-calibrated for all datasets with any single temperature.

On Q-Prime (purpose-built), the Born-rule classifier gives ECE < 0.01 on classification axes that Q-Prime is trained on. This is not a general property of Q-Prime; it is specific to the classification problems for which its prototypes are orthogonal.

### 4. Top-1 accuracy

Top-1 accuracy is preserved: $\arg\max_c P_{\mathrm{Born}}(c) = \arg\max_c P_{\cos}(c)$ whenever all inner products have the same sign.

Because cosine maps to the positive half of the unit sphere for L2-normalised embeddings, the signs are generally the same, and the argmax agrees. Exceptions occur when the embedding has been trained to produce signed inner products (Q-Prime does), in which case the Born-rule classifier might choose a different argmax — and we've seen these cases be the right call on polarity-laden inputs.

### 5. Few-shot extensions

If you have $k$ labelled examples per class, build the centroid:

$$
\mathbf{v}_c = \frac{1}{k} \sum_i \mathbf{u}_{d_i^{(c)}} \quad \mathrm{normalized}
$$

The Born-rule classifier over these centroids gives a calibrated few-shot classifier. No training step. No temperature.

Compare with standard kNN-style few-shot: similar argmax, better calibration, no free parameters.

### 6. Interaction with purpose-built embeddings

The Born-rule classifier's calibration depends on the geometry. Purpose-built embeddings with orthogonal class axes (like Q-Prime's polarity / scope / obligation axes) give near-perfect calibration.

This is an asymmetric advantage that favours the purpose-built embedding direction. A well-trained Q-Prime + Born-rule classifier stack gives probabilities you can put in a production gate. A general-purpose embedding + Born-rule classifier gives a material improvement but still should not be trusted for high-stakes decisions.

## Related work

- **Prototypical networks** (Snell et al., 2017): few-shot classifier using centroids and a squared-Euclidean distance kernel. Similar structure; different probability rule.

- **CLIP / SigLIP**: contrastive vision-language models with cosine-softmax zero-shot classifier. Same critique applies; Born-rule classifier gives a drop-in improvement.

- **Temperature-scaled calibration**: post-hoc calibration by tuning $\tau$ on held-out data. Does not generalise; the Born rule is a *structural* calibration, not post-hoc.

- **Density-based calibration**: fitting a probability model over logit space. Heavier machinery; similar calibration gain; less interpretable.

- **Compositional / quantum NLP**: uses Born-rule-like structures for sentence meaning (Coecke, Clark). Different objective; shares mathematical machinery.

## Limitations

1. **Not a miracle on discrimination.** If your embedding conflates two classes, the Born rule still won't separate them. Q-Prime addresses polarity / scope / obligation specifically; other classes are your responsibility.

2. **Open-set requires care.** The Born-rule probability mass is only over the given class basis. For open-set problems, use the residual-norm extension described in the preprint.

3. **Quadratic in inner product.** The squared amplitude penalises near-zero inner products more than cosine-softmax. For diffuse distributions (many near-tie classes), the result is sharper; this is usually desirable but can be surprising.

4. **Numerical.** Squared inner products approach zero quickly; for very small amplitudes, use log-space computation.

## Closing

This is a deliberately compact preprint. The Born-rule classifier is a three-line replacement for cosine-softmax that gives you calibration for free, with minimal risk to top-1 accuracy.

If you run a zero-shot classification pipeline in production, I would be interested to hear about your calibration metrics before and after. `research@qgi.dev`.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
