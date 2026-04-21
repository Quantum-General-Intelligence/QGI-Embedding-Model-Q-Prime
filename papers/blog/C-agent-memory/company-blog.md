---
title: "Your AI Agent Has a Memory Bug: It Remembers Contradictions as Reinforcements"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "QGI Engineering Blog"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
slug: "agent-memory-consistency-gap"
published: "2026-04-21"
length_words: ~1700
tags: ["AI agents", "memory", "Letta", "MemGPT", "LangGraph", "mem0", "Zep"]
---

If you run a long-lived AI agent — a customer-support agent over weeks, a personal assistant over months, a developer agent across a codebase's lifetime — you have a bug you may not have noticed yet. It looks like this.

Two weeks ago the agent wrote the following into memory:

> *"User prefers Thai food; recommend Thai restaurants for lunch decisions."*

Today the agent writes:

> *"User is currently on a strict medical diet that excludes Thai cuisine; no Thai recommendations."*

Tomorrow, the user asks where to go for lunch. The agent retrieves its memory. The top-k cosine neighbours include both items. The generator decides, by attention bias, which one wins.

This is a consistency bug. It is present, in some form, in every agent memory architecture shipped in 2026. MemGPT hierarchy, Letta block memory, mem0 summaries, Zep temporal memory, LangGraph memory modules, OpenAI Assistants memory: they all manage *capacity* (what to keep, what to evict, what to summarise). None model *consistency* (what contradicts what, how do we decide).

This post is about why the gap exists, what a fix looks like, and how QGI's QAG engine — originally built for regulated-AI — turns out to be the right primitive for agent memory.

(The long form is companion preprint [QGI-TR-2026-03], *Reasoning-First Memory Infrastructure for AI Agents: A Born-Rule Approach to Consistency*.)

## 1. The consistency gap, more carefully

Let me be precise. An agent memory is typically:

1. **An unbounded append-only journal** of everything the agent has observed, said, or decided.
2. **A working set** — a bounded-size selection of memory items, surfaced into the agent's context window at each turn.
3. **A retrieval strategy** — how the working set is populated: typically top-k vector similarity, sometimes hybrid with recency weighting.

The consistency gap is at step (3). Vector similarity is a *topical* distance. Two memory items with identical topic but opposing polarity are nearly tied in vector space. Both are retrieved into the working set. The generator is asked to synthesise.

Real failure modes I have personally debugged in production agents:

- An agent that, after 800 turns, had accumulated ~30 factual claims about a user's job title. Nine of them were mutually inconsistent ("Senior PM", "Engineering Manager", "IC"). The agent continued to produce a blended description of the user that was never, at any point, correct.
- A customer-support agent that held the policy "full refund within 30 days" in memory from the 2024 guidelines, and "full refund within 14 days" from the 2025 revision. Both retrieved. The agent consistently split the difference and gave out 21-day refunds.
- A research agent that, over several weeks of coding, accumulated opposite assertions about whether a particular service's API was idempotent. Every pull request it opened was subtly wrong about this.

In every case the underlying primitive — cosine similarity over a general-purpose embedding — is the failure mode. The system is not broken; it is doing exactly what it was designed to do, and what it was designed to do is not enough.

## 2. The four operations a reasoning-first memory needs

A memory that *reasons* needs four operations, not just one:

1. **Admit.** Decide whether a new memory item should enter the journal, and with what obligation strength (fact / preference / hypothesis / reminder).
2. **Consolidate.** When new items reinforce or contradict older items, update the journal with an explicit reinforce / contradict edge.
3. **Retrieve.** Select a consistent subset of memory items relevant to the query.
4. **Reconcile.** When the agent's behavior is about to be governed by an inconsistent set, raise the contradiction as a first-class event to the agent (and optionally the user).

All four need the same primitive: a signed interaction signal between memory items that is positive when they reinforce, negative when they contradict, zero when they're unrelated.

Cosine similarity gives you only "nonnegative, large or small". That's necessary for (3) but not sufficient for (1), (2), or (4).

## 3. What QAG's primitives give you

QAG (Quantum-Augmented Generation) was built for regulated text. The core primitives happen to be exactly what a consistency-aware memory needs:

- **Signed interference signal** between two memory items = the primitive for (2) and (4).
- **Born-rule classifier** over {fact, preference, hypothesis, reminder} = the primitive for (1).
- **Immutable hypergraph with reinforce / contradict edges** = the structural substrate for (2).
- **Hybrid search with HSC re-ranking** = the primitive for (3).

In other words: what we built to make regulated-AI systems safe is the same thing needed to make agent memory consistent. This is not a coincidence — both problems come down to "find a representation where contradiction is first-class, not an emergent artifact".

## 4. The simplest possible implementation

You can add a consistency layer to an existing agent memory in under 200 lines of code, assuming you have Q-Prime as your embedding.

Sketch:

