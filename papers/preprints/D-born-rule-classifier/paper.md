---
title: |
  A Born-Rule Classifier
subtitle: |
  Zero-Shot Categorisation via Squared Amplitudes on Class Centroids
author:
  - name: Sam Sammane
    email: sam@qgi.dev
    affiliation: Chief Technology Officer and Founder, Quantum General Intelligence, Inc.
date: "21 April 2026"
version: "Preprint v1.0"
abstract: |
  We introduce the **Born-rule classifier**: a zero-shot
  categorisation rule that predicts
  $\hat{y}(\psi) = \arg\max_{c}\,|\langle c\,|\,\psi\rangle|^2$ ---
  the argmax over class centroids of the squared-amplitude
  probability given by the quantum-mechanical Born rule. The classifier
  requires no gradient training, produces a calibrated probability
  distribution over classes by construction, and is a function only of
  the embedding space and a single centroid per class. It generalises
  two widely used baselines --- nearest-centroid classification and
  zero-shot CLIP-style prototype classification --- by applying the
  Born rule directly as the probability law, which yields a consistent
  treatment of confidence, ties, and out-of-distribution scoring.

  The classifier is the zero-shot labelling primitive of QGI's QAG
  engine [QGI-TR-2026-01]. In that deployment it supplies
  topic, obligation type, severity, and polarity labels for extracted
  rules. The method note here describes the algorithm, its calibration,
  its properties, and the scope within which it can replace a
  supervised classifier. Numerical evaluation across public and
  internal benchmarks is reserved for a forthcoming companion
  evaluation paper.

keywords:
  - zero-shot classification
  - Born rule
  - nearest-centroid classifier
  - prototype networks
  - calibration
  - quantum machine learning
  - embedding models
  - Q-Prime
  - QAG

header-includes:
  - \AtBeginDocument{\hypersetup{pdftitle={A Born-Rule Classifier},pdfauthor={Sam Sammane --- Quantum General Intelligence},pdfsubject={Zero-shot categorisation via squared amplitudes},pdfkeywords={zero-shot classification, Born rule, nearest-centroid, prototype networks, calibration, quantum ML, Q-Prime},pdfcreator={pandoc + xelatex},pdfproducer={QGI typesetting pipeline},bookmarksopen=true,bookmarksnumbered=true}}
---

> **Preprint --- v1.0, 21 April 2026.**
> Not peer-reviewed. Authored by Dr. Sam Sammane, CTO and Founder,
> Quantum General Intelligence, Inc. (`sam@qgi.dev`). Method note
> companion to [QGI-TR-2026-01] and [QGI-TR-2026-02]. Comments
> welcome at `research@qgi.dev`.

# Introduction

Zero-shot classification --- assigning a label to an input without
task-specific training --- has become a staple of production ML
pipelines. The dominant recipe is: embed the input, embed a class
description, rank classes by similarity. Prototypical networks
[@snell2017prototypical] codified this pattern for few-shot learning;
CLIP [@radford2021clip] demonstrated that a large general-purpose
embedding model can serve a broad zero-shot surface over image and
text. Both share a structural choice: the class score is an
*unnormalised* similarity (an inner product, a cosine, or a
negative distance), turned into a probability by a downstream
softmax.

The softmax is convenient but arbitrary. Its temperature is
task-dependent; its calibration degrades off-distribution; it obscures
the semantics of the score at the class level. A different choice is
available, and it is the choice quantum mechanics makes in the
corresponding situation: given a state $\psi$ and a complete set of
class basis vectors $\{|c\rangle\}$, the probability of observing
class $c$ is exactly
$P(c\,|\,\psi) = |\langle c\,|\,\psi\rangle|^2$. No temperature; no
softmax; calibration is the Born rule itself.

This paper presents the Born-rule classifier as the natural choice
when the embedding space is treated as a Hilbert space. We describe
the algorithm, its calibration, and its properties; we position it
against nearest-centroid and prototype-network baselines; and we
reserve numerical comparison for the forthcoming companion evaluation
paper.

# Method

## Setup

Let $\mathcal{H}$ be the embedding space, treated as a Hilbert space
with inner product $\langle \cdot\,|\,\cdot \rangle$ and induced norm
$\|\cdot\|$. Each input $x$ maps to an embedding
$\psi(x) \in \mathcal{H}$. For a task with $K$ classes, we construct a
class centroid $c_k \in \mathcal{H}$ for each $k = 1, \ldots, K$.

The class centroid is not restricted. It can be:

