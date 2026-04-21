---
license: other
license_name: qgi-commercial-model-license-v1
license_link: LICENSE.md
language:
  - en
tags:
  - quantum-augmented-generation
  - qag
  - compliance
  - regulated-ai
  - audit-trail
  - conflict-detection
  - regulated-news
  - compliance-ai
inference: false
---

# Q-Prime — QGI Quantum Embedding Model

**Q-Prime** is a quantum-structured embedding model purpose-built for regulated AI. It powers the **QAG engine** — Quantum-Augmented Generation — QGI's successor category to classical RAG for applications that cannot afford to hallucinate.

Q-Prime is accessed **as a managed API**. Weights are not distributed. See [How to access](#how-to-access) below.

> **License**: QGI Commercial Model License v1.0 — evaluation access available on request, paid commercial license required for production. See [`LICENSE.md`](./LICENSE.md) for full terms, [`LICENSE-FAQ.md`](./LICENSE-FAQ.md) for a plain-English summary. Licensing: `contact@qgi.dev`.

---

## What Q-Prime Does

Classical embedding models fail on one task that compliance, legal, audit, and regulated-news teams cannot live without: **telling two contradictory statements apart**.

Two statements that differ by a single negation — "must report" vs "must not report" — look nearly identical to cosine similarity. Retrieval systems built on cosine quietly merge contradictions. Generators downstream paper over the gap. Audits fail.

Q-Prime participates in a pipeline — the **QAG engine** — that produces an interference signal with polarity. The signal is signed: same-polarity related statements reinforce, opposite-polarity related statements cancel. The sign is the decision.

## Headline Result

On QGI's regulatory-conflict benchmark:

| Signal | F1 |
|---|---|
| Classical cosine similarity (across five widely used embedding models from four organizations) | **0.000** |
| QAG interference signal (Q-Prime + polarity) | **1.000** |

The interference effect replicates across embedding families. It is a property of the language of regulation, not of any single model. Q-Prime adds production-grade margin, latency, and throughput on top.

Full evaluation methodology and benchmark suite are released under evaluation agreement — `contact@qgi.dev`.

---

## Who Q-Prime Is For

| Audience | Use |
|---|---|
| **Regulated-industry engineering teams** | Embed rules, policies, contracts, and case documents where contradictions must be caught, not averaged |
| **Compliance and audit functions** | Continuous rule-to-rule conflict detection across versioned policy sets |
| **Regulated-news and research desks** | Synthesis of multiple sources where the sign of the claim matters |
| **Risk and model-governance leaders** | An embedding layer whose failure mode is explainable, not statistical |

Q-Prime is **not** intended for general-purpose web retrieval, low-stakes question answering, or non-English corpora.

---

## How to Access

Q-Prime is distributed exclusively as a managed API. There are three access paths.

### 1. Evaluation access (researchers, engineers, non-production)

Request an evaluation token at `contact@qgi.dev`. Evaluation is governed by `LICENSE.md` §3 (90-day grant, non-production only, academic use permitted).

### 2. OpenRouter (public API, pay-per-call)

Q-Prime will be listed on OpenRouter as part of the QAG engine progressive beta. Listing status and pricing are announced at [qgi.dev](https://qgi.dev).

### 3. Enterprise (production, SLA, audit, dedicated endpoints)

`contact@qgi.dev`. Tiers: Startup, Growth, Enterprise, OEM / Channel. See [`LICENSE-FAQ.md`](./LICENSE-FAQ.md) for tier overview.

**General availability of the full QAG engine is targeted for June 21, 2026.** Q-Prime is available to selected customers in progressive beta before that date.

---

## What You Do Not Get

To set expectations before first contact:

- **No weights download.** Q-Prime weights, adapters, and supporting parameters are not distributed. Redistribution requires a separately negotiated license.
- **No training recipe.** Data curation, training procedure, and internal evaluation methodology are confidential trade secrets (see `LICENSE.md` §2.2).
- **No architecture disclosure beyond the paper.** Architectural details are released in the accompanying paper under a separate schedule.

We are aware this is unusual for a model page on HuggingFace. Q-Prime is a commercial product, not an open research artifact. The card exists so developers and procurement teams can evaluate fit, not so the model can be cloned.

---

## The Broader Stack

Q-Prime is the model layer. The commercial stack sits on top.

| Layer | Product | Status |
|---|---|---|
| **Model** | Q-Prime (this card) | Progressive beta via API |
| **Engine** | QAG engine — extraction, interference, versioning, audit trail | General availability June 21, 2026 |
| **Agent platform** | Neural Symbolic Agents — enterprise agent runtime with QAG underneath | Enterprise evaluation via `contact@qgi.dev` |
| **Vertical models** | Qualtron — tuned for mortgage, banking, healthcare, regulated news | Enterprise pilots via `contact@qgi.dev` |

Q-Prime by itself does not extract rules, detect conflicts end-to-end, produce audit trails, or host agents. Those are the jobs of the QAG engine, the Neural Symbolic Agents platform, and the Qualtron vertical models.

---

## License

Q-Prime is licensed under the **QGI Commercial Model License v1.0** ([`LICENSE.md`](./LICENSE.md)).

- Evaluation is free for 90 days from grant of access, subject to the terms in `LICENSE.md` §3
- Academic research use is free
- Any production, commercial, or hosted use requires a Permitted Commercial License
- Redistribution of any Q-Prime material requires a Permitted Commercial License
- Training a competing model using Q-Prime or its outputs is prohibited under `LICENSE.md` §5.3

A plain-English walkthrough is in [`LICENSE-FAQ.md`](./LICENSE-FAQ.md).

---

## Responsible Use

Q-Prime may not be used to:

- Automate decisions that materially affect an individual's legal rights, employment, housing, credit, healthcare access, education, or liberty, except under a certified pipeline that includes qualified human review
- Circumvent legal or regulatory obligations in the user's jurisdiction
- Misrepresent the user's compliance posture to a regulator, counter-party, or auditor
- Train a model intended to compete with Q-Prime, QAG, Neural Symbolic Agents, or any Qualtron model

See `LICENSE.md` §5 for the binding language.

---

## Citation

Peer-reviewed publication accompanying Q-Prime is forthcoming. Until the paper is on a recognized preprint server or accepted venue, please cite as:

```bibtex
@misc{qgi2026qprime,
  title        = {Q-Prime: A Quantum-Structured Embedding Model for Regulated AI},
  author       = {{Quantum General Intelligence, Inc.}},
  year         = {2026},
  howpublished = {Model card, \url{https://huggingface.co/qgi/qgi-q-prime}},
  note         = {QAG Engine documentation}
}
```

Academic work using Q-Prime must cite per `LICENSE.md` §8.

---

## Contact

| Need | Where |
|---|---|
| Evaluation access, API keys, documentation | `contact@qgi.dev` |
| Commercial license, enterprise pilots, SLA, support | `contact@qgi.dev` |
| QAG engine waitlist (GA June 21, 2026) | [qgi.dev](https://qgi.dev) |
| Partnership (cloud providers, hyperscalers, channel) | `partner@qgi.dev` |
| Press and analyst relations | `press@qgi.dev` |
| Security disclosure | `security@qgi.dev` |

---

## Company

**Quantum General Intelligence, Inc.** — Delaware corporation, founded 2025.

Website: [qgi.dev](https://qgi.dev) · HuggingFace: [qgi/qgi-q-prime](https://huggingface.co/qgi/qgi-q-prime) · GitHub: [qgi/qgi-q-prime](https://github.com/qgi/qgi-q-prime)

© 2025–2026 Quantum General Intelligence, Inc. All rights reserved.
"Q-Prime", "QAG", "Quantum-Augmented Generation", "QGI", "Neural Symbolic Agents", and "Qualtron" are trademarks of Quantum General Intelligence, Inc.
