---
title: |
  Purpose-Built Embedding Models for Rule-Bearing Text
subtitle: |
  Why General-Purpose Embeddings Score F1 = 0 on Regulatory Conflict
  and What a Compliance-Grade Model Must Preserve
author:
  - name: Sam Sammane
    email: sam@qgi.dev
    affiliation: Chief Technology Officer and Founder, Quantum General Intelligence, Inc.
date: "21 April 2026"
version: "Preprint v1.0"
abstract: |
  The default assumption in the current retrieval-augmented ecosystem is
  that embedding models are interchangeable: pick any sufficiently large
  one, wire it to a vector store, and tune the prompt. This position
  paper argues that the assumption is false for rule-bearing text ---
  regulations, policies, contracts, clinical guidelines --- and that the
  failure is architectural, not a question of scale.

  We enumerate six properties any embedding layer must preserve in order
  to support automated reasoning over rules: polarity, quantifier
  scope, structured conditions, obligation strength, cross-rule
  correlation, and signed interference. We show that each of these
  properties is, by construction of the standard contrastive training
  objective, absent from general-purpose embedding models trained on
  open-web prose. The canonical pathology follows: "must report" and
  "must not report" are retrieved as near-duplicates; a downstream
  generator trained to synthesise produces a confident and wrong
  answer; the audit record is a liability.

  We then describe the design premise of **Q-Prime**, a purpose-built
  embedding model built around the *signed interference* operation of
  quantum probability. Q-Prime preserves the six properties above as
  separable directions in a 1,536-dimensional Hilbert space, making
  each recoverable by a named projection rather than requiring a
  downstream classifier. The headline empirical consequence: on QGI's
  regulatory-conflict benchmark, classical cosine similarity across
  five widely used embedding models from four organisations scores an
  F1 of **0.000**; the QAG interference signal driven by Q-Prime scores
  an F1 of **1.000**. We discuss what the gap means for practitioners
  and provide a short evaluation checklist for teams choosing an
  embedding model for rule-bearing content. Detailed benchmark
  methodology is reserved for a forthcoming companion evaluation paper.

keywords:
  - embedding model
  - dense retrieval
  - contrastive learning
  - polarity
  - negation
  - quantifier scope
  - regulatory NLP
  - compliance AI
  - conflict detection
  - Born rule
  - Q-Prime
  - QAG

header-includes:
  - \AtBeginDocument{\hypersetup{pdftitle={Purpose-Built Embedding Models for Rule-Bearing Text},pdfauthor={Sam Sammane --- Quantum General Intelligence},pdfsubject={The case for Q-Prime},pdfkeywords={embedding model, polarity, negation, quantifier scope, regulatory NLP, compliance AI, Q-Prime, QAG, Born rule},pdfcreator={pandoc + xelatex},pdfproducer={QGI typesetting pipeline},bookmarksopen=true,bookmarksnumbered=true}}
---

> **Preprint --- v1.0, 21 April 2026.**
> Not peer-reviewed. Authored by Dr. Sam Sammane, CTO and Founder,
> Quantum General Intelligence, Inc. (`sam@qgi.dev`). Companion to the
> canonical QAG engine preprint [QGI-TR-2026-01]. Comments welcome at
> `research@qgi.dev`.

# The embedding layer is load-bearing

Retrieval-Augmented Generation (RAG) is the dominant pattern for
grounding language-model answers in a corpus of documents. Under the
hood, RAG rests on one quiet assumption: the embedding model flattens a
passage to a vector, and nearby vectors are considered interchangeable
for the purpose of retrieval. The assumption is benign for many
applications --- product search, FAQ retrieval, conversational agents
whose factual claims can tolerate small drift. It is catastrophic for
any application that depends on *rules* rather than on prose.

A regulation is not a paragraph. It is a rule: a structured
conditional with a trigger, a set of conditions, an obligation, an
exception, and a sanction. Two rules can be lexically and semantically
very close --- "a broker **must** disclose material conflicts" and "a
broker **must not** disclose material conflicts" --- and differ only
in a token of negation. In the embedding space of every widely
deployed general-purpose model, those two sentences are near-duplicates.
A retriever asked for information about disclosure obligations returns
them together. The downstream generator, told to synthesise, makes a
fluent, citeable, and wrong statement about the obligation.

This is not a hyperparameter problem. It is a property of the training
objective. Scaling the model larger does not make negation *louder*; it
only makes the space around both sentences larger by the same factor.

