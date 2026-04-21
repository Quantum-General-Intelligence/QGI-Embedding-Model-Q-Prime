---
title: "Hypergraphs for Policy: Why Your Neo4j-Shaped Compliance Graph Isn't Working"
byline: "Dr. Sam Sammane"
venue: "Medium / Towards Data Science"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~900
tags: ["knowledge graphs", "hypergraphs", "Neo4j", "compliance", "regulated AI"]
---

If you've been running a compliance knowledge graph in production, there's a specific pain point I'm going to call out in this post. Everyone I've talked to in the space has it. Almost no one talks about it.

The pain point: your policies are not pairwise relationships, but your graph forces them to be.

## The three-workaround problem

You have a graph. Neo4j, maybe. TigerGraph, maybe. AWS Neptune, maybe. It has nodes for entities, edges for relationships. You started building it to represent policies, and you quickly discovered that a policy doesn't fit.

Take any real regulation:

> *"A broker-dealer registered with FINRA must disclose material conflicts of interest to the client before executing an equity trade, unless the client has signed a Pro-Customer Waiver under Rule 2113."*

That's six entities: broker-dealer, conflicts, client, trade, waiver, rule. One obligation: "must disclose". One exception: "unless waiver signed". One cross-reference: "Rule 2113".

You have three ways to fit it into a property graph:

1. **Reify the rule as a node.** Attach six edges from the rule node to the six entities. Store the obligation as a property of the node. Store the exception as... another node, reified similarly. You now have a graph with the shape of a bipartite graph between rules and entities, but you've lost the reason for having a graph in the first place.

2. **Decompose into pairwise edges.** "broker-dealer→discloses→conflicts", "conflicts→to→client", "disclosure→before→trade", "disclosure→unless→waiver". Five edges for one rule. The atomicity of the rule is gone. Searching "what does Rule X require?" requires reassembling from five edges.

3. **Store the rule as a text blob.** The graph becomes a metadata index pointing to text. The graph part is now vestigial.

Every enterprise compliance graph I've reviewed uses one of these three. All three lose information.

## The shape of the right answer

The right data structure for policy is a **hypergraph**: edges that connect any number of nodes, with typed roles.

A hyperedge for the broker rule above would have six participating nodes, each with a typed role (subject, object, recipient, trigger, exception, cross-reference), a typed edge (governs), a sign (positive, conditionally), and a pointer to the original document.

Hypergraphs are not new. The literature is 40+ years old. What's new is treating them as a production data model for enterprise policy. Off-the-shelf tools (HyperNetX, hypergraphx) don't handle the properties that enterprise compliance needs:

1. **Versioning.** Time-travel queries on regulations that changed.
2. **Signed semantics.** Obligations have polarity; contradictions are first-class.
3. **Conflict as an edge.** When two rules contradict, the contradiction is an entity in the graph, not a query-time inference.
4. **Source provenance.** Every edge traces back to a document span.
5. **Tombstone for GDPR.** Immutable history + partial deletion semantics.

## QHG

We built Quantum HyperGraph (QHG) with all five. It's the substrate of QGI's QAG engine (canonical paper: [QGI-TR-2026-01]) and has its own preprint ([QGI-TR-2026-05]) because we think the data-model contribution stands on its own.

The "Quantum" in the name reflects the fact that each node has an associated Hilbert-space state vector, and hyperedges correspond to quantum *observables*. The Born-rule probability of a hyperedge — for example, the probability that two rules are in conflict — is a well-defined expectation value.

This is not stylistic. It means we get a principled probability framework over the symbolic graph without a separate model on top.

## What this actually gives you

1. **Atomic rule queries.** "What does Rule X say?" returns a single hyperedge, not five fragments.
2. **Compiled conflict discovery.** Contradictions are detected offline (once per ingestion), stored as `conflicts_with` hyperedges, and queried in O(1).
3. **Time-travel without external versioning.** "What was the rule set on January 1?" is a primitive query.
4. **Explicit provenance.** Every answer cites a document span, enforced by the data model, not by conventions in code.
5. **GDPR tombstones.** Right-to-be-forgotten is handled by tombstone hyperedges, without destroying audit history.

## Comparison

| Property | Neo4j | RDF | HyperNetX | QHG |
|---|---|---|---|---|
| Hyperedges | ✗ | ✗ | ✓ | ✓ |
| Typed roles | reify | ✓ | partial | ✓ |
| Signed edges | as property | via subgraphs | ✗ | ✓ |
| Versioning | external | external | ✗ | ✓ |
| Conflict as first-class | ✗ | ✗ | ✗ | ✓ |
| Source provenance | as property | ✓ | ✗ | ✓ |
| Time-travel | external | external | ✗ | ✓ |

QHG isn't the first hypergraph data model. It's the first we know of that treats all the above as first-class in one integrated system.

## Performance anecdote

On a 10,000-rule enterprise corpus, QHG's offline conflict-discovery pass runs in 3-5 minutes on a single GPU. Structural queries are ~5ms median. Semantic queries (HSC-re-ranked) are ~50ms median. This is competitive with a tuned property graph for the same workload, and about 5x faster than the "reify-as-node" property-graph pattern for conflict discovery, because reification forces a query-time join.

## Access

- **Preprint:** *Quantum HyperGraph: A First-Class Data Model for Rule-Bearing Knowledge* — arXiv link in the header.
- **Specification:** appendix of the preprint, including schema, query interface, and migration patterns from property graphs.
- **Hosted service:** part of the QAG engine, GA 21 June 2026.
- **Architecture review** conversations for teams stuck in one of the three workarounds: `research@qgi.dev`.

If your policy graph feels like a pointer back to text, the diagnosis is that your data model is pairwise and your policies aren't. QHG is one way out.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
