---
title: |
  Beyond Retrieval-Augmented Generation
subtitle: |
  A Review of Reasoning-First Alternatives for Rule-Bearing Content
  and the Emergence of Quantum-Augmented Generation (QAG)
author:
  - name: Sam Sammane
    email: sam@qgi.dev
    affiliation: Chief Technology Officer and Founder, Quantum General Intelligence, Inc.
date: "21 April 2026"
version: "Preprint v1.0"
abstract: |
  Retrieval-Augmented Generation (RAG) has been the dominant pattern
  for grounding language-model output in external knowledge since its
  introduction in 2020. In the intervening five years, the pattern
  has accreted a large family of variants: corrective, reflective,
  hypothetical-query-driven, tree-structured, graph-augmented, and
  hypergraph-augmented. A parallel line of work has focused on agent
  memory --- OS-as-memory, temporal knowledge graphs, lightweight
  memory APIs, and reflection-based consolidation. A third line has
  pushed toward longer context windows and the engineering that
  accompanies them. A fourth has improved the embedding layer that
  all three lines sit on. A fifth, older, line has developed
  compositional and quantum approaches to semantics.

  This survey maps the landscape and asks a structural question:
  which of these approaches, alone or in combination, addresses the
  class of content that regulated enterprises and long-running AI
  agents both generate --- *rule-bearing* content whose correctness
  depends on polarity, quantifier scope, obligation strength, and
  cross-rule dependency? We propose a small set of criteria (§8),
  evaluate each family against them (§9), and identify the structural
  gap none of the families fills: the combination of deterministic
  parsing, immutable versioned graph state, conflict-as-first-class
  edge, purpose-built embedding preserving sign and scope, and
  audit-replayable trace. We describe **Quantum-Augmented Generation
  (QAG)** [QGI-TR-2026-01] as one concrete
  realisation of a reasoning-first alternative that fills this gap.

  The survey is intentionally positioning: the author is an interested
  party. Criteria, gap analysis, and taxonomy are offered as
  load-bearing artefacts a reader can disagree with on their own
  merits; the references are comprehensive enough that a reader can
  reproduce or contest the positioning.

keywords:
  - Retrieval-Augmented Generation
  - RAG
  - knowledge graphs
  - agent memory
  - long context
  - embedding models
  - quantum NLP
  - reasoning-first
  - QAG
  - compliance AI
  - survey

header-includes:
  - \AtBeginDocument{\hypersetup{pdftitle={Beyond RAG: A Review of Reasoning-First Alternatives},pdfauthor={Sam Sammane --- Quantum General Intelligence},pdfsubject={Survey of reasoning-first alternatives to RAG},pdfkeywords={RAG, knowledge graphs, agent memory, long context, embedding models, quantum NLP, QAG, compliance AI, survey},pdfcreator={pandoc + xelatex},pdfproducer={QGI typesetting pipeline},bookmarksopen=true,bookmarksnumbered=true}}
---

> **Preprint --- v1.0, 21 April 2026.**
> Not peer-reviewed. Authored by Dr. Sam Sammane, CTO and Founder,
> Quantum General Intelligence, Inc. (`sam@qgi.dev`). Companion to
> [QGI-TR-2026-01]. Comments and pointers to missed prior work are
> welcome at `research@qgi.dev`.

# Introduction

Five years after Lewis et al. [@lewis2020rag] formalised
Retrieval-Augmented Generation, the dominant architecture for
grounding a language model in external knowledge is still some
variant of retrieve-then-stuff: embed a corpus, index the
embeddings, retrieve the top-$k$ nearest neighbours of a query,
concatenate them into a prompt, generate. The pattern has proved
remarkably productive. Production deployments grounded in RAG
variants now support chatbots, copilots, search assistants,
customer-service automation, and a growing fraction of agent
frameworks.

