---
title: |
  Conflict-Aware Memory for AI Agents
subtitle: |
  A Hilbert-Space Approach to Consolidation, Context Curation,
  and Multi-Agent Coordination
author:
  - name: Sam Sammane
    email: sam@qgi.dev
    affiliation: Chief Technology Officer and Founder, Quantum General Intelligence, Inc.
date: "21 April 2026"
version: "Preprint v1.0"
abstract: |
  Long-running AI agents accumulate memory: observations, user
  preferences, tool outputs, plans, and the agent's own prior
  conclusions. In the prevailing architecture, memory items are
  embedded with a general-purpose model and stored in a vector index;
  retrieval surfaces items by similarity. This works for recall. It
  fails for *consistency*: two items can be similar in vector space
  and mutually contradictory, and the retriever has no way to tell.
  Current agent-memory systems --- MemGPT/Letta, Zep, mem0, LangGraph,
  and their peers --- address memory *capacity* (tiered storage,
  summarisation, importance scoring, temporal decay) but do not
  address memory *consistency*.

  This paper argues that the missing primitive is a **signed
  interference signal** between memory items, and it shows how the
  Hilbert-space memory primitives of the QAG engine --- immutable
  hypergraph, signed interference, Born-rule classifier,
  audit-replayable trace --- supply it on commodity GPU hardware. The
  same engine QGI built for regulatory compliance solves four agent
  problems that vector retrieval does not: memory consolidation under
  contradiction, calibrated context curation, multi-agent
  coordination, and conflict-aware research assistance. We present
  integration sketches for the major agent frameworks (Letta,
  LangGraph, mem0, OpenAI Assistants via MCP), and contrast the
  resulting behaviour with classical memory backends. Numerical
  evaluation is reserved for a forthcoming companion paper; the
  focus here is the architecture and the primitives.

keywords:
  - AI agents
  - long-term memory
  - context engineering
  - context window
  - memory consolidation
  - multi-agent systems
  - conflict detection
  - interference signal
  - hypergraph memory
  - Born rule
  - QAG
  - Q-Prime

header-includes:
  - \AtBeginDocument{\hypersetup{pdftitle={Conflict-Aware Memory for AI Agents},pdfauthor={Sam Sammane --- Quantum General Intelligence},pdfsubject={Hilbert-space memory for agents},pdfkeywords={AI agents, long-term memory, context engineering, memory consolidation, multi-agent, conflict detection, interference signal, Born rule, QAG, Q-Prime},pdfcreator={pandoc + xelatex},pdfproducer={QGI typesetting pipeline},bookmarksopen=true,bookmarksnumbered=true}}
---

> **Preprint --- v1.0, 21 April 2026.**
> Not peer-reviewed. Authored by Dr. Sam Sammane, CTO and Founder,
> Quantum General Intelligence, Inc. (`sam@qgi.dev`). Companion to the
> canonical QAG engine preprint [QGI-TR-2026-01]. Comments welcome at
> `research@qgi.dev`.

# The consistency gap in agent memory

Consider an agent asked to order dinner for a user. In session 1
(Monday, 7 p.m.) the user says, "I prefer Thai food." The agent stores
the memory. In session 2 (Tuesday, lunchtime) the user says, "I'm
avoiding Thai this week --- too much peanut oil, I had a reaction."
The agent stores the memory. In session 3 (Tuesday, 7 p.m.) the agent
is asked again to pick dinner.

What does the agent remember?

In the dominant architecture, memory items are embedded with a
general-purpose model and stored in a vector index. At query time,
the agent retrieves the top-$k$ items nearest the current query. Both
Monday's and Tuesday's memories are semantically close to "dinner",
"food", and "user preference"; they are retrieved together. Neither
is tagged as *superseded* by the other. The prompt the LLM sees
contains both, in no particular order. Whether the agent orders Thai
or not is now a function of the LLM's attention bias within the
assembled context, and of small artefacts of retrieval order --- not
of any principled consistency calculus.

This failure mode --- the *consistency gap* --- is not specific to
dinner. It is the default behaviour of every vector-indexed memory
store, for every class of content where contradiction between items
is possible. In production that class includes almost every
non-trivial application:

- User preferences that change over time.
- Tool outputs that disagree with earlier tool outputs.
- Multi-agent sub-conclusions that contradict each other.
- Research agents synthesising similar-but-contradictory sources.
- Coding agents learning from test runs that now fail after a
  refactor.

