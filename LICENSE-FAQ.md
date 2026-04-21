# Q-Prime License — Plain-English FAQ

This FAQ is a summary of the **QGI Commercial Model License v1.0** (see [`LICENSE.md`](./LICENSE.md)). It is informational only. If this FAQ and `LICENSE.md` conflict, `LICENSE.md` controls. This FAQ is not legal advice.

---

## The one-sentence version

**Q-Prime is available as a managed API. Evaluation access is free for 90 days, free for academic research, and free for non-commercial experimentation. Any production use, commercial use, hosted redistribution, or derivative model requires a paid commercial license — `contact@qgi.dev`.**

---

## How do I access Q-Prime?

Q-Prime is not distributed as a downloadable model. Access is via API.

| Path | What you get | Who it's for |
|---|---|---|
| **Evaluation API key** (`contact@qgi.dev`) | 90-day non-production access | Researchers, engineers evaluating fit |
| **OpenRouter listing** | Public API, pay-per-call | Developers building products (requires commercial license once in production) |
| **Enterprise endpoint** (`contact@qgi.dev`) | Dedicated endpoint, SLA, data-handling agreement, Qualtron pilot eligibility | Regulated-industry companies in production |

There is no `pip install qgi-q-prime`. Q-Prime is not distributed as a downloadable artifact.

---

## Who is this license for?

| If you are... | What you can do for free | What you need a paid license for |
|---|---|---|
| **A developer evaluating the API** | Call the evaluation API on your own data for 90 days | Any production traffic, any revenue-bearing use |
| **An academic researcher** | Use it in papers, cite it, publish benchmark comparisons | Nothing further, for truly non-commercial research |
| **A hobbyist** | Non-commercial experimentation via the evaluation API | Anything that earns revenue |
| **A startup** | Evaluate for 90 days on non-production traffic | Any production traffic, even free / pre-revenue MVPs |
| **A mid-market or enterprise company** | Evaluate in a sandbox for 90 days | Any production use, even internal |
| **A cloud / API provider** | Nothing by default | Any hosted redistribution — talk to us first |
| **Another AI vendor** | Nothing — you may not train competing models (see §5.3) | A specially negotiated license, if applicable |

---

## The short list of "can I?"

**Yes (free, under the Evaluation Grant)**

- Request an evaluation API key and call Q-Prime for 90 days on non-production data
- Measure its performance on your own data
- Compare it against other models and publish honest benchmark results (we encourage this)
- Cite it in academic work
- Teach it in a classroom
- Use the evaluation API in a personal non-commercial project

**No (without a paid commercial license)**

- Put it into production — any production system, even internal
- Offer a hosted API, SaaS product, or managed service that routes through it
- Redistribute any Q-Prime material, including API outputs used as training targets
- Train a student model by distilling Q-Prime's outputs
- Train a model that competes with Q-Prime, QAG, Neural Symbolic Agents, or any Qualtron model
- Release a derivative publicly or under a more permissive license
- Use it in fully automated high-stakes decisions about people (credit, housing, employment, healthcare access, liberty) without a negotiated license and qualified human review
- Use it in embargoed jurisdictions or by sanctioned parties

**Needs a conversation (`contact@qgi.dev`)**

- Any use you're about to deploy to customers
- Any revenue-bearing integration, even a small one
- Any regulated-industry use (mortgage, banking, healthcare, regulated news)
- Any use that embeds Q-Prime's outputs as part of another model or product
- Any channel / reseller scenario
- Self-hosted or on-premises deployment

---

## Frequently asked specifics

### "How do I try Q-Prime as a developer?"

Email `contact@qgi.dev` with a short description of your evaluation goals. We issue an evaluation API key. The Evaluation Grant runs for 90 days from issuance. If you decide to keep using Q-Prime after that, or start reaching real users, you need a commercial license.

### "Is there a pip package I can install?"

No. Q-Prime is not available as a downloadable model. You call it over HTTPS using the evaluation API key (OpenAI-compatible request format). Integration examples are provided with the key.

### "I run a startup. We're pre-revenue. Can we use it in our MVP?"

An MVP that reaches users is a production use, even if it's free and pre-revenue. You need a commercial license. We have startup-friendly terms — tell us your stage and use case at `contact@qgi.dev` and we'll size a license appropriately.

### "I'm an academic researcher. Can I use this in a paper?"

Yes. Academic, non-commercial research is explicitly permitted. You must cite Q-Prime per `LICENSE.md` §8. You may publish benchmark comparisons, replication studies, and critical analyses freely.

### "Can I publish a benchmark showing Q-Prime is worse than model X on task Y?"

Yes. We encourage honest benchmarking. The license does not restrict benchmark publication. We ask only that you use the documented evaluation configuration and disclose that configuration in your paper.

### "Can I fine-tune Q-Prime on my own data?"

Fine-tuning is not available under the Evaluation Grant. For enterprise customers whose use case requires custom adaptation, domain tuning is available under a separate commercial arrangement — `contact@qgi.dev`.

### "Can I use Q-Prime's outputs to train my own embedding model?"