The pattern has also accumulated a recognisable failure mode in
settings where *rules* rather than *prose* determine correctness ---
regulations, policies, contracts, clinical guidelines, and the
accumulated rule-like memory of a long-running agent. The failure
mode is visible at three levels of the stack. The embedding layer
collapses polarity, scope, and obligation strength. The retrieval
layer returns similar-but-contradictory passages as indistinguishable
neighbours. The generator downstream produces confident, fluent, and
wrong answers.

The purpose of this survey is to position the large body of work
that has responded to this failure mode, to organise it by the
primitive each family changes, and to identify the structural gap
that remains. We cover six families:

1. **RAG variants** that optimise the retrieve-then-stuff primitive.
2. **Knowledge-graph-grounded** systems that replace or augment the
   vector index with a graph.
3. **Agent memory systems** that address long-running memory as a
   first-class concern.
4. **Long-context models** and the context-engineering practice
   around them.
5. **Embedding models** that have improved faster than any other
   layer.
6. **Compositional / quantum NLP** that has developed much of the
   mathematics reasoning-first systems rest on.

We then articulate criteria (§8), a gap analysis (§9), and a
positioning of Quantum-Augmented Generation (§10). The survey is
structurally biased: the author is an employee and shareholder of
the company that ships the QAG engine. We attempt to compensate by
being explicit about the criteria, quoting primary sources, and
inviting counter-positioning.

# Family 1: RAG and its direct variants

## The original formulation

The foundational paper [@lewis2020rag] introduced dense retrieval
over Wikipedia, fused with generation via marginalisation over
retrieved documents. DPR [@karpukhin2020dpr] established dense
passage retrieval as a competitive alternative to BM25
[@robertson2009bm25]. Gao et al. [@gao2024ragsurvey] survey the
space as of 2024 and distinguish *naive*, *advanced*, and *modular*
RAG; the modular variant is closest to modern production systems.

## Corrective, reflective, and self-critical RAG

Self-RAG [@asai2024selfrag] interleaves generation with retrieval
and critique. CRAG [@yan2024crag] adds a correctness evaluator
between retrieval and generation. FLARE [@jiang2023flare] triggers
retrieval on low-confidence generations. RAFT [@zhang2024raft]
fine-tunes the generator to attend correctly to retrieved content.
These are improvements of the retrieve-then-stuff pattern along the
axis of *when* and *how often* to retrieve; they do not change
*what* is retrieved.

## Hypothetical-query retrieval

HyDE [@gao2023hyde] embeds a hypothetical answer rather than the
query, improving retrieval for long-tail questions. Query expansion
and pseudo-relevance feedback follow the same pattern. These
interventions move the query closer to a useful embedding but do
not change the structure of the corpus embedding.

## Tree and hierarchical retrieval

RAPTOR [@sarthi2024raptor] builds a tree of abstractive summaries
over chunks. MetaRAG [@zhou2024metarag] introduces meta-rewriting
steps. These allow multi-granularity retrieval --- section, document,
paragraph --- but still retrieve passages, and still use cosine
similarity as the primary scoring function.

## Evaluation frameworks

RAGAS [@es2024ragas] and ARES [@saad2023ares] propose automated
evaluation metrics for RAG systems (faithfulness, relevance, answer
quality). These frameworks themselves illustrate the gap: the
metrics largely measure whether the retrieved chunks *support*
the answer, not whether the retrieved chunks *contradict* each
other or *contradict* the source of truth.

## What the RAG family does not solve

Across all variants, three properties remain absent:

- **Conflict detection as a first-class signal.** A retriever that
  returns two contradictory passages does so indifferently; no
  downstream layer flags the contradiction before the generator
  sees it.
- **Polarity-aware similarity.** Cosine similarity discards sign;
  negation is a small perturbation under every widely deployed
  general-purpose embedding.
- **Deterministic extraction.** Retrieval is reproducible (top-$k$
  over a fixed index), but the upstream chunking and the downstream
  generation are not; prompt, temperature, and model-version changes
  all drift.

# Family 2: Knowledge-graph-grounded generation

## Classical knowledge graphs