- A single prototype text embedded into $\mathcal{H}$ ("an example
  of spam is...").
- The mean of a small set of prototypes.
- The first principal direction of a larger prototype set.
- A rule-derived centroid extracted from a QAG rule state (the
  regime used inside the QAG engine).

The centroids are normalised so that $\|c_k\| = 1$ for all $k$. The
state $\psi$ is normalised analogously. These normalisations make the
inner products admissible amplitudes under the Born rule.

## The classifier

The predicted label is the argmax over squared amplitudes:

$$
\hat{y}(\psi) \;=\; \arg\max_{k \in \{1,\ldots,K\}} \; |\langle c_k\,|\,\psi\rangle|^2
$$

The predicted probability of class $k$ is

$$
P(k\,|\,\psi) \;=\;
\frac{|\langle c_k\,|\,\psi\rangle|^2}{\sum_{j=1}^K |\langle c_j\,|\,\psi\rangle|^2}.
$$

When the basis $\{c_k\}$ is orthonormal, $\sum_j |\langle c_j\,|\,\psi\rangle|^2 = 1$
and the denominator disappears; the squared amplitudes *are* the
probabilities, exactly as in the quantum setting. When the basis is
non-orthogonal (the common case in practice), the denominator
restores normalisation and the rule degrades gracefully.

## Centroid construction

Given a set of prototypes $\{p_{k,1}, \ldots, p_{k,n_k}\}$ for
class $k$, three constructions are useful in practice.

**Mean centroid.** $c_k = \text{normalise}\big(\frac{1}{n_k}\sum_i p_{k,i}\big)$.
Robust when prototypes are topically homogeneous.

**First principal direction.** $c_k = $ the top-1 eigenvector of the
sample covariance of $\{p_{k,i}\}$. Robust when prototypes are
semantically diverse but share a latent direction.

**Rule-derived centroid.** In the QAG regime, $c_k$ is the
projection of a QAG rule onto the subspace associated with the class
(e.g. a `severity = high` rule gives the severity-high centroid).
The centroid is a byproduct of the ordinary QAG pipeline; no
additional training data is required.

## Calibration

Two calibration properties follow from the Born rule directly.

**Probability simplex.** By the normalisation above,
$\sum_k P(k\,|\,\psi) = 1$ and $P(k\,|\,\psi) \geq 0$ for all $k$.
The predicted distribution is a genuine probability distribution, not
a score transformed by an arbitrary softmax.

**Tie semantics.** When two classes have equal squared amplitudes,
the rule returns a uniform distribution between them. This is the
quantum-mechanical answer to "which outcome will be observed" and
corresponds to the natural Bayesian uncertainty in the argmax.

A one-time **per-task temperature**, if desired, can be applied as
$P(k\,|\,\psi) \propto |\langle c_k\,|\,\psi\rangle|^{2\tau}$ for
$\tau \in (0, +\infty)$. Setting $\tau = 1$ recovers the Born rule;
$\tau > 1$ sharpens; $\tau < 1$ softens. We recommend $\tau = 1$ by
default, and we find in internal evaluation that no task-specific
temperature tuning is required.

## Complexity

For $K$ classes and an embedding dimension $d$, scoring one input
is $O(Kd)$ floating-point operations. Centroid construction is a
one-time $O(n_{\text{max}} d)$ operation. No gradient training is
performed; there is no optimiser state; there is no loss function;
there is nothing to schedule.

The classifier is, operationally, a lookup-and-dot-product. On
commodity GPUs it saturates memory bandwidth before it saturates FLOPs.

# Properties

## No training required

The classifier consumes an embedding model and a set of class
exemplars. Given those, it produces a calibrated classifier. Neither
component requires task-specific training: the embedding model is
frozen; the class exemplars are textual or rule-derived.

## Consistent with the underlying embedding

Any geometry present in the embedding is reflected faithfully by the
classifier. A class whose exemplars are topically coherent produces
a centroid that concentrates; a class with diffuse exemplars
produces a centroid with low expected amplitude on any single input.
Both are honest expressions of the embedding's geometry. A softmax
over logits from a trained head would obscure both effects.

## Calibrated out of the box

The Born rule normalisation is task-independent. A probability of
$0.8$ for class $k$ on one task has the same Bayesian semantics as
$0.8$ on another: the squared projection of $\psi$ onto the $c_k$
subspace, normalised over the available basis. This is the
calibration property a downstream agent needs to make
threshold-independent decisions.

## Resilient to class imbalance

Centroids are computed per class, independently. Class imbalance
affects centroid quality only through the prototype pool for that
class, not through a shared training loss that biases the classifier
toward majority classes. In production, long-tail classes do not
require up-weighting or resampling.

## Ties and out-of-distribution inputs

An input whose amplitude on every centroid is small --- a genuine
out-of-distribution case --- produces a uniform-ish distribution. The
entropy of the predicted distribution is therefore a *free*
OOD signal: no calibration network, no temperature search, no
holdout set required.

# Related work

The classifier is, in the nearest-neighbour sense, close to the
nearest-centroid baseline long used in ML pipelines [@manning2008ir];
the novelty is the choice of the probability law. Prototypical
networks [@snell2017prototypical] use an inner-product or negative
distance as a logit and softmax over the logits to produce
probabilities. CLIP-style zero-shot classification
[@radford2021clip] follows the same pattern with class descriptions
used as prototypes. Nearest-mean classifiers [@mensink2013ncm]
without softmax produce argmax labels but do not produce calibrated
probabilities.

The Born rule has been used as a probability law in quantum machine
learning (e.g. variational quantum classifiers
[@havlicek2019qml]), but the classifier considered in those works is
a parameterised quantum circuit, not a classical embedding model. The
present paper's contribution is the observation that the classical
case --- a frozen general-purpose or purpose-built embedding treated
as a Hilbert space --- is the regime in which the Born rule provides
the calibration and tie-handling that softmax does not.

The quantum cognition literature [@aerts2014quantum;
@bruza2015qcognition] argues, for independent reasons, that squared
amplitudes are a better model of human judgement probabilities than
classical probabilities. The Born-rule classifier is a machine-learning
operationalisation of this argument.

# Evaluation plan

The empirical evaluation of the classifier is reserved for the
forthcoming companion evaluation paper. That paper will report:

- Accuracy on public zero-shot benchmarks (AG News, DBpedia, Yahoo
  Answers, GoEmotions, intent classification suites).
- Accuracy against supervised baselines trained on the same label
  space.
- Calibration metrics (ECE, Brier score) against softmax
  classifiers with and without temperature scaling.
- Performance of the classifier when driven by Q-Prime versus by
  general-purpose embeddings, on the QAG topic, severity, and
  obligation-type tasks.
- Latency and throughput on production GPU hardware.

The numbers are held until that paper; we do not anticipate them
here.

# Limitations

- **Centroid quality.** The classifier is only as good as the
  centroids. Poorly constructed prototypes produce a poor
  classifier, regardless of the probability law. This is a
  data-curation problem, not a method problem.
- **Non-orthogonal basis.** When class centroids are strongly
  correlated (not uncommon for fine-grained taxonomies), the
  classifier's probabilities are not directly interpretable as
  quantum-mechanical probabilities in an orthonormal basis; the
  normalisation restores sensible behaviour but the mathematical
  purity of the Born-rule assignment is degraded.
- **Open-ended labels.** The classifier assumes a fixed, finite
  class set. For open-set problems it needs to be combined with an
  explicit OOD detector (for which the entropy signal is a starting
  point, but not the whole answer).
- **Not a replacement for supervised training when labels are
  abundant.** Supervised heads with large labelled corpora can
  outperform the Born-rule classifier on their specific task. The
  classifier's advantage is the regime where labels are scarce,
  drifting, or expensive to maintain.

# Competing interests

The author is an employee and shareholder of Quantum General
Intelligence, Inc.; the classifier is a component of the commercial
QAG engine.

# References

1. [@snell2017prototypical] Snell, J., Swersky, K., Zemel, R.
   (2017). *Prototypical Networks for Few-shot Learning*. NeurIPS
   2017.
2. [@radford2021clip] Radford, A. et al. (2021). *Learning
   Transferable Visual Models From Natural Language Supervision*
   (CLIP). ICML 2021.
3. [@manning2008ir] Manning, C. D., Raghavan, P., Schütze, H.
   (2008). *Introduction to Information Retrieval*. CUP.
4. [@mensink2013ncm] Mensink, T., Verbeek, J., Perronnin, F.,
   Csurka, G. (2013). *Distance-Based Image Classification:
   Generalizing to New Classes at Near-Zero Cost*. IEEE TPAMI.
5. [@havlicek2019qml] Havlíček, V. et al. (2019). *Supervised
   learning with quantum-enhanced feature spaces*. Nature.
6. [@aerts2014quantum] Aerts, D., Sozzo, S. (2014). *Quantum
   Structure in Cognition*.
7. [@bruza2015qcognition] Bruza, P. D. et al. (2015). *Quantum
   Cognition*. Trends in Cognitive Sciences.
8. [@qgi-tr-01] Sammane, S. (2026). *Quantum-Augmented Generation
   (QAG)*. QGI-TR-2026-01.
9. [@qgi-tr-02] Sammane, S. (2026). *Purpose-Built Embedding Models
   for Rule-Bearing Text*. QGI-TR-2026-02.
10. [@qgi-modelcard] Sammane, S. (2026). *Q-Prime public model
    card*.

# Cite this as

```bibtex
@techreport{sammane2026bornrule,
  author      = {Sam Sammane},
  title       = {A Born-Rule Classifier: Zero-Shot Categorisation via Squared Amplitudes on Class Centroids},
  institution = {Quantum General Intelligence, Inc.},
  number      = {QGI-TR-2026-04},
  year        = {2026},
  month       = {April},
  type        = {Preprint},
  note        = {Version 1.0, 21 April 2026. Not peer-reviewed.}
}
```

# Version history

| Version | Date | Notes |
|---|---|---|
| v1.0 | 21 April 2026 | Initial preprint release. |

---

© 2025--2026 Quantum General Intelligence, Inc. All rights reserved.