No. §5.3 prohibits using Q-Prime outputs to train a competing model. §5.4 prohibits using outputs as training targets for any publicly released or more-permissively-licensed model. If you have an adjacent use case — for example, training a downstream classifier that consumes Q-Prime outputs but does not attempt to reproduce them — contact us and we'll clarify.

### "Can I reverse-engineer what's inside the model?"

No. §5.5 prohibits reverse-engineering trade secrets. Architectural details are confidential until QGI publishes the accompanying paper. Probing the API to infer internals is a material breach of §5.5.

### "Can I self-host Q-Prime on my own servers?"

Self-hosting is only available under specific enterprise commercial licenses that authorize on-premises deployment. By default, Q-Prime is delivered as a managed API.

### "We're a cloud provider. Can we host Q-Prime for our customers?"

Not without a negotiated commercial license that authorizes hosted redistribution. Talk to us — `contact@qgi.dev`.

### "Is my data safe?"

When you call Q-Prime via OpenRouter, OpenRouter's privacy and data-handling terms apply. When you call the QGI evaluation or enterprise endpoints, QGI's data-handling terms apply. QGI does not train on customer inputs and does not retain them beyond what is necessary to answer the call. Enterprise customers receive a dedicated data-processing addendum as part of the commercial license.

### "What counts as 'production'?"

Anything that: reaches external users, runs continuously without a sunset date, is part of a customer deliverable, supports a decision that affects a real-world outcome, or generates revenue. If you're unsure whether your deployment counts as production, assume it does and ask us.

### "What happens if I accidentally violate the license?"

Contact `contact@qgi.dev` as soon as you notice. Most cases get resolved by retroactively bringing the deployment under a commercial license with reasonable pricing. What we will not tolerate is redistribution of Q-Prime material, use of outputs to train competing models, and any of the §5.3 / §5.4 violations — those are material breaches and will terminate the license.

### "Can I use Q-Prime in the EU?"

Yes, subject to §5.7 (export controls) and §5.8 (no high-risk AI Act use without a qualified license). Standard European use cases are fine; EU AI Act "high-risk" uses require a qualified commercial license with explicit allocation of oversight responsibilities.

### "What's the deal with trademarks?"

"Q-Prime", "QGI", "QAG", "Quantum-Augmented Generation", "Neural Symbolic Agents", and "Qualtron" are trademarks of Quantum General Intelligence, Inc. You may use them to factually identify the service ("we use QGI's Q-Prime embedding API"). You may not use them in a way that suggests endorsement, or as part of your own product name.

### "Does QGI patent this?"

QGI retains all patent rights related to Q-Prime, including any that may be issued or pending. This license does not grant any patent license except what is necessary for authorized Use. A commercial license may include explicit patent-license terms depending on the customer's needs — ask us.

### "What if QGI sues me?"

The license has a defensive-patent clause (§11.3): if you sue QGI for IP infringement first, your license terminates automatically. We don't plan on suing customers acting in good faith. For everyone else, the license, Delaware governing law, and New Castle County venue apply.

---

## How to get a commercial license

1. Email `contact@qgi.dev` with your use case, expected scale, and target go-live date.
2. We'll schedule a 30-minute scoping call.
3. We issue an Order Form with scope, fees, term, support, and SLA.
4. Signed Order Form + signed QGI Commercial Model License Schedule = Permitted Commercial License.

Typical commercial tiers (subject to the Order Form — contact us for current pricing):

| Tier | For whom | What you get |
|---|---|---|
| **Startup** | Pre-Series B, < $5M ARR | Production API rights, community support, discounted pricing |
| **Growth** | < $50M ARR | Production API rights, email support, standard SLA |
| **Enterprise** | $50M+ ARR or regulated industry | Production API rights, dedicated endpoint, dedicated support, SLA, audit cooperation, Qualtron pilot eligibility |
| **OEM / Channel** | Bundling Q-Prime into another product or service | Negotiated scope, royalty model, co-marketing |

Self-hosted and on-premises deployment are available under Enterprise and OEM / Channel tiers, subject to scope review.

---

## What "strong IP" means in this license — in plain terms

The license is deliberately strong on IP because Q-Prime's value is in the training work behind it, not in any single artifact. Five guardrails matter most:

1. **No distillation.** You cannot use Q-Prime's outputs to train a student model and release it, commercially or otherwise. This is the one we're most strict about.
2. **No competing-model training.** You cannot use Q-Prime — or its outputs — to train something that competes with us.
3. **No redistribution of Q-Prime material.** Q-Prime is an API service. Reselling, wrapping, or hosting that service requires a commercial license that explicitly authorizes it.
4. **No reverse engineering of trade secrets.** Until the accompanying paper is published, architectural specifics are confidential. Probing the API to infer internals is a breach.
5. **Defensive patent.** If you attack our IP, your license goes away.

Everything else — evaluating, benchmarking, academic use, critical analysis — is intentionally frictionless. We want developers and researchers to try Q-Prime. We need commercial customers to pay for the work.

---

## Last word

If you're about to deploy and you're not sure whether you need a license, the answer is almost always yes — and the earliest conversation is the cheapest one. `contact@qgi.dev`.