Entity-relation graphs (Freebase, DBpedia, Wikidata [@vrandecic2014wikidata])
supplied structured knowledge to QA systems long before RAG. The
pattern has been revived under the name **KG-RAG** or
**graph-augmented RAG**.

## LLM-built graphs

GraphRAG [@edge2024graphrag] uses an LLM to extract entities and
relations from a corpus, cluster them into communities, and
summarise for query-focused retrieval. LightRAG
[@guo2024lightrag] simplifies the ingestion path. Pan et al.
[@pan2024kgllm] survey the space as of 2024.

Two properties of LLM-built graphs matter for rule-bearing content.
First, the graph is *non-deterministic* at ingestion: the same
corpus produces different graphs under different prompts, different
sampling temperatures, and different LLM versions. Second, *conflict*
is not a modelled edge type: two rules that disagree either end up
as one merged node (if the LLM decides they are the same fact) or as
two independent nodes with no edge indicating disagreement.

## Hypergraph-augmented generation

HyperGraphRAG [@luo2024hypergraphrag] extends graph-augmented
retrieval to hyperedges, permitting $N$-ary relations. This is a
step toward the data model required for rule-bearing content, but
the hypergraph is still LLM-built, mutable, and un-versioned, and
conflict is still not a first-class edge type.

## What graph-augmented systems do not solve

The graph primitive is the right primitive, but three gaps remain:

- Graph construction uses an LLM and is therefore
  non-deterministic.
- Conflict is not a first-class edge type.
- Versioning and immutability are not modelled; audit replay is not
  supported.

Deterministic graph construction (QNR2) plus first-class conflict
edges (QHG) plus immutable versioning [@qgi-tr-05] is the
combination that closes these gaps; it is described in
[QGI-TR-2026-05].

# Family 3: Agent memory

## OS-as-memory

MemGPT [@packer2023memgpt] (productised as Letta) models an LLM as
an operating system with tiered primary and archival memory. The
novelty is capacity management; consistency between archival items
is not modelled.

## Temporal knowledge graphs

Zep [@rasmussen2024zep] builds a temporally-indexed knowledge graph
over chat-derived facts. Facts carry timestamps; later facts can
supersede earlier ones. Supersession is computed by LLM inference,
losing determinism; conflict detection is partial.

## Lightweight memory APIs

mem0 [@mem0], LangGraph's memory primitive, and LlamaIndex's
memory module provide an ergonomic layer over vector stores with
optional importance scoring and summarisation. Capacity is managed;
consistency is not.

## Reflective / generative memory

Generative Agents [@park2023generative] introduced reflection loops
that summarise recent memory. Voyager [@wang2023voyager] maintains
a growing skill library for an embodied agent. Reflection can
incidentally surface contradiction but does so via LLM inference,
not as a first-class signal, and only within the reflection window.

## What agent memory systems do not solve

The conflict-detection gap identified in [QGI-TR-2026-03] is the
structural gap of the entire family. No system in this family
provides a *signed* signal between memory items; the consolidation
routines downstream therefore cannot be principled, and the agent
ends up believing multiple contradictory items at once, or neither,
depending on heuristic retrieval order.

# Family 4: Long context and context engineering

## Context-length extensions

Gemini 1.5 [@gemini15] extended context to 1 M tokens; GPT-4.5,
Claude 3.5, and successors followed. YaRN [@peng2024yarn] and Ring
Attention [@liu2024ring] extend context algorithmically; StreamingLLM
[@xiao2023streaming] addresses arbitrary-length streaming.

## Lost-in-the-middle and positional effects

Liu et al. [@liu2024lostmiddle] showed that long contexts are not
accessed uniformly: models strongly prefer content near the
beginning and the end. Long contexts *relax* the retrieval bottleneck
but do not eliminate the *selection* bottleneck; given a fixed
budget, which items should enter, and in what order?

## Context engineering as a practice

