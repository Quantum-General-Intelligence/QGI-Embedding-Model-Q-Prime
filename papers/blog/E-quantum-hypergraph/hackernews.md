---
title: "Show HN: Quantum HyperGraph — versioned hypergraph data model for rule-bearing text"
venue: "Hacker News (Show HN)"
canonical: "https://arxiv.org/abs/XXXX.XXXXX"
published: "2026-04-21"
length_words: ~250
---

Policy/regulation/contract/guideline data is inherently multi-party: a single rule involves 5-6 entities with typed roles and a signed obligation that toggles under structured conditions. Property graphs (Neo4j, Neptune, TigerGraph) force pairwise edges, so teams use one of three lossy workarounds: reify as node, decompose into pairwise edges, or store as text blob with metadata-only in graph.

We built Quantum HyperGraph (QHG). Five properties missing from off-the-shelf hypergraph tools:

1. First-class versioning / time-travel ("as of January 1, what was the rule set?").
2. Signed semantics (positive / negative / conditionally-signed hyperedges).
3. Conflict as a first-class hyperedge type (contradictions materialised, not inferred).
4. Source provenance enforced by schema.
5. Tombstone hyperedges for GDPR-style partial deletion.

The "quantum" part: each node has a Hilbert-space state vector; hyperedges correspond to observables; hyperedge probabilities are expectation values (Born rule).

Performance on 10K-rule enterprise corpus: 3-5 min offline conflict discovery, ~5ms median structural query, ~50ms HSC-re-ranked semantic query.

Preprint [QGI-TR-2026-05]: full schema, operational semantics, comparison with Neo4j, RDF, HyperNetX, hypergraphx.

One of 6 preprints released today from QGI. Full pack at [arXiv URL].

Hosted QHG ships as part of QAG engine at GA 21 June 2026.

Architecture discussions: research@qgi.dev. We're especially interested in critiques from the RDF/graph-DB and hypergraph-data-management research communities.
