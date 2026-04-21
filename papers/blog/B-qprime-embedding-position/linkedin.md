---
title: "Your Embedding Model Cannot Read a Contract. Here's the Proof."
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "LinkedIn Pulse"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~700
---

Ten lines of Python will prove this to you.

Take any general-purpose embedding model — OpenAI, Cohere, Voyage, Google's Gemini embedding, an open-source BGE, E5, GTE. Embed two sentences:

1. *"The broker must disclose material conflicts of interest to the client."*
2. *"The broker must **not** disclose material conflicts of interest to the client."*

Compute the cosine similarity.

In every production embedding model on the market in 2026, that number is above 0.95. In several, it's above 0.98. Both sentences land in the same neighbourhood in vector space. A retrieval system will treat them as near-duplicates.

Now think about what that means for every enterprise application that runs on top of such a model.

- A legal AI reviewing a contract retrieves, for the query "what are the disclosure obligations?", both sentences as near-ties. The downstream LLM hallucinates a compromise that neither sentence actually says.
- A compliance engine verifying a procedure cites a prohibition as if it were a requirement, because the embedding placed them 0.02 apart.
- An AI agent's memory store retrieves, for the query "user's dietary preference", both "prefers Thai" and "avoiding Thai" — because the two sentences are semantically adjacent. The agent picks one by attention bias.

This is not an edge case. This is the single largest failure mode of embedding-driven retrieval in 2026.

## Why scaling will not fix it

The training objective of general-purpose embedding models rewards topical similarity. "Discloses" and "does not disclose" share every topical word. Absent an explicit polarity supervisory signal, contrastive training will push them together.

Making the model bigger makes the space bigger. The *ratio* of topic-change distance to polarity-flip distance does not improve. I've seen this tested across six embedding families; the number stays fixed.

Fine-tuning helps partially on seen domains. It does not generalise. It also does not address five other properties a rule-bearing embedding must preserve: **quantifier scope, structured conditions, obligation strength, cross-rule dependency, and signed interference**.

## What the fix looks like

QGI built **Q-Prime**, a 1,536-dimensional embedding model where polarity, scope, obligation strength, and cross-rule dependency are trained as *separable directions* with dedicated supervision.

On QGI's regulatory-conflict benchmark, the gap between a general-purpose embedding + cosine similarity and Q-Prime + signed interference is:

- **F1 = 0.000** vs **F1 = 1.000**.

That is not an incremental improvement. The task is *not expressible* in a general-purpose embedding space, and is *fully expressible* in Q-Prime's.

This result sits inside QGI's larger reasoning-first architecture — QAG (Quantum-Augmented Generation) — which is described in a companion preprint released today.

## What this means for your AI procurement

If you are evaluating an AI platform for regulated, contractual, clinical, or policy-bearing content, stop asking about BLEU or BERTScore. Ask this instead:

*"For any rule and its logical negation, what is your pipeline's output? Is the contradiction detected as a first-class event, or does it require a downstream LLM to infer it?"*

If the vendor's answer involves a chain-of-thought LLM judging contradiction after retrieval, ask about their evaluation on **paired rule / negation** corpora. You will usually hear silence.

## Access

- Preprint: *Embedding Rule-Bearing Text: The Case for a Purpose-Built Model* — [arXiv URL].
- Companion preprint on the full QAG engine: [QGI-TR-2026-01].
- Q-Prime API via OpenRouter (public beta). Enterprise SLA available.
- QAG engine GA: **21 June 2026**.
- Evaluation token: `contact@qgi.dev` with a one-line description of your use case.

---

**Dr. Sam Sammane** is CTO and Founder of Quantum General Intelligence, Inc. (`sam@qgi.dev`). QGI is headquartered in Toronto with a research lab in Paris.