The informal practice of context engineering --- ordering, truncating,
compressing, and annotating context before it enters the prompt ---
has emerged as a craft. Tools like LangGraph's context nodes and
OpenAI's Assistants tool ecosystem operationalise parts of it. The
quantitative basis for the craft --- what *should* enter the prompt
--- is exactly where reasoning-first signals add value.

## What long-context approaches do not solve

Longer context does not detect contradictions within the context.
The pathology it does solve is "my document does not fit in the
prompt"; the pathology it leaves untouched is "the items selected
for the prompt silently contradict each other". A Born-rule
relevance classifier plus a conflict-aware packer address the second
pathology directly and are orthogonal to context-length extensions.

# Family 5: Embedding models

## Contrastive pretraining

Sentence-BERT [@reimers2019sbert] established siamese training on
NLI data; BGE [@xiao2024cpack], E5 [@wang2022e5], GTE
[@li2023gte], and the commercial embeddings from OpenAI, Cohere,
and Anthropic extend the pattern with larger corpora.

## Benchmarking

MTEB [@muennighoff2023mteb] and BEIR [@thakur2021beir] drive the
leaderboards. Both are excellent for the tasks they measure, both of
which reward topical similarity.

## The negation and scope limitation

Work on negation sensitivity in embeddings [@kassner2020negation;
@hossain2022negation] documents the core limitation: contrastive
training over open-web prose does not reward keeping "must" and
"must not" apart, nor "all" and "some". The limitation is reflected
directly in downstream compliance pipelines.

## Purpose-built embeddings

Domain-specific embeddings --- Legal-BERT [@chalkidis2020legalbert],
SciBERT [@beltagy2019scibert], BioBERT [@lee2020biobert] ---
improve *topical* coverage of specialised domains but retain the
contrastive objective's sign-blindness. Purpose-built embeddings
that target the specific properties rule-bearing content depends on
--- polarity, scope, obligation, cross-rule dependency --- remain
rare; Q-Prime [@qgi-tr-02] is one such model.

## What embedding advances do not solve

Scale does not help; larger embeddings do not make negation louder
relative to topical variance. The gap closes only when the objective
is redesigned around the features downstream reasoning depends on.

# Family 6: Compositional and quantum NLP

## Compositional distributional semantics

DisCoCat [@coecke2010discocat] proposed compositional distributional
models of meaning based on category theory and tensor products.
Subsequent work on pregroup grammars [@lambek2008], compact closed
categories [@coecke2013], and string diagrams [@coecke2017pic]
developed the mathematical machinery.

## Quantum NLP

Coecke, Kartsaklis, and collaborators [@kartsaklis2021lambeq]
developed quantum-circuit models of language. Meichanetzidis et al.
[@meichanetzidis2020qnlp] ran small QNLP experiments on real quantum
hardware. The lineage is mathematically rich; production-scale
deployments to compliance and agent-memory workloads have, until
recently, been absent.

## Quantum cognition

Aerts, Sozzo, Bruza, and Busemeyer [@aerts2014quantum;
@bruza2015qcognition] argue that quantum-mechanical probability
models human judgement better than classical probability in a range
of situations (conjunction effects, order effects, interference
effects). The line of work is empirical; its connection to ML is
indirect but supportive.

## What compositional / quantum NLP has *already* solved

The formalism. A Hilbert space with an inner product, the Born rule
as a probability law, and signed interference between states are all
mathematically well-defined and have been developed for decades.
QAG's contribution is not the formalism; it is the engineering
realisation on commodity GPUs for production workloads, and the
combination of the formalism with deterministic parsing, immutable
hypergraph state, and an audit-replayable trace.

# Criteria for rule-bearing content

We propose seven criteria any system handling rule-bearing content
should meet. None is proprietary; each is structural.

1. **Deterministic extraction.** The same input produces the same
   structured representation every time.
2. **First-class conflict detection.** Contradictions between items
   are surfaced as a typed event, not a retrieval artefact.
