---
title: |
  Quantum-Augmented Generation
subtitle: |
  A Reasoning-First Alternative to RAG for the Regulated Enterprise
  --- Executive Whitepaper
author:
  - name: Sam Sammane
    email: sam@qgi.dev
    affiliation: Chief Technology Officer and Founder, Quantum General Intelligence, Inc.
date: "21 April 2026"
version: "Executive Whitepaper v1.0"
abstract: |
  AI copilots and agents now run on content the enterprise cannot afford
  to get wrong --- regulations, policies, contracts, case law, clinical
  guidelines. The dominant architecture, Retrieval-Augmented Generation
  (RAG), was built for the open web and fails these documents
  systematically: similar-but-contradictory clauses are retrieved as
  near-duplicates, the generator produces a confident-and-wrong answer,
  the audit trail is a liability. The gap is categorical, not
  incremental. On QGI's regulatory-conflict benchmark, classical
  similarity across five widely used embedding models from four
  organisations scores an F1 of **0.000**; the QGI engine, driven by
  a purpose-built embedding (**Q-Prime**), scores **1.000**.

  This whitepaper is the CIO and procurement version of QGI's technical
  paper series. It explains, without mathematics, what
  **Quantum-Augmented Generation (QAG)** is, what it costs today to
  run without it, how it sits alongside existing AI investments, and
  what evaluation criteria your team should use when selecting an
  AI vendor for rule-bearing content. A six-point evaluation checklist
  is provided for use in RFPs.

keywords:
  - executive whitepaper
  - Quantum-Augmented Generation
  - QAG
  - Q-Prime
  - regulated AI
  - compliance
  - contract intelligence
  - policy management
  - due diligence
  - agent memory

header-includes:
  - \AtBeginDocument{\hypersetup{pdftitle={QAG Executive Whitepaper},pdfauthor={Sam Sammane --- Quantum General Intelligence},pdfsubject={A reasoning-first alternative to RAG for the regulated enterprise},pdfkeywords={QAG, Q-Prime, regulated AI, compliance, contract intelligence, policy management, due diligence, agent memory},pdfcreator={pandoc + xelatex},pdfproducer={QGI typesetting pipeline},bookmarksopen=true,bookmarksnumbered=true}}
---

> **Executive Whitepaper --- v1.0, 21 April 2026.**
> Authored by Dr. Sam Sammane, CTO and Founder, Quantum General
> Intelligence, Inc. (`sam@qgi.dev`). Companion to the QAG technical
> paper [QGI-TR-2026-01]. Intended audience: CIO, CRO, Chief
> Compliance Officer, VP Engineering, Head of AI Governance.

# The cost of getting rules wrong

