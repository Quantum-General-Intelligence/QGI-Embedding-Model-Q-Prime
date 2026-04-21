---
title: "QHG: A Hypergraph Data Model for Rule-Bearing Knowledge"
byline: "Dr. Sam Sammane"
venue: "Substack (QGI Research Notes)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~1200
tags: ["hypergraphs", "knowledge graphs", "data model", "compliance", "policy"]
---

This note accompanies preprint [QGI-TR-2026-05], *Quantum HyperGraph: A First-Class Data Model for Rule-Bearing Knowledge*. It presents the graph schema, operational semantics, and comparison with prior art.

## Context: the property-graph failure mode for policy

Policy — regulation, contract, clinical guideline, organisational rule — is intrinsically *multi-party* and *conditionally signed*. The standard property-graph model (Neo4j-style nodes-and-labelled-directed-edges) is intrinsically *pairwise* and *unsigned*.

A paradigmatic example:

> *"A broker-dealer registered with FINRA must disclose material conflicts of interest to the client before executing an equity trade, unless the client has signed a Pro-Customer Waiver under Rule 2113."*

Entities: 6. Roles: subject / object / recipient / trigger / exception / cross-reference. Sign: positive (must) guarded by a condition (unless). This is one rule; it is not a pairwise edge.

The three common workarounds in industry:

1. **Reification** of edges as nodes with attributes. Loses clean graph semantics.
2. **Decomposition** into pairwise edges. Loses rule atomicity.
3. **Opaque text** with metadata-only in the graph. Loses the graph.

All three degrade the representation.

## The formal data model

A QHG is a tuple:

$$
G = (V, E, \tau_V, \tau_E, \rho, \sigma, \iota, \mathrm{ver})
$$

where

- $V$: node set (versioned, append-only).
- $E \subseteq \mathcal{P}(V) \setminus \{\emptyset\}$: hyperedge set (also versioned).
- $\tau_V: V \to T_V$: node type (rule, predicate, entity, jurisdiction, exception, cross-reference).
- $\tau_E: E \to T_E$: hyperedge type (governs, defines, references, conditional_on, conflicts_with, supersedes).
- $\rho: V \times E \to \mathrm{Roles}$: partial role assignment for each node participating in each hyperedge.
- $\sigma: E \to \{+1, -1, \mathrm{cond}\}$: sign, with a conditional guard predicate when $\sigma(e) = \mathrm{cond}$.
- $\iota: V \cup E \to \mathrm{Source}$: provenance pointer to source document + span.
- $\mathrm{ver}: V \cup E \to (\mathbb{Z}_{\geq 0}, \mathbb{Z}_{\geq 0} \cup \{\infty\})$: $(\mathrm{start}, \mathrm{end})$ version range.

The immutability property is encoded in $\mathrm{ver}$: to "edit" a hyperedge, we close its current version range and open a new one.

Query primitives are parametric in version: every query is of the form "at version $v$, compute $Q$". Time-travel is free.

## The quantum layer

Each node $v$ has an associated state vector $|\psi_v\rangle$ in a Hilbert space $\mathcal{H}$. Hyperedges correspond to *observables* on the joint state of participating nodes:

$$
\hat{O}_e : \mathcal{H}^{\otimes |e|} \to \mathbb{R}
$$

For concreteness:

- **`conflicts_with`** is an observable whose expectation is large when the participating nodes (two rules, say) have opposite polarity on overlapping scope.
- **`reinforces`** is the complement.
- **`subsumes`** is an asymmetric observable with a specific structure.

Given the state $|\Psi\rangle = \bigotimes_{v \in e} |\psi_v\rangle$, the hyperedge probability is $\langle\Psi | \hat{O}_e | \Psi\rangle$. This is the Born rule applied to the joint state.

The quantum layer gives us:

- A consistent probability framework over all hyperedge types.
- Expressibility of complex observables as expectation values.
- A natural coupling between the symbolic graph layer and the intelligence layer (HSC).

This is the sense in which QHG is "quantum" — the formalism of the intelligence layer is quantum mechanical, and the graph is structured to support that formalism natively.

## Operational semantics

**Ingestion.** A rule is parsed by the deterministic extractor (QNR2) into an AST. The AST is materialised as (nodes + hyperedges), with version start = current version.

**Query.**

- Structural: "return all rules of type $t$ that reference jurisdiction $j$". Standard hypergraph traversal.
- Semantic: "return all rules semantically related to $q$". HSC re-ranking over structural candidates.
- Conflict: "return all active `conflicts_with` hyperedges in version $v$". O(1) lookup (conflicts are pre-materialised by an offline job).
- Time-travel: "at version $v_0$, return [structural query]". Standard versioned-graph primitive.

**Update.** No in-place edits. An "edit" closes the version range of the existing hyperedge(s) and opens a new version with the updated state.

**Erasure.** Tombstone hyperedges mark nodes as subject to the right-to-be-forgotten. Pruning is scheduled and partial — the audit trail up to the tombstone is preserved.

## Comparison with prior art

| Property | Property graph (Neo4j) | RDF | HyperNetX | QHG |
|---|---|---|---|---|
| Hyperedges | No | No | Yes | Yes |
| Typed roles | Via reification | Yes | Partial | Yes |
| Typed hyperedges | N/A | Indirect | Partial | Yes |
| Signed edges | Via properties | Via subgraphs | No | Yes |
| Versioning | External | External | No | First-class |
| Conflict as first-class | No | No | No | Yes |
| Source provenance | Via properties | Yes | No | First-class |
| Time-travel | External | External | No | First-class |

QHG is not the first hypergraph data model. It is the first, to our knowledge, that treats the six properties above as first-class simultaneously.

## Performance

On a 10K-rule corpus (typical enterprise scale):

- Ingestion: ~2 minutes.
- Structural query: ~5ms median.
- Semantic query (HSC re-rank): ~50ms median (bounded by GPU embedding pass).
- Conflict discovery (offline pass): 3-5 minutes per full pass.
- Conflict lookup (online): ~2ms median.

On a 100K-rule corpus (big bank territory):

- Ingestion: ~15 minutes.
- Structural query: ~15ms median.
- Conflict discovery (offline): 30-40 minutes per full pass.

All measurements on single-GPU, non-distributed storage. Scaling to multi-GPU and distributed is trivial for the offline pass; structural queries benefit from standard graph partitioning.

## Limitations

- **Storage growth.** Immutability means storage grows with every edit. Retention policies are necessary (e.g., prune versions older than 7 years for enterprise audit).
- **Tombstone is partial deletion.** The audit trail up to the tombstone remains. For some consumer GDPR contexts, this is not enough.
- **Hyperedge complexity.** Very high-cardinality hyperedges (>20 participating nodes) are possible but rare; the query cost scales linearly in $|e|$.
- **No automatic schema evolution.** Adding a new hyperedge type is a schema change. We version the schema separately; existing queries unaffected.

## Why "quantum"?

The naming convention follows the QAG stack. The mathematics that couples QHG to the HSC intelligence layer is genuinely quantum (observables, Hilbert-space joint states, Born rule for hyperedge probability). There is no QPU. The execution is classical.

## Access

- Preprint: [arXiv URL].
- Specification: appendix of the preprint.
- Reference implementation: private; limited availability for evaluation partners.
- Hosted service: part of QAG engine, GA 21 June 2026.

Comparisons with specific property-graph implementations, RDF stores, and other hypergraph libraries are in the preprint's experimental section.

---

*Dr. Sam Sammane, CTO and Founder, Quantum General Intelligence, Inc.*
