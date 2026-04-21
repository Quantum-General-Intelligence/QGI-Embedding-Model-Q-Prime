# QGI Paper Series — release plan

**Author:** Dr. Sam Sammane — Chief Technology Officer and Founder, Quantum General Intelligence, Inc. (`sam@qgi.dev`)
**Status:** ready-to-go corpus, not yet published.
**Destination:** private staging repo (do not push to the current public remote).

## Corpus at a glance

| ID | Title (working) | Length | Audience | Day-0? |
|---|---|---|---|---|
| A | Quantum-Augmented Generation (QAG): A Reasoning-First Memory Infrastructure for AI Agents and Regulated Systems | 22–26 pp | Systems / applied NLP | yes |
| B | Purpose-Built Embedding Models for Rule-Bearing Text: Why General-Purpose Embeddings Score F1 = 0 on Regulatory Conflict | 10–12 pp | Embedding / IR | yes |
| C | Conflict-Aware Memory for AI Agents: A Hilbert-Space Approach to Consolidation and Context Curation | 10–12 pp | Agent frameworks | yes |
| D | A Born-Rule Classifier: Zero-Shot Categorisation via Squared Amplitudes on Class Centroids | 5–7 pp | ML method | yes |
| E | Quantum HyperGraph (QHG): A First-Class Data Model for Rule-Bearing Knowledge | 6–8 pp | KG / DB | yes |
| F | Beyond Retrieval-Augmented Generation: A Review of Reasoning-First Alternatives for Rule-Bearing Content | 12–14 pp | Broad ML/NLP | yes |
| G | *Quantum Signatures of Regulatory Language: A Cross-Backbone, Cross-Extractor Evaluation* | 14–18 pp | Empirical NLP | **no — waits for evaluation-agreement release** |
| — | Executive whitepaper | 8–10 pp | CIO / procurement | yes |

## Folder layout

```
papers/
├── PLAN.md                 — this file
├── README.md               — manifest
├── Makefile                — renders all preprints and the whitepaper
├── common/
│   ├── _metadata.yaml      — shared YAML for every preprint
│   ├── reference.docx      — shared pandoc DOCX style
│   ├── build_reference_docx.py
│   └── figures/            — six shared PNG figures
├── preprints/
│   ├── A-qag-engine/
│   │   └── paper.{md,pdf,docx}
│   ├── B-qprime-embedding-position/
│   ├── C-agent-memory/
│   ├── D-born-rule-classifier/
│   ├── E-quantum-hypergraph/
│   └── F-beyond-rag-survey/
├── whitepaper-executive/
│   └── whitepaper.{md,pdf,docx}
└── blog/
    └── <paper-id>/
        ├── company-blog.md
        ├── linkedin.md
        ├── substack.md
        ├── medium.md
        ├── hackernews.md
        └── twitter.md
```

## Shared conventions

- **Author block** — Dr. Sam Sammane, CTO & Founder, QGI, `sam@qgi.dev`.
- **Typesetting** — shared LaTeX and DOCX styles (see `common/`), five-colour QGI palette.
- **Math** — use `$...$` dollars; Pandoc 3.1 does not reliably route `\(...\)` through math mode in this corpus.
- **Cross-references** — pandoc-style `[§8](#anchor)`, no `pandoc-crossref`.
- **Every preprint** carries a preprint banner, competing-interests note, ethics note, data-availability note, version-history block, and Cite-this-as block.

## Release cadence (recommended)

1. Week 1 — Paper A (canonical).
2. Week 3 — Paper F (landscape review).
3. Week 5 — Paper C (agent memory) — broadens audience.
4. Week 7 — Paper B (Q-Prime position).
5. Week 9 — Papers D + E (paired method/structure notes).
6. When cleared — Paper G (evaluation paper) with its own press cycle.

Executive whitepaper accompanies each preprint release; blog derivatives per paper stagger across 5–7 days inside each release week.

## Known hold-backs

- **No numerical benchmark results** appear in any preprint other than paper G. Every reference to quantitative evaluation is forward-looking ("forthcoming companion paper").
- **No model weights** are distributed; Q-Prime stays managed-API.
- **No QNR2 parser internals** beyond the public-facing semantics.

## Build recipe

```bash
cd papers
make all            # build every PDF and DOCX
make clean          # remove generated artefacts
```

Requires `pandoc` ≥ 3.1, `xelatex`, and a Python venv with `python-docx`, `pypdf`, `cairosvg`.