A regulated enterprise runs on rules. Regulations, internal policies,
supplier contracts, clinical guidelines, trading limits, franchise
agreements, collective-bargaining agreements --- the list is long
and each entry encodes structure that a human counsel can read and
that a standard AI system cannot. That structure includes triggers
("when X happens"), conditions ("provided that Y"), obligations
("the institution shall report within 30 days"), exceptions ("except
where Z applies"), and cross-references ("see §4.2(b)").

What happens today when an enterprise asks a standard AI copilot a
question about its own policies? The copilot retrieves the passages
in its index most similar to the question, concatenates them into a
prompt, and asks a language model to summarise. If two of the
retrieved passages are *similar but contradictory* --- for example,
"the institution must disclose" and "the institution must not
disclose" --- the copilot cannot tell. It produces a fluent, confident
answer that references both. The language model downstream has no
way to know that the underlying evidence contradicts itself.

The business consequences are well known to anyone who has run
compliance on the receiving end of an AI-generated answer:

- **Wrong answers in the audit report.** A regulator asks about
  the firm's policy; the copilot cites a superseded paragraph
  alongside the current one; the answer is produced under a plausible
  but incorrect synthesis.
- **Silent policy drift.** New regulations arrive quarterly; the
  copilot never flags the contradictions between new regulation and
  existing policy, so contradictions accumulate until a human notices
  them.
- **Contract inconsistency at scale.** Vendor contracts signed under
  different legal teams over five years contain conflicting
  termination clauses; the standard copilot treats them as related
  but does not surface the conflict.
- **Unusable provenance.** A regulator or internal auditor asks
  "why did the AI answer this way?". The copilot can return the
  chunks it retrieved, but cannot explain why those chunks rather
  than others, nor reproduce the answer next quarter if the
  embedding model has been updated.

These are not edge cases. They are the default failure mode of the
dominant AI architecture applied to the work the regulated enterprise
actually does.

# What's broken: similarity-based retrieval

The failure mode has a single root cause. Every standard AI copilot
relies on *similarity-based retrieval*: pieces of text are converted
into vectors by an embedding model, and the system returns the
passages whose vectors are closest to the question's vector. The
approach is beautifully simple. It has one property that disqualifies
it for rule-bearing content: similarity is indifferent to
*contradiction*.

Two passages that say the opposite of each other --- "must disclose"
and "must not disclose" --- are close to each other in vector space.
So are two paraphrases of the same rule. The system cannot
distinguish them. From the downstream generator's perspective, they
are interchangeable pieces of evidence.

This is not a tuning problem. Larger models do not make the
disagreement between "must" and "must not" larger. The issue is the
primitive itself, built into the embedding model's training
objective. Every widely deployed general-purpose embedding ---
OpenAI, Cohere, Anthropic, BGE, E5, GTE --- shares the primitive.
Swapping one for another does not resolve the failure. A longer
context window does not resolve it either; longer contexts contain
more contradictions, not fewer.

The correct response is to change the primitive.

# What QAG does differently

**Quantum-Augmented Generation (QAG)** is QGI's replacement for the
retrieve-then-stuff primitive. It is built around four architectural
choices that give it properties standard copilots cannot match. In
plain language:

**Choice 1 --- Understand structure, not just text.** QAG uses a
deterministic, algorithmic parser (**QNR2**) to read each rule and
extract its structure: trigger, condition, action, exception,
obligation strength, scope, cross-references. The parser produces
the same output every time and records why it produced what it did.

**Choice 2 --- Store rules as a graph, not a list.** Parsed rules
live in a **Quantum HyperGraph (QHG)**, where each connection can
link any number of rules at once. That is what real compliance looks
like: a single obligation typically joins three or four rules
simultaneously. The graph is immutable --- past states can always be
recovered --- and changes produce new versions, not overwrites.

**Choice 3 --- Measure agreement and disagreement, not just
similarity.** QAG uses a purpose-built embedding model
(**Q-Prime**) that preserves *polarity* (the difference between
"must" and "must not"), *scope* (the difference between "all" and
"some"), and *obligation strength* (the difference between "must"
and "may"). With these preserved, QAG can compute an *interference
signal* between any two rules that is positive when they agree,
negative when they disagree, and small when they are unrelated. This
is the single property every standard copilot lacks.

**Choice 4 --- Answer, then prove the answer.** Every QAG answer
cites a specific rule in a specific section of a specific document.
Claims that cannot be grounded are not produced. The entire pipeline
from question to answer is recorded in a trace that an auditor can
step through.

The headline result on QGI's regulatory-conflict benchmark: classical
similarity across five widely used embedding models scores an F1 of
**0.000**; the QAG engine driven by Q-Prime scores **1.000**. The
gap is categorical.

# What QAG publishes vs. what QAG does not

A single message to put in every procurement conversation: **QAG
publishes signals, not decisions.**

QAG tells your compliance, contracts, or operations team:

- Which rules apply to this question. (*Relevance*)
- Where rules contradict each other. (*Conflict*)
- Where rules overlap without contradicting. (*Overlap*)
- Which rules are duplicates or near-duplicates. (*Redundancy*)
- Where the rule set has gaps. (*Coverage*)
- Whether the rule set is internally consistent. (*Coherence*)
- What the dependency topology looks like. (*Topology*)

QAG does *not* decide:

- Whether a loan should be originated.
- Whether a trade is compliant.
- Whether a patient's prior authorisation is granted.
- Whether a news item may be published.
- Whether an employee should be hired or fired.

Those decisions belong to qualified humans, supported by QAG's
signals and audit trail. The separation is a design choice: QAG is
built to be the AI component of a pipeline that includes certified
human review. It is not a substitute for that review.

# Five enterprise applications on one engine

The same QAG pipeline supports five enterprise applications; they are
not five separate products.

**1. Regulatory compliance.** Ingest thousands of regulations and
internal policies. Conflict signals surface contradictions between
company policy and external regulation; coverage signals reveal the
gaps. Every finding is audit-trail-ready. Typical domains: SOX,
HIPAA, FERC, Basel III, MiFID II.

**2. Contract intelligence.** Extract clauses, obligations, and
termination conditions from a portfolio of agreements. Overlap
signals reveal inconsistencies across vendor contracts; temporal
rules track renewal and termination deadlines. Reduces hours-to-answer
from days to minutes on typical procurement questions.

**3. Policy management.** Detect redundant, conflicting, or outdated
internal rules at scale. In reference deployments, redundancy
recommendations reduce policy bloat by **40--60 %** on first pass ---
without changing the underlying authority of any rule, only removing
duplicates and flagging contradictions for human review.

**4. Due diligence.** Ingest entire data rooms in hours rather than
weeks. Topology analysis reveals hidden dependencies and risk
clusters across hundreds of documents. The structure a human reviewer
would find after a month of reading, surfaced in the first hours.

**5. AI agent grounding.** Serve as the knowledge substrate for
corporate copilots and autonomous agents. Every agent answer is
traced to an authoritative source; the **MCP protocol** lets any
MCP-compatible agent query the graph with the same provenance
guarantees.

All five sit on the same engine. A QAG deployment that starts in
regulatory compliance expands into contracts, policy, due diligence,
and agent grounding without a second procurement cycle.

# Trust architecture: the five properties regulators ask for

Every regulator, internal auditor, and model-governance leader we
have worked with asks for five things, usually in the following
order. Most AI systems satisfy one or two. QAG satisfies all five
by construction:

1. **Deterministic parsing.** The ingestion pipeline produces the
   same structured output every time from the same input. No
   temperature, no sampling, no silent version drift.
2. **Immutable state.** Once the rule graph is built, it is not
   mutated. Updates produce a new version. The rule set in force on
   any past date can always be recovered.
3. **Advisory-only intelligence.** The AI layer informs; the human
   layer decides. No AI output silently modifies the rule base.
4. **Source-grounded answers.** Every claim traces to a specific
   rule in a specific document. Hallucination is prevented by
   construction, not filtered after the fact.
5. **Replayable audit trace.** Every step of the pipeline ---
   question, intent, candidate rules, signals, answer, grounding,
   confidence --- is recorded and reproducible.

An AI system that lacks any of the five is, by the standards of every
major regulator we have read, unfit for use in a controlled decision.

# Beyond compliance: the same primitives solve agent memory

The QAG engine's primitives --- conflict detection, signed
interference, audit-replayable trace --- apply beyond compliance. A
long-running AI agent that serves customers, runs a sales motion, or
operates a factory accumulates rules, preferences, and observations
over time. The same "similar is not the same as consistent" failure
that breaks compliance retrieval breaks agent memory: the agent
retrieves yesterday's preference together with today's contradicting
preference, and picks one of them by attention bias.

QAG-backed agent memory applies the interference signal to memory
items directly. When two memory items disagree, the disagreement is
surfaced as an event the agent (or its operator) must resolve. The
companion preprint [QGI-TR-2026-03] develops this use case in
detail. The relevant message for the enterprise is that the same
engine you license for compliance becomes the memory layer for your
copilots. You do not pay twice; you deploy once.

# Six evaluation criteria for AI procurement

Use the six questions below in any RFP or vendor-evaluation process
for AI systems that touch rule-bearing content. QAG passes all six;
most current systems pass one or two.

1. **Does the system detect contradictions between pieces of
   retrieved content?** Ask for a demonstration on your own
   corpus, not on a vendor's benchmark. Ask specifically for the
   **conflict F1**.
2. **Does the system distinguish "must" from "must not", and "all"
   from "some", in its similarity scores?** Ask for a test with
   a pair that differs only by a negation and a pair that differs
   only by a quantifier.
3. **Is the parser deterministic?** Ask whether the ingestion
   pipeline uses a large language model. If it does, accept that
   your graph state is non-deterministic.
4. **Is the state immutable and versioned?** Ask whether past rule
   states can be recovered without log replay.
5. **Is every answer traceable to a specific source?** Ask for a
   demonstration with a synthetic document in which the correct
   answer appears only in a specific paragraph.
6. **Is the entire pipeline replayable?** Ask whether the vendor
   can, three months from now, produce the exact same answer to a
   current question given the same inputs.

A vendor that cannot answer "yes" to all six should not be selected
for production use against rule-bearing content.

# Deployment options

QAG is delivered as a managed engine. Three access paths are
available today:

**Evaluation access.** 90-day evaluation under QGI Commercial Model
License v1.0, no charge. Token request: `contact@qgi.dev`.

**Enterprise.** Production SLA, dedicated endpoints, audit support,
indemnification options. Tiers: Startup, Growth, Enterprise, OEM /
Channel. Inquiry: `contact@qgi.dev`.

**OpenRouter.** Q-Prime is listed on OpenRouter as part of the QAG
public beta for teams that want to test a narrow slice of the
functionality before engaging procurement.

General availability for the full QAG engine is 21 June 2026.

# About Quantum General Intelligence

Quantum General Intelligence (QGI) is a research and product company
building the infrastructure for AI systems that enterprises can
audit. The founding team combines backgrounds in quantum
information, formal verification, and applied ML. QGI's products are:

- **Q-Prime** --- the purpose-built embedding model underneath the
  QAG engine; public beta available now.
- **QAG engine** --- the reasoning-first knowledge infrastructure
  described in this paper; GA 21 June 2026.
- **Neural Symbolic Agents** --- an enterprise agent runtime on top
  of QAG.
- **Qualtron** --- vertical models for mortgage, banking, healthcare,
  and regulated news.

QGI is headquartered at `qgi.dev`. Commercial inquiries:
`contact@qgi.dev`. Research: `research@qgi.dev`. Press:
`press@qgi.dev`.

# Notes and disclosures

This whitepaper is a vendor publication. The author is an employee
and shareholder of Quantum General Intelligence, Inc., and has a
commercial interest in Q-Prime and the QAG engine. Numerical claims
in this paper are drawn from QGI's internal regulatory-conflict
benchmark and are released under evaluation agreement; full
methodology is forthcoming in a companion evaluation paper.

---

© 2025--2026 Quantum General Intelligence, Inc. All rights reserved.
Trademarks: Q-Prime, QAG, QHG, Quantum-Augmented Generation, QGI,
Neural Symbolic Agents, Qualtron.
