---
title: "If You're Modelling Policy in a Pairwise Graph, You're Modelling Something Else"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "LinkedIn Pulse"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~700
---

Your compliance team has a knowledge graph. It is built on Neo4j or AWS Neptune or TigerGraph. It has been growing for three years. It covers ten thousand regulatory paragraphs, a dozen jurisdictions, hundreds of products.

It is not a policy graph. It is a policy *index*. And the two are not the same thing.

A regulation is not a pairwise relationship. Take any real one and count the entities involved:

> *"A broker-dealer registered with FINRA must disclose material conflicts of interest to the client before executing an equity trade, unless the client has signed a Pro-Customer Waiver under Rule 2113."*

That single sentence relates six entities — broker-dealer, conflicts, client, trade, waiver, cross-referenced rule — with a signed obligation (must) that toggles under a structured condition (unless). A property-graph edge connects two. So your team did one of three things:

1. Reified the rule as a node with attributes, losing the graph semantics.
2. Broke the rule into five separate edges, losing the atomicity.
3. Stored the rule as a text blob and kept only metadata in the graph.

All three are lossy. Your "policy graph" is not a representation of the policy; it is a fragmented pointer back to the text.

## What you actually need: a hypergraph

A **hyperedge** can connect any number of nodes with typed roles. It's the right shape for a rule. But off-the-shelf hypergraph tools (HyperNetX, hypergraphx) don't handle the three properties enterprise policy needs:

1. **Versioning.** Rules change. You need time-travel queries: "what was the rule set as of January 1?"
2. **Signed semantics.** An obligation has polarity (must vs must not) and conditional sign ("must, unless").
3. **Conflict as a first-class edge.** When two rules contradict, the contradiction is not a property inferred at query time. It is an entity in the graph.

We built Quantum HyperGraph (QHG) specifically for this. It's described in our preprint [QGI-TR-2026-05] released today, and ships inside the QAG engine at GA on **21 June 2026**.

## What this buys you, operationally

1. **Atomic rule citation.** When an AI system answers "what is the disclosure requirement?", it points to a single hyperedge — the atomic unit your compliance team actually uses. No more citations to decomposed pairwise fragments.

2. **Conflict discovery as a scheduled job.** The conflict hyperedges are computed offline, across the whole graph, and stored. Discovery is O(1) at query time. On a 10,000-rule corpus, the offline pass runs in 3-5 minutes.

3. **Time-travelled queries.** "What was the obligation as of the date of the transaction?" is a standard query. The graph is immutable and versioned.

4. **Multi-party hyperedges.** A rule involving five parties is one hyperedge with five typed roles, not five pairwise edges that you have to re-assemble.

5. **Tombstone deletion for GDPR.** Immutable doesn't mean un-GDPR-able. Tombstone hyperedges mark entities as erased; the erasure ripples through queries without destroying the audit trail.

## A practical observation

The single most impactful change to an enterprise compliance stack in my career has been replacing a property graph with a hypergraph. The time to answer "is this procedure compliant?" dropped by 10x. The rate of false conflicts dropped by 5x. Auditors asked for fewer follow-ups.

Most of that isn't because hypergraphs are mysterious and powerful. It's because the representation *actually corresponds* to what a regulation is. When the data model matches the domain, everything downstream gets easier.

## Access

- **Preprint** on arXiv: *Quantum HyperGraph: A First-Class Data Model for Rule-Bearing Knowledge*.
- **QHG specification**: included in the preprint.
- **Full QAG engine** (includes QHG): GA 21 June 2026.
- **Architecture review conversations** for teams building their own policy graphs: `research@qgi.dev`.

If your compliance team has a property graph and is unhappy with it, I'd be interested to hear why. You're probably running into one of the three compromises above, and I've seen all of them.

---

**Dr. Sam Sammane** is CTO and Founder of Quantum General Intelligence, Inc. (`sam@qgi.dev`).