3. **Polarity-aware similarity.** Same-polarity and opposite-polarity
   items produce signed values, not magnitude-only distances.
4. **Quantifier and scope sensitivity.** "All" and "some" are
   separable in the representation.
5. **Immutable, versioned state.** Past answers can be replayed
   against the state they were produced against.
6. **Grounded output.** Every claim traces to a specific rule or
   item in the source.
7. **Replayable audit trace.** The pipeline's decisions are
   reproducible from stored inputs.

These criteria are domain-independent. A research agent, a customer
service agent, a compliance pipeline, a clinical decision-support
system, and a legal-due-diligence tool all benefit from every
criterion; none is regulated-industry-specific.

# Gap analysis

The matrix below summarises which family addresses which criterion.
A check mark indicates that the primary systems in the family satisfy
the criterion by design; partial marks indicate partial or conditional
support; dashes indicate the criterion is not addressed.

|  | Determ. extr. | First-class conflict | Polarity-aware | Quantifier-scope | Immutable + versioned | Grounded output | Replayable trace |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| RAG family (§3) | --- | --- | --- | --- | --- | partial | --- |
| KG-augmented (§4) | --- | --- | --- | --- | --- | partial | --- |
| Hypergraph-augmented (§4) | --- | --- | --- | --- | --- | partial | --- |
| Agent memory (§5) | --- | --- | --- | --- | partial | partial | partial |
| Long context (§6) | n/a | --- | --- | --- | n/a | partial | --- |
| General embeddings (§7) | n/a | --- | --- | --- | n/a | n/a | n/a |
| Domain embeddings (§7) | n/a | --- | --- | --- | n/a | n/a | n/a |
| Compositional / QNLP (§8) | partial | partial | partial | partial | n/a | n/a | n/a |
| **QAG (§10)** | **yes** | **yes** | **yes** | **yes** | **yes** | **yes** | **yes** |

The gap is the collection of empty cells above the QAG row. No
system in families 1--5 satisfies all seven criteria. Family 6 has
the mathematics but not the engineering realisation at production
scale.

# Quantum-Augmented Generation as one answer

We position QAG as one --- deliberately not *the* --- answer to the
gap the matrix identifies. QAG's contribution is not the formalism,
which is standard quantum mechanics, nor the hypergraph, which is a
known data structure, nor the deterministic parser, which follows
engineering practice. QAG's contribution is the combination, operated
under a discipline that maps each criterion to a specific system
component:

- **Deterministic extraction.** QNR2, the LLM-free parser.
- **First-class conflict detection.** HSC's Conflict signal plus the
  QHG conflict edge.
- **Polarity-aware similarity.** Q-Prime, the purpose-built embedding
  model.
- **Quantifier and scope sensitivity.** Q-Prime's training objective.
- **Immutable, versioned state.** QHG snapshots and supersession
  edges.
- **Grounded output.** Source-grounding constraint on every QAG
  answer.
- **Replayable trace.** The QAG audit-trace component.

The full architecture is described in [QGI-TR-2026-01]; the embedding
position is [QGI-TR-2026-02]; the agent-memory realisation is
[QGI-TR-2026-03]; the classifier is [QGI-TR-2026-04]; the data model
is [QGI-TR-2026-05]. Other architectures satisfying the same
criteria are possible and welcome.

# Research agenda

The gap analysis suggests five research directions that are, in our
view, open.

1. **Deterministic extractors with broader coverage.** QNR2 covers
   rule-bearing content; casual narrative requires additional work.
2. **Open-weights purpose-built embeddings.** Q-Prime is a managed
   API; an open-weights analogue would accelerate community
   progress.
3. **Benchmarks for conflict and scope.** Public benchmarks that
   measure conflict F1, quantifier sensitivity, and obligation
   strength directly would let the embedding field improve on the
   right objective. The current leaderboards reward topical
   similarity.
4. **Interference signals in multi-modal content.** The quantum NLP
   machinery extends to image and audio representations; practical
   signal layers there are underdeveloped.