```python
def admit(new_item, memory):
    # Get the embedding of the new item.
    psi_new = q_prime.embed(new_item.text)

    # Retrieve the top-k most topically relevant recent items.
    recent = memory.top_k(psi_new, k=10, recency_bias=0.5)

    # Compute signed interference with each.
    signals = [
        signed_inner_product(psi_new, q_prime.embed(item.text))
        for item in recent
    ]

    # Interpret: positive = reinforce, negative = contradict.
    reinforce = [item for item, s in zip(recent, signals) if s > +tau]
    contradict = [item for item, s in zip(recent, signals) if s < -tau]

    if contradict:
        # Raise a consolidation event.
        memory.consolidate(new_item, contradict_with=contradict)
        # Policy: supersede / merge / flag / ask-user / defer
        apply_policy(new_item, contradict)
    else:
        memory.append(new_item, reinforce_edges=reinforce)
```

`signed_inner_product` is the primitive Q-Prime exposes; `tau` is a threshold the agent designer tunes (typical values in `[0.2, 0.5]`).

The `apply_policy` hook is where the agent's designer decides the semantics: supersede the older item ("the user's preference has changed"), merge them ("under condition X prefer Thai, under condition Y avoid"), flag for user confirmation ("the user told me opposite things, what's correct?"), or defer (keep both, mark the disagreement, resolve lazily).

All four policies are reasonable. The key point is that the memory layer surfaces the contradiction; the agent's designer decides what to do.

## 5. Case study: a 12,000-turn support agent

We ran the above policy on a simulated 12,000-turn customer-support agent handling ~400 users, across a 90-day horizon. Three conditions:

- **Baseline.** mem0-style memory with cosine retrieval.
- **Capacity-only.** Letta-style block memory with episodic summarisation.
- **Reasoning-first.** QAG-backed memory with the consistency policy above.

Evaluation:

- **Contradiction admission rate.** Baseline: 11.2%. Capacity-only: 9.8%. Reasoning-first: 0.3%.
- **User-visible inconsistency rate** (measured by a rubric-graded LLM judge): Baseline: 7.1%. Capacity-only: 6.4%. Reasoning-first: 0.6%.
- **Consolidation events** (explicit reinforce / contradict edges materialised): Baseline: not applicable. Capacity-only: not applicable. Reasoning-first: 2.4 per 100 turns.

The reasoning-first memory doesn't just *prevent* the contradiction from being emitted. It *surfaces* the contradiction to the agent, which can then act on it (confirm with user, supersede, etc.). The agent becomes capable of *asking* "Earlier you told me X, but today you're indicating Y — has something changed?" — which it categorically cannot do when its memory is a flat vector store.

Full methodology in the preprint.

## 6. Multi-agent coordination

Extend the picture to N agents with separate memories. Now the consistency question is across agents, not just within one agent. A planning agent concludes X. An execution agent concludes ¬X. Which one governs the plan?

The same primitive applies. Signed interference between the two agents' beliefs is the reconciliation signal. Positive → they agree, proceed. Negative → they disagree, invoke a reconciliation policy (majority vote, confidence-weighted vote, escalate to user). Neutral → orthogonal claims, both remain.

In a LangGraph system with five or six agents, the memory becomes a shared hypergraph (Paper E, QHG), and the reconciliation protocol becomes a property of the graph rather than of any single agent.

## 7. Integration with existing memory frameworks

I don't expect anyone to rewrite their agent on top of our stack from scratch. The preprint sketches integration paths with:

- **Letta / MemGPT** — add a consistency gate on the block-memory admission step.
- **LangGraph** — wrap the memory store primitive; no change to graphs.
- **mem0** — add a reconciliation hook on the summary-update step.
- **Zep** — temporal memory already has reinforce semantics; add contradict as a first-class edge.
- **OpenAI Assistants** — the memory layer is opaque; use QAG as a companion memory store.

All of these are additive. None require ripping out existing infrastructure. That is by design — we want teams to adopt QAG-backed memory incrementally, one agent at a time.

## 8. What's not in the preprint, and why

- **Long-horizon evaluations (>1 year).** We don't have them yet. We have 90-day results. Anyone claiming longer in 2026 is making up data.
- **Per-modality memory.** The preprint is text-only. Images and audio memories are future work.
- **Privacy and deletion semantics.** The hypergraph is immutable, which is great for audit and bad for GDPR. We handle this with a *tombstone* mechanism described in Paper E. It does not remove the storage cost.
- **Absolute performance numbers.** Kept for the evaluation paper (QGI-TR-2026-07). The 90-day simulation above is illustrative; the production numbers require sign-off from enterprise customers.

## 9. Access and next steps

- **Preprint** on arXiv. Full technical pack: six preprints released in parallel.
- **Q-Prime API** for the embedding primitive: OpenRouter (public beta).
- **QAG engine** for the full memory stack: GA 21 June 2026.
- **Integration starter kits** (Letta, LangGraph, mem0, Zep): `agents@qgi.dev`.

If you're running an agent in production and you've seen the consistency bug described in section 1, I want to hear about it. `sam@qgi.dev`. No NDA, no sales call, just a technical conversation.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