This paper makes three claims. First, any embedding layer used for
rule-bearing text must preserve six specific properties, which we
enumerate in §2. Second, every widely deployed general-purpose
embedding model violates at least five of the six (§3). Third, a
purpose-built embedding model designed around the *signed interference*
operation of quantum probability preserves them all (§4), with the
empirical consequence that classical cosine similarity scores F1 =
0.000 on the regulatory-conflict benchmark while the QAG interference
signal driven by Q-Prime scores F1 = 1.000 (§5).

The argument extends straightforwardly to AI agents: an agent that
accumulates rules, preferences, and observations as memory items
suffers exactly the same pathology if its memory store uses a
general-purpose embedding. We discuss this extension briefly at the
end, but the companion paper [QGI-TR-2026-03] is the canonical
treatment of agent memory.

# Six properties any compliance-grade embedding must preserve

A reasoning-first pipeline over rule-bearing text --- such as QGI's
QAG engine [QGI-TR-2026-01] --- projects the embedding of a rule onto
named subspaces to expose conflict, overlap, redundancy, and related
observables. The projection operation assumes that the quantities
projected *exist* as separable directions in the embedding space. If
they do not, no downstream layer can recover them. We enumerate the
six properties required:

## Polarity as a first-class parameter

Two clauses that differ only by a negation are *opposite*, not
*similar*. The distinction is not a matter of degree; it is a matter
of sign. The embedding representation must carry polarity as a
measurable direction, such that "must report" and "must not report"
have inner products of opposite sign under the relevant projection.
Token-level perturbations of a general-purpose embedding are far too
small to constitute a sign change under cosine similarity.

## Scope and quantifier sensitivity

Rule bodies routinely use universal ("all", "every", "any",
"no") and existential ("some", "at least one", "there exists")
quantifiers. The distinction determines whether a clause applies at
all. "All employees must report" and "some employees must report"
differ by a single token but by a universe of obligation. The
embedding must keep these apart in a direction recoverable by the
HSC projection; absent that, downstream reasoning confuses "applies
to me" with "applies to somebody".

## Structured conditions

A regulatory clause takes the internal shape *(trigger, condition,
action, exception, scope)*. An embedding that captures only the
surface sentence throws the structure away; a retrieval system that
uses that embedding cannot express a query like "give me all rules
whose *exception* mentions the jurisdiction of California". Q-Prime
exposes each structural role as a first-class direction.

## Obligation strength

"Shall", "must", "may", "should", "recommended", "encouraged", "is
expected to", "is required to", "may at its discretion" form a
continuous spectrum of obligation strength with legal consequence.
Two clauses identical except for the modal verb are *not*
interchangeable. The embedding must place them at different points
along an obligation axis; classical training objectives that reward
topical proximity do not provide this axis.

## Cross-rule correlation (entanglement)

The compliance status of rule $R$ frequently depends on the activation
status of other rules. An embedding that represents each rule in
isolation loses this information. Q-Prime's representation supports
joint states: the state of rule $R$ is a vector in the same space as
the joint state of ($R_1$, $R_2$), so the "does this rule conflict
with this other rule?" question is well-posed as a single projection.

## Signed interference

A correlation signal that reports only magnitude is useless for
conflict detection, because conflict is defined by opposite orientation,
not merely by non-similarity. The representation must carry
orientation such that the inner product of two related rules can be
positive (reinforcement) or negative (cancellation). Classical
embeddings trained against cosine similarity are, by construction,
blind to sign.

## Why these six, and only these six

The list is derived from the operations QAG's downstream layer needs
to perform, not from intuitions about natural language. Each property
corresponds to a named observable or to a requirement of the
interference calculation. Omitting any one of them causes the
corresponding downstream signal to degrade from useful to noise.

# Why general-purpose embeddings fall short

General-purpose embedding models --- Sentence-BERT
[@reimers2019sbert], BGE [@xiao2024cpack], E5 [@wang2022e5], GTE
[@li2023gte], OpenAI text-embedding-3, Cohere embed-v3 --- are trained
on vast open-web corpora against a contrastive objective. The
objective rewards the model for mapping semantically similar pairs
close and dissimilar pairs far apart. They are excellent at that task;
on MTEB [@muennighoff2023mteb] and BEIR [@thakur2021beir] they
dominate the leaderboards.

For rule-bearing text three structural limitations follow from the
objective:

**Negation is a small perturbation.** In standard embedding models,
inserting "not" shifts the vector by a distance small relative to the
distance between two completely different topics. Under cosine
similarity, "must report" and "must not report" are near-duplicates.
This is not a defect of any one model; it is a property of training
on text where negation is a rare event relative to topical variation.