5. **Theoretical analysis of when contrastive objectives can
   encode sign.** Under what objective modifications does a
   contrastive embedding acquire polarity sensitivity without a
   full retraining?

# Limitations of this survey

- **Author bias.** Sam is an employee and shareholder of QGI. The
  criteria and gap analysis are chosen to foreground where QAG
  differs from prior work. Readers should test the criteria against
  their own workloads.
- **English-first bibliography.** A substantial non-English
  literature on quantum cognition, compositional semantics, and
  regulatory NLP is not represented proportionally.
- **Five-year horizon.** The RAG, agent-memory, and long-context
  families are moving quickly; specific papers will be superseded
  between this preprint's release and its peer review. The criteria
  are more stable than the rankings.

# Competing interests

The author is an employee and shareholder of Quantum General
Intelligence, Inc., and has a commercial interest in the QAG
engine and in Q-Prime.

# References

1. [@lewis2020rag] Lewis, P. et al. (2020). *Retrieval-Augmented
   Generation for Knowledge-Intensive NLP Tasks*. NeurIPS.
2. [@karpukhin2020dpr] Karpukhin, V. et al. (2020). *Dense Passage
   Retrieval for Open-Domain Question Answering*. EMNLP.
3. [@robertson2009bm25] Robertson, S., Zaragoza, H. (2009). *The
   Probabilistic Relevance Framework: BM25 and Beyond*. FTIR.
4. [@gao2024ragsurvey] Gao, Y. et al. (2024). *RAG for LLMs: A
   Survey*. arXiv:2312.10997.
5. [@asai2024selfrag] Asai, A. et al. (2024). *Self-RAG*. ICLR.
6. [@yan2024crag] Yan, S.-Q. et al. (2024). *Corrective RAG*.
   arXiv:2401.15884.
7. [@jiang2023flare] Jiang, Z. et al. (2023). *Active Retrieval
   Augmented Generation* (FLARE). EMNLP.
8. [@zhang2024raft] Zhang, T. et al. (2024). *RAFT: Adapting
   Language Model to Domain Specific RAG*.
9. [@gao2023hyde] Gao, L. et al. (2023). *HyDE*. ACL.
10. [@sarthi2024raptor] Sarthi, P. et al. (2024). *RAPTOR*. ICLR.
11. [@zhou2024metarag] Zhou, Y. et al. (2024). *MetaRAG*.
12. [@es2024ragas] Es, S. et al. (2024). *RAGAS*. EACL demo.
13. [@saad2023ares] Saad-Falcon, J. et al. (2023). *ARES*.
14. [@vrandecic2014wikidata] Vrandečić, D., Krötzsch, M. (2014).
    *Wikidata: A Free Collaborative Knowledgebase*. CACM.
15. [@edge2024graphrag] Edge, D. et al. (2024). *GraphRAG*.
    arXiv:2404.16130.
16. [@guo2024lightrag] Guo, Z. et al. (2024). *LightRAG*.
    arXiv:2410.05779.
17. [@luo2024hypergraphrag] Luo, H. et al. (2025). *HyperGraphRAG*.
    arXiv:2503.21322.
18. [@pan2024kgllm] Pan, J. Z. et al. (2024). *Unifying LLMs and
    KGs*. IEEE TKDE.
19. [@packer2023memgpt] Packer, C. et al. (2023). *MemGPT*.
    arXiv:2310.08560.
20. [@rasmussen2024zep] Rasmussen, P. et al. (2025). *Zep*.
    arXiv:2501.13956.
21. [@mem0] mem0 contributors (2024). *mem0*.
22. [@park2023generative] Park, J. S. et al. (2023). *Generative
    Agents*. UIST.
23. [@wang2023voyager] Wang, G. et al. (2023). *Voyager*.
    arXiv:2305.16291.
