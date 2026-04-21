---
title: "The $0.000 F1 Problem: Why Your AI Copilot Can't Read a Regulation"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "LinkedIn Pulse"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~800
---

Every CIO I meet has the same story. The internal AI copilot works beautifully on the demo. It answers "what is our remote-work policy?" with a confident paragraph. Then someone from Compliance types in "which of our policies conflict with the latest MiFID II amendment?" and the answer goes sideways — fluent, citeable, and silently wrong.

There is a reason this keeps happening, and it is not the language model.

**The embedding model underneath your copilot cannot tell "must" from "must not".**

That is not an exaggeration. In QGI's regulatory-conflict benchmark, classical cosine similarity across five widely used embedding models from four organisations (OpenAI, Cohere, and two leading open-source families) scores an **F1 of 0.000** on the task of detecting rule-level contradictions. Not 0.300. Not 0.500. **Zero.** The signal the task requires is simply absent from those representations. Swapping one for another doesn't help. Using a larger one doesn't help. The primitive itself — similarity without sign — cannot distinguish agreement from contradiction.

Today my team at Quantum General Intelligence is releasing the technical preprint of our replacement for that primitive: **Quantum-Augmented Generation (QAG)**. And on the same benchmark, QAG driven by our purpose-built embedding model, **Q-Prime**, scores an **F1 of 1.000**.

The gap is categorical, not incremental.

---

**What is QAG?** It is a reasoning-first knowledge and memory infrastructure. Four architectural choices define it:

1. **A deterministic parser (QNR2)** — ingestion is algorithmic, not LLM-based. Same input, same output, every time.
2. **An immutable, versioned hypergraph (QHG)** — rules live as nodes; *conflict* and *dependency* are first-class edge types. Past states are always recoverable.
3. **A purpose-built embedding model (Q-Prime)** — preserves polarity, quantifier scope, obligation strength, and cross-rule dependency as separable directions.
4. **Real quantum formalism on classical GPUs** — Hilbert spaces, the Born rule, superposition, signed interference. No QPU required; only the mathematics.

On top sits an intelligence layer — Hilbert Space Compacting — that publishes seven named signals: **Relevance, Conflict, Overlap, Redundancy, Coverage, Coherence, Topology**. Each is a typed answer, not a metric. Every answer traces to a specific rule in a specific document. The entire pipeline is replayable.

**QAG publishes signals, not decisions.** That distinction is load-bearing. Whether a loan should be originated, a trade is compliant, a prior authorisation is granted — that is the call of a qualified human reviewer. QAG's job is to give that reviewer the signals that make the call defensible.

---

**What it means for your procurement process.** I have six questions I would put in every AI RFP today:

1. Does the system detect contradictions between retrieved passages?
2. Does similarity distinguish "must" from "must not", "all" from "some"?
3. Is the parser deterministic?
4. Is the state immutable and versioned?
5. Is every answer traced to a specific source?
6. Is the pipeline replayable three months from now?

A vendor that cannot answer yes to all six should not be selected for rule-bearing content.

---

**Beyond compliance.** The same primitives address problems no current system solves in AI-agent memory: the consistency of long-running memory, the curation of what enters a context window, the coordination of multi-agent output, the triage of conflicting sources in research agents. The engine you license for compliance becomes the memory layer for your copilots.

---

The preprint is QGI-TR-2026-01. The companion preprints on Q-Prime, agent memory, the Born-rule classifier, QHG, and a landscape review of the entire space are all out this week.

If you run regulated AI, or you run long-running agents, I'd like to hear your counterarguments. Email me: `sam@qgi.dev`. For evaluation tokens: `contact@qgi.dev`. Q-Prime is in progressive beta today; QAG engine GA is **21 June 2026**.
