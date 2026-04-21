---
title: "Twitter/X thread — Reasoning-first memory for AI agents"
byline: "@sam_sammane / @qgidev"
venue: "Twitter / X"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
---

# Thread (11 tweets)

---

**1/**
Your AI agent has a memory bug.

When it retrieves memories relevant to a current query, it retrieves contradictions as if they were reinforcements.

Every agent memory shipped in 2026 has this bug. I can name them: MemGPT, Letta, mem0, Zep, LangGraph memory, OpenAI Assistants.

🧵

---

**2/**
The bug is at the retrieval primitive.

Cosine similarity is a *topical* distance. Two memories with identical topic and opposite polarity ("prefer X" vs "prefer NOT X") are nearly tied in vector space.

Both get retrieved. Generator synthesises. Attention bias picks one.

---

**3/**
Concrete production examples I've personally debugged:

• Agent with 9 different stacked user job titles, averaged in every answer
• Support agent mixing 2024 refund policy (30 days) with 2025 (14 days), consistently giving 21
• Codegen agent holding "Stripe is idempotent" AND "Stripe is not idempotent"

---

**4/**
These are not prompt bugs. They are primitive bugs.

What you need: a SIGNED signal between two memory items. Positive when they agree, negative when they contradict, zero when unrelated.

Cosine can't be negative in a meaningful way. The quantum inner product can.

---

**5/**
Signed interference:

$I(a, b) = \text{polarity}(a, b) \cdot \langle a | b \rangle$

For this to work, the embedding needs a polarity axis separable from topical content. General-purpose embeddings: don't have one. Q-Prime: has one by design.

---

**6/**
Four operations a consistency-aware memory needs:

① admit — classify and check against recent memory
② consolidate — on contradiction, raise an event
③ retrieve — select a CONSISTENT subset
④ reconcile — periodic housekeeping on contradictions

All four use the same signed primitive.

---

**7/**
The interesting new capability: the agent can NOTICE its own inconsistency.

In a 90-day simulation:

• Baseline agent asks user for clarification: 0.1 / 100 turns
• QAG-memory agent asks: **1.8 / 100 turns**

Not because it's dumber. Because it DETECTS contradictions it held silently before.

---

**8/**
"My agent is noisy but converges" is a fiction you can test.

Count contradictions admitted per 100 turns:

• mem0 baseline: 11.2%
• Letta capacity-only: 9.8%
• QAG reasoning-first: **0.3%**

The capacity-only designs get you almost nothing. Consistency is a separate axis.

---

**9/**
Integration is additive. You don't rewrite your stack.

```python
mem_wrapper.admit(item):
  signals = [I(item, recent_i) for i in top_k_recent]
  if any(s < -tau):
    raise_consolidation_event(item, contradict_set)
  else:
    underlying_mem.append(item)
```

Starter kits for Letta, LangGraph, mem0, Zep.

---

**10/**
This is NOT "hallucination detection" as commonly discussed.

It's *belief update triggering* — the agent recognising its own belief state is inconsistent with a new observation, then invoking a policy.

Separate and more reliable than an LLM-judge pattern over the output.

---

**11/**
Access:
• Preprint: [QGI-TR-2026-03]
• Q-Prime API: OpenRouter (public beta)
• QAG engine GA: 21 June 2026
• Starter kits: agents@qgi.dev

One of six preprints released today. If you run agents in prod, the paper and integration conversation are both free.

sam@qgi.dev.

/fin