24. [@gemini15] Gemini Team, Google (2024). *Gemini 1.5*.
25. [@peng2024yarn] Peng, B. et al. (2024). *YaRN*. ICLR.
26. [@liu2024ring] Liu, H. et al. (2024). *Ring Attention*. ICLR.
27. [@xiao2023streaming] Xiao, G. et al. (2023). *Efficient
    Streaming Language Models with Attention Sinks*.
28. [@liu2024lostmiddle] Liu, N. F. et al. (2024). *Lost in the
    Middle*. TACL.
29. [@reimers2019sbert] Reimers, N., Gurevych, I. (2019).
    *Sentence-BERT*. EMNLP.
30. [@xiao2024cpack] Xiao, S. et al. (2024). *C-Pack* (BGE family).
    SIGIR.
31. [@wang2022e5] Wang, L. et al. (2022). *E5*. arXiv:2212.03533.
32. [@li2023gte] Li, Z. et al. (2023). *GTE*. arXiv:2308.03281.
33. [@muennighoff2023mteb] Muennighoff, N. et al. (2023). *MTEB*.
    EACL.
34. [@thakur2021beir] Thakur, N. et al. (2021). *BEIR*. NeurIPS
    Datasets.
35. [@kassner2020negation] Kassner, N., Schütze, H. (2020).
    *Negated and Misprimed Probes for Pretrained Language
    Models*. ACL.
36. [@hossain2022negation] Hossain, Md. M. et al. (2022). *An
    Analysis of Negation in Natural Language Understanding
    Corpora*.
37. [@chalkidis2020legalbert] Chalkidis, I. et al. (2020).
    *LEGAL-BERT*. Findings of EMNLP.
38. [@beltagy2019scibert] Beltagy, I., Lo, K., Cohan, A. (2019).
    *SciBERT*. EMNLP.
39. [@lee2020biobert] Lee, J. et al. (2020). *BioBERT*.
    Bioinformatics.
40. [@coecke2010discocat] Coecke, B., Sadrzadeh, M., Clark, S.
    (2010). *Mathematical Foundations for a Compositional
    Distributional Model of Meaning*.
41. [@lambek2008] Lambek, J. (2008). *From Word to Sentence*.
42. [@coecke2013] Coecke, B. (2013). *An alternative Gospel of
    structure: order, composition, processes*.
43. [@coecke2017pic] Coecke, B., Kissinger, A. (2017). *Picturing
    Quantum Processes*. CUP.
44. [@kartsaklis2021lambeq] Kartsaklis, D. et al. (2021). *lambeq*.
    arXiv:2110.04236.
45. [@meichanetzidis2020qnlp] Meichanetzidis, K. et al. (2020).
    *Grammar-Aware Question-Answering on Quantum Computers*.
46. [@aerts2014quantum] Aerts, D., Sozzo, S. (2014). *Quantum
    Structure in Cognition*.
47. [@bruza2015qcognition] Bruza, P. D. et al. (2015). *Quantum
    Cognition*. Trends in Cognitive Sciences.
48. [@qgi-tr-01] Sammane, S. (2026). *QAG: A Reasoning-First
    Memory Infrastructure*. QGI-TR-2026-01.
49. [@qgi-tr-02] Sammane, S. (2026). *Purpose-Built Embedding
    Models for Rule-Bearing Text*. QGI-TR-2026-02.
50. [@qgi-tr-03] Sammane, S. (2026). *Conflict-Aware Memory for
    AI Agents*. QGI-TR-2026-03.
51. [@qgi-tr-04] Sammane, S. (2026). *A Born-Rule Classifier*.
    QGI-TR-2026-04.
52. [@qgi-tr-05] Sammane, S. (2026). *Quantum HyperGraph (QHG)*.
    QGI-TR-2026-05.

# Cite this as

```bibtex
@techreport{sammane2026beyondrag,
  author      = {Sam Sammane},
  title       = {Beyond Retrieval-Augmented Generation: A Review of Reasoning-First Alternatives for Rule-Bearing Content},
  institution = {Quantum General Intelligence, Inc.},
  number      = {QGI-TR-2026-06},
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