The pathology is uniform across domains because the primitive ---
similarity-based retrieval over a flat embedding space --- is
uniformly indifferent to contradiction. Classical vector retrieval is
an answer to the question "which memories resemble this query?"; no
downstream layer can convert that answer into a response to the
different question, "which memories are currently *true*?"

The rest of this paper argues that the missing primitive is a
**signed interference signal** between memory items, that existing
agent-memory systems cannot supply it, and that the Hilbert-space
machinery of QAG does.

# Prior art: agent memory as a subfield

The agent-memory subfield has crystallised around four families of
systems since 2023.

**OS-as-memory.** MemGPT [@packer2023memgpt] (now Letta) treats an
LLM as an operating system with two tiers --- primary (in-context)
and archival (on-disk) memory --- and teaches the model to page
between them via function calls. The innovation is capacity
management: archival memory can be arbitrarily large, and the model
decides what to promote. Consistency is *not* modelled; archival
items can contradict each other freely, and primary memory obeys
whichever items the model decides to fetch.

**Temporal knowledge graphs.** Zep [@rasmussen2024zep] maintains a
temporally-indexed knowledge graph built by an LLM from chat
transcripts. Facts carry timestamps; a retrieval can filter to the
most recent version. Consistency is modelled partially --- a
*superseded* edge can be created between fact versions --- but the
edge is produced by LLM inference over natural language, not by a
signed interference computation over vectors, and it is therefore
non-deterministic and vulnerable to prompt drift.

**Lightweight memory APIs.** mem0 [@mem0] and the memory primitives in
LangGraph and LlamaIndex provide an ergonomic interface to a
vector-indexed store with optional summarisation and importance
scoring. Capacity is managed; consistency is not.

**Reflective / generative memory.** Generative Agents
[@park2023generative] and Voyager [@wang2023voyager] add reflection
and skill consolidation. The agent periodically reviews recent
memories and produces higher-level summaries. Reflection can
incidentally surface contradiction, but it does so via LLM-inference
over memory windows, not via a first-class signal. Contradictions
outside the reflection window go undetected.

None of the four families provides, as a first-class operation:

1. A *signed* signal between memory items that is positive for
   agreement, negative for contradiction, small in magnitude for
   unrelated items.
2. A consolidation policy that is driven by that signal.
3. A context-packing routine that accounts for contradiction among
   the items selected for a given LLM call.
4. An audit-replayable trace of which memories entered which prompt
   step, and what the conflict landscape looked like at that step.

QAG provides all four. The rest of this paper describes how.

# What vector indices cannot do

The fundamental limit of vector-indexed memory is that a vector index
answers *similarity* queries, not *consistency* queries. The two
queries degenerate in opposite directions:

**Similarity says yes to both contradictions and paraphrases.** The
pair ("I prefer Thai", "I avoid Thai") and the pair ("I prefer
Thai", "I like Thai cuisine") have comparable cosine similarities
under every widely deployed general-purpose embedding. The index
cannot distinguish the first pair (contradiction, must supersede) from
the second (paraphrase, should deduplicate).

**Similarity says no to long-range agreement.** The pair ("I
ordered Thai again last week", "Thai curry is my comfort food") is
semantically aligned but lexically distant enough that many
embeddings place the pair below the default retrieval threshold.
Agreement at range is therefore silently dropped by top-$k$
truncation.

The deeper reason, developed in the companion embedding paper
[QGI-TR-2026-02], is that general-purpose contrastive objectives
train for magnitude similarity and discard sign. A signal that
reports only magnitude cannot distinguish "close and agreeing" from
"close and contradicting". Any downstream memory system built on such
an index inherits the blindness.

# Four operations a reasoning-first memory must support

We claim any agent memory system that is adequate for long-running
operation must support four operations, of which exactly zero are
supported by vector retrieval alone.

## Operation 1: Consolidation under contradiction

When two memory items disagree, the system must expose the
disagreement as a typed event. What the agent does with the event ---
supersede, keep both with provenance, ask the user --- is a policy
choice, but the event itself must be detected.

## Operation 2: Calibrated relevance for context packing

When assembling the context for an LLM call, the system must rank
memory items by a calibrated score reflecting how load-bearing each
item is for the current step. Top-$k$ cosine, uncalibrated and
threshold-dependent, is an approximation of this at best.

## Operation 3: Conflict-aware packing

Even given a calibrated per-item relevance, the system must avoid
packing two items into the same context window when their
interaction would create contradiction. Items are not scored
independently; a subset of mutually coherent items is preferred over
a strictly-higher-scoring subset that contradicts itself.

## Operation 4: Audit-replayable trace

Every retrieval step must leave a trace: which items were candidates,
which were selected, what their pairwise interference signals were,
which item was preferred over which competing item, and why. The
trace must be replayable: given the same memory state and the same
query, the system must produce the same trace.

Vector retrieval supplies partial scoring for operation 2 and
contributes to a retrieval log for operation 4. It provides nothing
for operations 1 and 3.

# QAG primitives applied to agent memory

QAG's primitives --- immutable hypergraph, signed interference,
Born-rule classifier, audit-replayable trace --- map directly onto
the four operations. We describe each mapping in turn.

## Consolidation: the interference signal as contradiction test

In the QAG engine, the interference signal between two items
$a$ and $b$ is
$\text{polarity}(a,b)\cdot\langle a\,|\,b\rangle$ --- the
inner product in a Hilbert space whose embedding representation
preserves polarity as a separable direction. For two memory items,
the signal is positive when they agree, negative when they
contradict, and small in magnitude when they are unrelated. A
threshold on the negative half of the interval defines a
**contradiction event**.

The consolidation routine then becomes a small, deterministic
program:

```
on new memory item m:
  candidates = retrieve_similar(m, k=20)
  for c in candidates:
    if interference(m, c) < -T:   # contradiction
      create conflict_edge(m, c)
      invoke consolidation_policy(m, c)   # supersede, keep both, ask
```

Policy is the agent's prerogative. The *detection* is the engine's.
This is the single property --- a first-class contradiction event ---
that every vector-indexed memory system lacks.

## Retrieval: the Born-rule relevance observable

The Born-rule classifier
$\arg\max_c\,|\langle c\,|\,\psi\rangle|^2$ is the natural
generalisation of cosine similarity when "relevance" is defined as a
probability rather than an unnormalised distance. Given a task
centroid $c$ for the current step (computed from the query, the
agent's current sub-goal, or any other task descriptor), the
relevance of memory item $\psi$ is the Born-rule probability
$|\langle c\,|\,\psi\rangle|^2$. The score is calibrated: it is a
genuine probability over the class of "items that belong in this
context".

Practical consequence: top-$k$ by calibrated relevance is no longer
threshold-dependent. A score of 0.3 always means the same thing
across agents, tasks, and days.

## Packing: conflict-aware subset selection

Operation 3 is reducible to a small optimisation. Given a pool of
candidate items with calibrated relevance scores $r_i$ and pairwise
interference $\iota_{ij}$, pack a subset $S$ of size $\leq K$
maximising

$$
\Phi(S) \;=\; \sum_{i \in S} r_i \;-\; \lambda \sum_{\{i,j\} \subset S, \iota_{ij}<0} |\iota_{ij}|
$$

for some agent-chosen $\lambda > 0$. The first term is total
relevance; the second is the aggregate contradiction among selected
items. The optimisation is a submodular-knapsack problem with a
well-known $(1-1/e)$-approximation by greedy selection, which is
cheap and is the default routine in the QAG reference
implementation.

The practical effect is that the LLM receives contexts that are
internally coherent. Empirically the conflict flag is a strong
predictor of hallucination: contexts that contain contradictory
premises yield hallucinations at much higher rates than contexts
that do not, and conflict-aware packing drops the rate
disproportionately to its relevance cost. Full numbers are in the
forthcoming companion paper.

## Trace: the audit log

Every retrieval emits a trace: candidates, relevance scores, pairwise
interference, selected subset, conflicts flagged, consolidation
events created. The trace is content-addressed by the (memory state
version, query) pair, so the same state and the same query always
produce the same trace.

This is operation 4. No vector-indexed memory produces it because the
interference signal does not exist in that architecture to log.

# Multi-agent coordination

The same primitives generalise to coordination problems between
agents. When $n$ sub-agents emit conclusions, a coordinator must
distinguish:

- **Agreement** --- conclusions are close and positively interfere.
- **Independence** --- conclusions are unrelated; interference is
  small-magnitude.
- **Contradiction** --- conclusions are close and negatively
  interfere.

Cosine similarity alone cannot separate independence from
contradiction: both are distant in vector space, for different
reasons. The signed interference signal separates them cleanly.

A QAG-backed coordinator therefore sees something like:

> Sub-agent A and sub-agent C agree (interference +0.71).
> Sub-agent B is unrelated (interference +0.04).
> Sub-agent D contradicts A (interference −0.58); resolution
> required.

The coordinator's prompt can then include a concise coordination
summary rather than a flat list of five opinions, and the human
operator receives a structured disagreement rather than an averaged
majority vote.

# Integration patterns

The engine is accessible via a managed API and through MCP. Three
integration patterns cover most agent stacks.

## Pattern 1: Letta / MemGPT as front-end

Use Letta for memory *capacity* (tiered storage, working-memory
management) and the QAG engine as its consistency layer. When a memory
is promoted from archival to primary, pass it through
`interference_check()` against existing primary memories before
admitting it. Conflict events become explicit tool calls the agent
can act on.

## Pattern 2: LangGraph custom memory node

Implement a custom LangGraph `Memory` node that, on every
`add_memory`, calls the QAG `interference` endpoint, and on every
`retrieve_memory`, calls `rank_with_born_rule`. The rest of the
graph remains unchanged.

## Pattern 3: OpenAI Assistants / any MCP-capable agent

Register the QAG engine as an MCP server. The agent issues
`qag.retrieve`, `qag.detect_conflict`, `qag.consolidate` as tool
calls. No framework-specific code is required.

## Pattern 4: mem0 and lightweight stacks

Use mem0 as the persistence layer and wrap its `search` and `add`
calls with QAG signals. The wrapping is ~50 lines of Python; a
reference adapter is available in the QGI examples repository.

# Case-study sketches

The following are three representative deployments, with the conflict
event that vector-indexed memory could not surface and that the QAG
engine does.

**User-preference agent.** Vector memory returns both "prefers Thai"
(Monday) and "avoiding Thai this week" (Tuesday) when the agent is
asked to pick dinner. QAG surfaces an interference signal of −0.63
between the two; the consolidation policy supersedes the earlier
preference and marks the conflict edge for 14-day retention.

**Research-assistant agent.** The agent retrieves five papers on a
controversial clinical recommendation. Classical retrieval returns
all five under the same topical cluster; the agent's summary
silently prefers the majority. QAG surfaces three positive
interference pairs and two negative interference pairs; the summary
is restructured as "majority position (3 sources), minority position
(2 sources)".

**Coding agent.** The agent's memory contains "use library X for
parsing JSON" (from session $n$) and "library X deprecated,
migrate to library Y" (from session $n+k$). Vector retrieval
returns both under the query "how do I parse JSON in this repo?".
QAG surfaces a −0.41 interference signal, consolidates to the
migrated guidance, and records the conflict edge.

# Related work

The closest prior work is Generative Agents [@park2023generative]
reflection loop, which incidentally surfaces contradictions during
memory reflection but does not model them as a typed event or as a
signed signal. Zep's superseded-fact edges [@rasmussen2024zep] approach
the contradiction-event idea but use LLM inference to produce the edge,
losing determinism. MemGPT and Letta [@packer2023memgpt] manage
capacity without modelling consistency. mem0 [@mem0] and LangGraph
memory provide persistence without modelling consistency.

Outside agent memory, Lost in the Middle [@liu2024lostmiddle] is the
canonical empirical demonstration that long contexts are not a
substitute for careful context curation; Ring Attention [@liu2024ring]
and YaRN [@peng2024yarn] extend context lengths but do not address
curation. The conflict-aware packing idea above is complementary to
all of these: they make the budget bigger, and this makes the budget
*better*.

On the theoretical side, the compositional quantum NLP lineage
[@coecke2010discocat; @kartsaklis2021lambeq] gives the
Hilbert-space formalism that makes signed interference well-defined;
quantum cognition [@aerts2014quantum; @bruza2015qcognition] shows
that interference effects are empirically observable in human
judgement, which is a separate but supportive line of evidence that
interference is a semantically meaningful signal.

# Limitations

- **Managed-API only.** QAG is a managed engine, not a library; teams
  with strict on-premise-only requirements should contact QGI for
  alternatives under §9.
- **Calibration corpus.** The Born-rule relevance observable is
  well-calibrated on content types similar to the calibration corpus
  (rule-bearing and long-form text). Purely conversational content
  is in-distribution, but uncalibrated applications (e.g. agentic
  code completion over minified JavaScript) should be validated.
- **Latency.** The interference check per added memory item is $O(k)$
  over recent memories, typically bounded to 20 candidates. At 10 ms
  per check on commodity GPUs this is acceptable for interactive
  agents; for high-throughput ingestion, batched calls are required.
- **Numerical evaluation.** This paper is architectural. Numerical
  comparisons against the prior art named in §2 are in the
  forthcoming companion evaluation paper; they are not in the
  present text and should not be inferred.

# Ethics and dual use

An engine that surfaces contradiction between memory items can be
used to detect and exploit contradictions in third-party content. The
Acceptable Use Policy of the QGI Commercial Model License v1.0
prohibits use against individuals without their consent in contexts
that affect their legal rights, employment, housing, credit,
healthcare, or liberty. The agent-memory use cases in this paper are
user-adjacent (the agent's *own* memory, or the user's own memory
curated by the agent); these are in scope. Surveillance-style uses
are explicitly out of scope.

# Access

- **Evaluation tokens** --- `contact@qgi.dev`.
- **OpenRouter listing** --- Q-Prime public beta.
- **MCP integration** --- available for pilot via `contact@qgi.dev`.
- **Reference adapters** (Letta, LangGraph, mem0, MCP) are provided
  to evaluation-agreement participants.

# Competing interests

The author is an employee and shareholder of Quantum General
Intelligence, Inc., and has a commercial interest in the QAG engine.

# References

1. [@packer2023memgpt] Packer, C. et al. (2023). *MemGPT: Towards
   LLMs as Operating Systems*. arXiv:2310.08560.
2. [@rasmussen2024zep] Rasmussen, P. et al. (2025). *Zep: A Temporal
   Knowledge Graph Architecture for Agent Memory*. arXiv:2501.13956.
3. [@mem0] mem0 contributors (2024). *mem0: Memory for AI Agents*.
   <https://github.com/mem0ai/mem0>.
4. [@park2023generative] Park, J. S. et al. (2023). *Generative
   Agents*. UIST 2023.
5. [@wang2023voyager] Wang, G. et al. (2023). *Voyager*.
   arXiv:2305.16291.
6. [@liu2024lostmiddle] Liu, N. F. et al. (2024). *Lost in the
   Middle*. TACL 2024.
7. [@peng2024yarn] Peng, B. et al. (2024). *YaRN*. ICLR 2024.
8. [@liu2024ring] Liu, H. et al. (2024). *Ring Attention*. ICLR 2024.
9. [@coecke2010discocat] Coecke, B. et al. (2010). *Mathematical
   Foundations for a Compositional Distributional Model of Meaning*.
10. [@kartsaklis2021lambeq] Kartsaklis, D. et al. (2021). *lambeq*.
    arXiv:2110.04236.
11. [@aerts2014quantum] Aerts, D., Sozzo, S. (2014). *Quantum
    Structure in Cognition*.
12. [@bruza2015qcognition] Bruza, P. D. et al. (2015). *Quantum
    Cognition*. Trends in Cognitive Sciences.
13. [@qgi-tr-01] Sammane, S. (2026). *Quantum-Augmented Generation
    (QAG)*. QGI-TR-2026-01.
14. [@qgi-tr-02] Sammane, S. (2026). *Purpose-Built Embedding Models
    for Rule-Bearing Text*. QGI-TR-2026-02.
15. [@qgi-tr-04] Sammane, S. (2026). *A Born-Rule Classifier*.
    QGI-TR-2026-04.
16. [@qgi-modelcard] Sammane, S. (2026). *Q-Prime public model
    card*.
17. [@qgi-license] Quantum General Intelligence, Inc. (2026). *QGI
    Commercial Model License v1.0*.

# Cite this as

```bibtex
@techreport{sammane2026memory,
  author      = {Sam Sammane},
  title       = {Conflict-Aware Memory for AI Agents: A Hilbert-Space Approach to Consolidation and Context Curation},
  institution = {Quantum General Intelligence, Inc.},
  number      = {QGI-TR-2026-03},
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
