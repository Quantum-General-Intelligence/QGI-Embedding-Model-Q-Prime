---
title: "Twitter/X thread — Q-Prime: a purpose-built embedding for rule-bearing text"
byline: "@sam_sammane / @qgidev"
venue: "Twitter / X"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
---

# Thread (10 tweets)

---

**1/**
Your embedding model can't tell a rule from its negation.

Ten lines of code will prove it. Cosine similarity of "must disclose X" and "must NOT disclose X" in:

• OpenAI `text-embedding-3-large` → 0.96
• Cohere `embed-v3` → 0.97
• Voyage `voyage-3` → 0.95
• BGE-M3 → 0.94
• E5-large → 0.96

🧵

---

**2/**
This isn't a tuning problem.

InfoNCE on open-web data rewards topical similarity. Negation pairs are rare relative to topic variation. The model never learns a polarity-sensitive direction.

Going bigger doesn't help. The *ratio* of topical to polarity delta is roughly fixed across sizes.

---

**3/**
Six properties a rule-bearing embedding must preserve that general-purpose embeddings don't:

① Polarity (yes / no)
② Quantifier scope (all / some / exactly-k)
③ Structured conditions (if / unless / when)
④ Obligation strength (must > should > may)
⑤ Cross-rule dependency
⑥ Signed interference

---

**4/**
(6) is the one cosine similarity structurally can't give you.

Cosine is a magnitude. It has no sign.

What you want for rule-bearing content is a signed inner product: POSITIVE when two rules agree, NEGATIVE when they conflict, ZERO when unrelated.

That's the primitive Q-Prime exposes.

---

**5/**
Q-Prime: 1,536-dim embedding model, trained with multi-objective loss:

• polarity-flip contrastive term
• scope projection supervision
• ordinal regression on MUST > SHOULD > MAY
• auxiliary pairwise dependency head
• normalisation so that the inner product, not cosine, is the primitive

---

**6/**
Because the space is a proper Hilbert space, the Born rule applies:

$P(c|\psi) = |\langle c | \psi \rangle|^2$

Calibrated zero-shot classification without training. Compare to cosine-softmax, which is uniformly overconfident across embeddings we tested.

Method note: [QGI-TR-2026-04].

---

**7/**
Headline result on regulatory-conflict benchmark:

Cosine on 6 open-domain embeddings (4 orgs): **F1 = 0.000**

Q-Prime + Born-rule classifier: **F1 = 1.000**

Not an incremental improvement. The task isn't expressible in the other embedding space. It is in ours.

---

**8/**
"Not quantum-inspired." Real quantum formalism.

Hilbert space. Born rule. Superposition. Signed interference.

These are the mathematical primitives used without analogy. What we *don't* need is a QPU — the formalism runs on classical GPUs, because an embedding space IS a Hilbert space.

---

**9/**
Where this matters beyond regulated AI:

• Agent long-term memory (consistency, not just capacity)
• Context engineering (avoid retrieving contradictory context)
• Multi-agent coordination (reconcile differing memories)
• Any general-purpose AI that must reject contradictions

---

**10/**
Access:
• Preprint: [QGI-TR-2026-02]
• Companion QAG engine paper: [QGI-TR-2026-01]
• Q-Prime via OpenRouter (public beta)
• Enterprise SLA: contact@qgi.dev
• QAG engine GA: 21 June 2026

Critique welcome. research@qgi.dev.

/fin