**Quantifier scope is not encoded.** There is no trained objective
that rewards the model for keeping "all" and "some" apart. They share
context, syntax, and often neighbouring tokens. In the contrastive
objective, pairs that differ only in quantifier scope are mostly
treated as positives.

**There is no orientation, only magnitude.** Cosine distance is a
symmetric similarity; it has no sign. Architectures trained with
cosine objectives push similar things together and dissimilar things
apart, but they do not learn *signed* relationships. Interference
--- the central operation a compliance pipeline depends on --- is
therefore not available.

These are not problems of insufficient scale. They are properties of
the objective. Training on larger corpora, or embedding in higher
dimensions, enlarges the whole space proportionally and leaves the
negation delta --- and the sign --- exactly as small as it was.

# What Q-Prime does differently

Q-Prime is QGI's purpose-built embedding model for the QAG pipeline.
It is built on a different premise from general-purpose embeddings:
instead of collapsing each passage into a single similarity-optimised
vector, it emits a **quantum-structured representation** that
preserves the six properties of §2 as separable directions. The name
is not an analogy. Q-Prime applies the operator algebra and
probability rule of quantum mechanics --- Hilbert space, the Born
rule, superposition, signed interference --- to language
representations, executed on classical GPUs.

Two design choices follow.

**Entangled superpositions as the primary object of training.** A
single regulatory clause typically asserts several things at once: an
obligation *and* an exception *and* a sanction. Q-Prime represents
the clause as a *linear combination* of those component states in a
way that makes each component recoverable by projection onto a
named subspace. Contrast with a classical embedding that trains on
the whole sentence as a single target, forcing all components into an
averaged location.

**Signed inner products as a first-class operation.** The inner
product $\langle a\,|\,b \rangle$ in Q-Prime's space can be positive,
zero, or negative. Two clauses whose polarity agrees produce a
positive inner product; two that contradict produce a negative one;
two that are unrelated produce a small-magnitude one. The downstream
**interference signal** is exactly
$\text{polarity}(a,b)\cdot\langle a\,|\,b\rangle$.

Two practical properties follow:

- **Q-Prime is more compact than a classical embedding of equivalent
  content.** The relational structure lives in the state itself,
  rather than being padded into extra dimensions. Fewer numbers carry
  more signal, so downstream search and reranking are cheaper at any
  given accuracy target.
- **Q-Prime exposes parameters that cosine similarity cannot see.**
  Polarity, scope, conditions, obligation, and cross-rule dependency
  are each recoverable as a named projection of the state, rather
  than being smeared across the whole vector as an artefact of
  training-corpus statistics.

Q-Prime is not the only possible design that satisfies the six
properties. It is, to our knowledge, the first one built and
productised.

# The headline empirical result

QGI maintains a regulatory-conflict benchmark built from real
regulatory and policy corpora. The benchmark measures the ability of
a retrieval system to correctly identify pairs of rules that
materially conflict --- the single task compliance teams cannot
afford to get wrong.

| Signal | F1 |
|---|---|
| Classical cosine similarity (five widely used embedding models from four organisations) | **0.000** |
| QAG interference signal (Q-Prime + polarity) | **1.000** |

The result is categorical, not incremental. Cosine similarity across
the broad family of commercially deployed embedding models does not
merely *underperform* --- it achieves **zero** F1. The signal the
task requires is simply absent from those representations.

Three clarifications are worth stating directly.

First, the interference *effect* --- that signed interference
separates same-polarity and opposite-polarity clauses --- is a
property of the language of regulation itself, not of any one model.
Internal evaluation shows it replicates across every embedding family
we have tested (dimensionalities from 384 to 3,072, drawn from four
organisations) and across out-of-domain corpora well outside the
regulatory training distribution (medical safety, educational
curricula, engineering safety, research ethics). Q-Prime's role is to
make the effect reliable under production conditions.

Second, conflict is only one of the quantum observables this embedding
geometry supports. The same state space supports a **Born-rule
classifier** --- $\arg\max_c\,|\langle c\,|\,\psi\rangle|^2$ --- for
zero-shot labelling of topic, obligation type, and severity. The
classifier is training-free and calibrated by construction; it is the
subject of the companion method note [QGI-TR-2026-04].

