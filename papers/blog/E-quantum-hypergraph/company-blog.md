---
title: "Your Policy Graph Is Lying to You: Pairwise Edges Cannot Represent Rules"
byline: "Dr. Sam Sammane — CTO and Founder, Quantum General Intelligence"
venue: "QGI Engineering Blog"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
slug: "quantum-hypergraph-policy-graph-data-model"
published: "2026-04-21"
length_words: ~1500
tags: ["knowledge graphs", "hypergraphs", "policy", "compliance", "data model"]
---

The last five years of enterprise knowledge graph work has converged on the **property graph** — Neo4j, Amazon Neptune, TigerGraph, Memgraph — as the default data model. Property graphs are good for a lot of things: organisational networks, supply chains, product catalogs, code dependencies. They are not good for *policy*.

If you have tried to model a regulation, a contract, or a clinical guideline in a property graph, you have probably hit the following wall. A regulation is not a pairwise relationship. It is a relationship among three, four, or more entities at once, with typed conditions and a signed stance. A property-graph edge cannot carry that structure. Your workaround is probably one of:

- Reifying the edge as a node, with attributes (and losing the crisp graph semantics).
- Decomposing the rule into several pairwise edges (and losing the atomicity of the rule).
- Storing rules as text blobs outside the graph and keeping only metadata in the graph (and losing the graph altogether).

None of these work. Each gives you a property graph that is shaped like a policy and *doesn't mean* the policy.

This post is about the data model we've built for policy: **Quantum HyperGraph (QHG)**. It is the substrate of the QAG engine described in our canonical paper [QGI-TR-2026-01], and we've given it its own preprint [QGI-TR-2026-05] because we believe the data-model choices are independently interesting for any team working on regulated-AI, compliance, or agent memory.

## 1. What's wrong with pairwise edges for policy

Consider the rule:

> *"A broker-dealer registered with FINRA must disclose material conflicts of interest to the client before executing an equity trade, unless the client has signed a Pro-Customer Waiver under Rule 2113."*

Count the entities:

- *broker-dealer* (subject, with an attribute: registered-with-FINRA)
- *material conflicts of interest* (object)
- *client* (recipient)
- *equity trade* (event triggering the rule)
- *Pro-Customer Waiver* (exception condition)
- *Rule 2113* (cross-reference supporting the exception)

Six entities are related by a single rule. The rule has:

- A subject and an object (two entities, so maybe a pairwise edge)
- A recipient (three now)
- A triggering event (four)
- An exception condition (five, relating the waiver to the rule)
- A cross-reference (six, linking the waiver to its governing rule)

A pairwise-edge graph has to decompose this into 5-6 separate edges, losing the *atomicity* of "one rule". Compliance teams care about this atomicity — when they say "Rule FINRA-2010-5(b)" they mean the unit, not the decomposition.

Moreover, the rule has a *signed stance*: the obligation direction is positive (must disclose) unless the exception fires (then the obligation is suspended). There's no clean representation of a sign on a pairwise edge with an exception.

The shape of the right data structure is a **hyperedge**: an edge that connects any number of nodes, with typed attributes, including a sign.

## 2. Hypergraphs, briefly

A **hypergraph** generalises a graph by allowing edges to connect more than two nodes. A hyperedge $e$ in a hypergraph $H = (V, E)$ is a subset of $V$ of arbitrary cardinality. Formally:

$$
H = (V, E), \quad E \subseteq \mathcal{P}(V) \setminus \{\emptyset\}
$$

The literature distinguishes *undirected* hypergraphs (symmetric hyperedges) and *directed* hypergraphs (ordered or role-assigned). Policy needs the latter: "broker discloses X to client" has roles.

For policy, we further need:

1. **Typed roles** per hyperedge. "Subject", "object", "recipient", "trigger", "exception condition" are distinct roles.
2. **Typed hyperedges**. A rule hyperedge has different semantics from a definition, a cross-reference, or a conflict.
3. **Signed** hyperedges. Obligation can be positive (must), negative (must not), or conditionally signed (must, unless).
4. **Versioning**. Regulations change. Hyperedges are versioned; older versions are retained for audit.
5. **Conflict as a first-class edge type**. Two rules contradict each other; this is a hyperedge from both rules to a conflict node, with type `conflict`.

No production property-graph implementation handles all five. A few hypergraph libraries (HyperNetX, hypergraphx) handle (1) and (2). None handle (3)–(5) out of the box.

## 3. QHG: what we built

QHG (Quantum HyperGraph) is a versioned, immutable, directed hypergraph with typed hyperedges and signed semantics.

