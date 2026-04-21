---
title: "Show HN: Reasoning-first memory for AI agents (preprint + integration with Letta/LangGraph/mem0)"
venue: "Hacker News (Show HN)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~280
---

Every agent memory framework shipping in 2026 (MemGPT, Letta, mem0, Zep, LangGraph memory, OpenAI Assistants) manages memory capacity. None of them model memory consistency.

The result: agents silently admit contradictions into memory and retrieve them together. Preference A from January and preference ¬A from March both score high on cosine, both get retrieved, generator picks one by attention bias.

Releasing [QGI-TR-2026-03] today, a preprint that:

1. Defines four operations a consistency-aware memory needs: `admit`, `consolidate`, `retrieve` (with consistency constraint), `reconcile`.
2. Grounds them in a signed-interference primitive — signed inner product in Hilbert space — which is positive when two memories agree, negative when they contradict, zero when unrelated.
3. Requires an embedding with a separable polarity axis to work. Our Q-Prime model has one; general-purpose embeddings do not.

90-day simulated evaluation of a 12,000-turn support agent:

- Contradiction admission rate: baseline 11.2%, QAG memory 0.3%.
- User-visible inconsistency: baseline 7.1%, QAG memory 0.6%.
- Agent asks user for arbitration: baseline 0.1/100 turns, QAG memory 1.8/100 turns.

The last metric is the interesting one. The QAG agent *notices* its own contradictions and asks the user. The baseline agent silently picks one.

Integration is additive — wrap the admission step in your existing memory store. Starter kits for Letta, LangGraph, mem0, Zep available.

Q-Prime API is in public beta on OpenRouter. QAG engine GA 21 June 2026.

This is one of six preprints released today from QGI. Full list at [URL].

Critique welcome. `sam@qgi.dev`.