Third, the headline numbers above are drawn from a held-out
benchmark and are released under evaluation agreement. Full
methodology --- corpora, embedding backbones, extractor-agnostic
validation, out-of-domain generalisation, and GPU throughput --- will
appear in a **forthcoming companion evaluation paper**. The numbers
above are neither the full extent of QGI's internal evaluation nor
marketing claims divorced from methodology.

# Implications

## For RAG integrators

The assumption that embedding models are interchangeable is false for
regulated content. The embedding layer is the location where the
features that determine correctness either exist or do not exist.
Replacing the generator in a pipeline whose embedding collapses
polarity is a cosmetic change; replacing the embedding is the
structural one.

## For evaluators

When benchmarking a retrieval-augmented system for rule-bearing
content, ask for the **conflict F1**, not recall@k, nDCG, or BLEU.
The quantity that determines whether the system will embarrass you in
front of an auditor --- or mislead an agent in production --- is its
ability to tell contradictory clauses apart. Everything else is
downstream.

## For teams building their own model

A purpose-built embedding model is substantially more work than
fine-tuning a general-purpose one. That is the honest assessment. It
is also, in our experience, the difference between a demo that looks
convincing and a system a regulator or a long-running agent accepts.
Teams without the resources to train a purpose-built embedding
should evaluate Q-Prime through one of the access paths in §9.

# Evaluating an embedding model for rule-bearing text: a checklist

The following checklist is the minimum practitioners should run
before adopting any embedding model for rule-bearing text. None of
the items require proprietary data; all of them can be run on any
handful of rules from any regulation.

1. **Negation discrimination.** For a pair of clauses that differ
   only by a single negation token, compute cosine similarity. If
   similarity is above 0.95, the embedding will merge contradictions.
2. **Quantifier discrimination.** For a pair of clauses that differ
   only by substituting "all" $\rightarrow$ "some", or "every"
   $\rightarrow$ "at most one", compute similarity. If similarity is
   above 0.95, the embedding is insensitive to scope.
3. **Obligation-strength discrimination.** For a pair of clauses that
   differ only by substituting the modal verb ("shall" $\rightarrow$
   "may", "must" $\rightarrow$ "should"), compute similarity. If it
   is above 0.95, the embedding is insensitive to obligation
   strength.
4. **Sign of inner product.** Compute the raw inner product (not the
   absolute value) between two clauses known to be in conflict.
   Standard cosine-trained models return only non-negative values.
   If the embedding cannot produce a negative inner product, it has
   no sign channel.
5. **Structured query.** Attempt to retrieve "rules whose exception
   references California". If the embedding does not expose structural
   roles, this query degrades to topical retrieval and will miss the
   structural intent.
6. **Conflict F1 on a tiny hand-built set.** Construct 20 pairs of
   clauses drawn from a real regulation; 10 pairs are in true
   conflict, 10 are unrelated. Run the system end-to-end. Any F1 below
   0.7 should disqualify the embedding for production use.

A model that fails items 1--4 cannot be rescued by prompt engineering
downstream.

# Related work

The full landscape review lives in the companion preprint
[QGI-TR-2026-06]; here we note only the items most directly
relevant to the embedding argument.

The contrastive-pretraining family (Sentence-BERT
[@reimers2019sbert], E5 [@wang2022e5], BGE [@xiao2024cpack],
GTE [@li2023gte]) establishes the design pattern whose limitations
this paper identifies. MTEB [@muennighoff2023mteb] and BEIR
[@thakur2021beir] are the leaderboards the family has been trained
to climb.

The quantum NLP lineage --- DisCoCat [@coecke2010discocat], QNLP
[@kartsaklis2021lambeq], quantum cognition [@aerts2014quantum;
@bruza2015qcognition] --- developed the mathematics Q-Prime applies
but did not pursue production-scale embedding models for regulatory
text.

The contract and legal NLP community has produced CUAD
[@hendrycks2021cuad] and LexGLUE [@chalkidis2022lexglue], both
of which include tasks that depend on the properties enumerated in
§2 and both of which demonstrate, indirectly, the ceiling that
general-purpose embeddings hit on structured legal language.

# Limitations

- **Managed-API only.** Q-Prime is not distributed as weights. Teams
  requiring fully local, open-weights embedding stacks are not the
  target audience of v1.0.
- **English-first.** The v1.0 training corpus is predominantly
  English regulatory and policy text. Other languages are on the
  roadmap but behind public release.
- **Not a general replacement for general-purpose embeddings.** For
  tasks where the six properties above are irrelevant (pure topical
  retrieval, product search, FAQ), a well-tuned general-purpose
  embedding will remain competitive.