- **Immutability.** No update-in-place. All changes produce a new version. Older versions retained.
- **Versioning.** Every hyperedge carries (`version_start`, `version_end`). Queries are time-travelable: "as of January 1, what was the state of the graph?".
- **Typed nodes.** Nodes come in typed flavours: *rule*, *predicate*, *entity*, *jurisdiction*, *exception*, *cross-reference*.
- **Typed hyperedges.** Hyperedges come in typed flavours: *governs*, *defines*, *references*, *conditional_on*, *conflicts_with*, *supersedes*.
- **Signed hyperedges.** Where relevant, hyperedges carry a sign (positive, negative, or conditionally-signed with a guard).
- **Role-assigned participation.** Each node in a hyperedge has a typed role.
- **Source traceability.** Every hyperedge carries a pointer to the source document and span from which it was extracted.
- **Conflict as a first-class edge.** When two rules contradict, the conflict is materialised as a hyperedge of type `conflicts_with` connecting the two rule nodes to a shared conflict node. The intelligence layer (HSC) produces this hyperedge; the graph stores it.

## 4. Why "quantum"?

The "quantum" in QHG refers to the interaction with the Hilbert-space intelligence layer — the HSC — that sits on top of the graph.

Each node in QHG has an associated embedding in a Hilbert space. Each hyperedge's semantics can be expressed as an *observable* on the joint state of its participating nodes. For example:

- The *conflict* observable is $\hat{O}_{\mathrm{conflict}} = \hat{P}_a \otimes \hat{P}_{\bar b} + \hat{P}_{\bar a} \otimes \hat{P}_b$, where $\hat{P}_x$ projects onto the "asserts $x$" subspace.
- The *reinforces* observable is its complement.
- The *subsumes* observable is an ordered product.

This makes certain graph queries expressible as expectation values on the joint state. It gives us the Born rule for hyperedge probability.

We are not claiming this is the only way to build a policy graph. We are claiming it is a particularly clean way, because it unifies the *symbolic* graph layer and the *probabilistic* intelligence layer in a single algebraic framework.

## 5. The query interface

QHG exposes four query families:

1. **Structural.** "Return all rules that govern trades of equity". These are classical hypergraph queries, executable with standard graph algorithms.

2. **Semantic.** "Return all rules about disclosure of conflicts". These use HSC re-ranking: structural candidates are re-ranked by Hilbert-space signal.

3. **Conflict discovery.** "Return all conflict hyperedges active in version V". These are first-class entities in the graph, not derived at query time.

4. **Time-travel.** "What was the rule graph as of Date D?". Standard versioned-graph queries.

Each query returns hyperedges, with signed semantics preserved, and with source pointers that trace back to the original document span.

## 6. Performance

Hypergraphs sound expensive. They're not, if you're careful.

We back QHG with a standard columnar store (Parquet + DuckDB for query, PostgreSQL for ACID) plus an auxiliary embedding store for vector operations. Key operations (structural queries, hyperedge lookups, time-travel) run in $O(\log n)$ per hop.

Conflict-discovery queries are the potentially expensive case (pairwise joins over large rule sets). We handle this with a two-phase plan:

1. **Offline phase.** Compute HSC conflict signal over all rule pairs in a batch. Materialise conflict hyperedges.
2. **Online phase.** Conflict queries are O(1) — the conflict hyperedges are already there.

On ~10,000-rule corpora (typical enterprise size) the offline phase runs in 3–5 minutes on a single GPU. On 100,000-rule corpora (big bank territory), 30-40 minutes. Neither of these is a bottleneck.

## 7. Privacy and GDPR

Immutability makes audit easy and deletion hard. Standard GDPR compliance requires the right to erasure. We handle this with a **tombstone** mechanism:

- An erasure request produces a tombstone hyperedge pointing to the to-be-erased node(s).
- The storage pruning for the tombstoned entities runs on a scheduled basis.
- Queries after the tombstone is applied return the tombstone in place of the node, with a "right-to-be-forgotten" marker.
- The graph remains auditable up to the tombstone event.

This is not free — the immutable history up to the tombstone is still retained, which means the erasure is partial. For enterprise compliance use cases, this is generally acceptable; for consumer contexts it requires more care.

## 8. Why this matters for agents

QHG is also a natural shared memory substrate for multi-agent systems. Each agent writes to the graph with typed provenance; contradictions between agents surface as conflict hyperedges. Reconciliation is a policy on conflict hyperedges.

Paper C ([QGI-TR-2026-03]) sketches this in more detail. The property that makes QHG useful for policy — signed, typed, multi-party hyperedges with versioning — is the same property that makes it useful for multi-agent belief reconciliation.

## 9. What we're releasing

- **QHG specification.** In preprint [QGI-TR-2026-05], with full schema, operational semantics, and reference query interface.
- **Reference implementation.** Private for now. Limited-availability source code for evaluation partners.
- **QHG-as-a-service.** Hosted, fully-managed, part of the QAG engine at GA 21 June 2026.

## 10. Access and criticism

- **Preprint:** *Quantum HyperGraph: A First-Class Data Model for Rule-Bearing Knowledge* — arXiv link in the header.
- **Companion preprints:** A, B, C, D, F — full pack on the QAG engine, embedding, agent memory, Born-rule classifier, and landscape review.
- **Specification discussions:** `research@qgi.dev`. Happy to do architecture review calls with teams building their own policy graphs.
- **Comparative benchmarks** against Neo4j, RDF triple-store, Memgraph, HyperNetX: included in the preprint's appendix.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