- **Benchmark scope.** The headline F1 result is on an internal
  regulatory-conflict benchmark. Full out-of-domain numbers are in
  the forthcoming companion evaluation paper.

# Access

- **Evaluation tokens** --- `contact@qgi.dev`. 90 days free under
  QGI Commercial Model License v1.0, §3.
- **OpenRouter listing** --- Q-Prime is listed as part of the QAG
  public beta.
- **Enterprise agreements** --- `contact@qgi.dev`.

Weights are not distributed; the model is accessed as a managed API.

# Ethics and dual use

An embedding model that reliably detects regulatory contradictions
also reliably detects regulatory loopholes. The model card's
Acceptable Use Policy restricts uses that produce legally-binding
automated decisions without certified human review, and the QGI
Commercial Model License v1.0 enforces the policy contractually. See
[@qgi-license] for details.

# Data availability

Benchmark corpora are proprietary and released under evaluation
agreement. Methodology and reproduction artefacts accompany the
forthcoming companion evaluation paper.

# Competing interests

The author is an employee and shareholder of Quantum General
Intelligence, Inc., and has a commercial interest in Q-Prime and the
QAG engine.

# References

1. [@reimers2019sbert] Reimers, N., Gurevych, I. (2019).
   *Sentence-BERT*. EMNLP 2019.
2. [@xiao2024cpack] Xiao, S., Liu, Z., Zhang, P., Muennighoff, N.
   (2024). *C-Pack: Packed Resources For General Chinese Embeddings*
   (BGE family). SIGIR 2024.
3. [@wang2022e5] Wang, L., Yang, N., Huang, X. et al. (2022). *Text
   Embeddings by Weakly-Supervised Contrastive Pre-training*.
   arXiv:2212.03533. (E5.)
4. [@li2023gte] Li, Z., Zhang, X., Zhang, Y. et al. (2023). *Towards
   General Text Embeddings with Multi-stage Contrastive Learning*.
   arXiv:2308.03281. (GTE.)
5. [@muennighoff2023mteb] Muennighoff, N., Tazi, N., Magne, L.,
   Reimers, N. (2023). *MTEB: Massive Text Embedding Benchmark*.
6. [@thakur2021beir] Thakur, N., Reimers, N., Rücklé, A. et al.
   (2021). *BEIR*. NeurIPS Datasets 2021.
7. [@coecke2010discocat] Coecke, B., Sadrzadeh, M., Clark, S. (2010).
   *Mathematical Foundations for a Compositional Distributional
   Model of Meaning*. Linguistic Analysis 36.
8. [@kartsaklis2021lambeq] Kartsaklis, D., Fan, I., Yeung, R. et al.
   (2021). *lambeq*. arXiv:2110.04236.
9. [@aerts2014quantum] Aerts, D., Sozzo, S. (2014). *Quantum
   Structure in Cognition*. Quantum Interaction 2014.
10. [@bruza2015qcognition] Bruza, P. D., Wang, Z., Busemeyer, J. R.
    (2015). *Quantum Cognition*. Trends in Cognitive Sciences.
11. [@hendrycks2021cuad] Hendrycks, D. et al. (2021). *CUAD*. NeurIPS
    2021.
12. [@chalkidis2022lexglue] Chalkidis, I. et al. (2022). *LexGLUE*.
    ACL 2022.
13. [@qgi-tr-01] Sammane, S. (2026). *Quantum-Augmented Generation
    (QAG): A Reasoning-First Memory Infrastructure*. QGI-TR-2026-01.
14. [@qgi-tr-03] Sammane, S. (2026). *Conflict-Aware Memory for AI
    Agents*. QGI-TR-2026-03.
15. [@qgi-tr-04] Sammane, S. (2026). *A Born-Rule Classifier*.
    QGI-TR-2026-04.
16. [@qgi-tr-06] Sammane, S. (2026). *Beyond Retrieval-Augmented
    Generation: A Review*. QGI-TR-2026-06.
17. [@qgi-modelcard] Sammane, S. (2026). *Q-Prime public model
    card*.
18. [@qgi-license] Quantum General Intelligence, Inc. (2026). *QGI
    Commercial Model License v1.0*.

# Cite this as

```bibtex
@techreport{sammane2026qprime,
  author      = {Sam Sammane},
  title       = {Purpose-Built Embedding Models for Rule-Bearing Text: Why General-Purpose Embeddings Score F1 = 0 on Regulatory Conflict},
  institution = {Quantum General Intelligence, Inc.},
  number      = {QGI-TR-2026-02},
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
