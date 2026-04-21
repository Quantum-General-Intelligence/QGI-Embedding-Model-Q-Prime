# QGI Paper Series

**Author:** Dr. Sam Sammane, Chief Technology Officer and Founder, Quantum General Intelligence, Inc. — `sam@qgi.dev`
**Status:** publication-ready preprint corpus. Not peer-reviewed.
**Release:** staged to a private repository; see `PLAN.md` for the recommended week-by-week cadence.

This folder is a self-contained release corpus. Six preprints, one executive whitepaper, and six blog-derivative suites share a common typesetting pipeline and a common figure corpus.

## Preprints

| ID | Title | Path | Length |
|---|---|---|---|
| A | Quantum-Augmented Generation (QAG): A Reasoning-First Memory Infrastructure for AI Agents and Regulated Systems | `preprints/A-qag-engine/` | 22–26 pp |
| B | Purpose-Built Embedding Models for Rule-Bearing Text: Why General-Purpose Embeddings Score F1 = 0 on Regulatory Conflict | `preprints/B-qprime-embedding-position/` | 10–12 pp |
| C | Conflict-Aware Memory for AI Agents: A Hilbert-Space Approach to Consolidation and Context Curation | `preprints/C-agent-memory/` | 10–12 pp |
| D | A Born-Rule Classifier: Zero-Shot Categorisation via Squared Amplitudes on Class Centroids | `preprints/D-born-rule-classifier/` | 5–7 pp |
| E | Quantum HyperGraph (QHG): A First-Class Data Model for Rule-Bearing Knowledge | `preprints/E-quantum-hypergraph/` | 6–8 pp |
| F | Beyond Retrieval-Augmented Generation: A Review of Reasoning-First Alternatives for Rule-Bearing Content | `preprints/F-beyond-rag-survey/` | 12–14 pp |

Paper G — the empirical evaluation — is held back until the evaluation-agreement release clears numbers.

## Executive whitepaper

`whitepaper-executive/whitepaper.md` — math-lite CIO / procurement version. Accompanies each preprint release.

## Blog derivatives

For each preprint, `blog/<id>/` contains six files, one per venue:

- `company-blog.md` — canonical long-form.
- `linkedin.md` — executive / LinkedIn Pulse.
- `substack.md` — researcher-oriented.
- `medium.md` — developer / general-ML.
- `hackernews.md` — short launch post + link.
- `twitter.md` — 10–14-tweet thread.

Each venue uses a distinct headline and lead angle; see the `qgi-paper-series` skill for the SEO / canonical-tag policy.

## How to build

Requires `pandoc` ≥ 3.1, `xelatex`, and a Python venv with `python-docx` and `pypdf`. From the repository root:

```bash
make -C papers all          # render every preprint and whitepaper to PDF + DOCX
make -C papers reference    # regenerate common/reference.docx
make -C papers clean        # remove generated artefacts
```

## Folder layout

```
papers/
├── PLAN.md                         release plan and cadence
├── README.md                       this manifest
├── Makefile                        build every PDF and DOCX
├── common/
│   ├── _metadata.yaml              shared YAML (author, paperdize, TOC)
│   ├── header.tex                  shared LaTeX preamble (colors, fonts, headers)
│   ├── reference.docx              shared pandoc DOCX style
│   ├── build_reference_docx.py     regenerates reference.docx
│   └── figures/                    shared PNG figures
├── preprints/
│   ├── A-qag-engine/paper.{md,pdf,docx}
│   ├── B-qprime-embedding-position/paper.{md,pdf,docx}
│   ├── C-agent-memory/paper.{md,pdf,docx}
│   ├── D-born-rule-classifier/paper.{md,pdf,docx}
│   ├── E-quantum-hypergraph/paper.{md,pdf,docx}
│   └── F-beyond-rag-survey/paper.{md,pdf,docx}
├── whitepaper-executive/whitepaper.{md,pdf,docx}
└── blog/
    ├── A-qag-engine/{company-blog,linkedin,substack,medium,hackernews,twitter}.md
    ├── B-qprime-embedding-position/…
    ├── C-agent-memory/…
    ├── D-born-rule-classifier/…
    ├── E-quantum-hypergraph/…
    └── F-beyond-rag-survey/…
```

## Hold-backs

- No numerical benchmark results are published in any preprint other than paper G.
- Q-Prime weights are not distributed; the model is a managed API.
- QNR2 parser internals are not published beyond public-facing semantics.

## Competing interests

The author is an employee and shareholder of Quantum General Intelligence, Inc., and has a commercial interest in Q-Prime and the QAG engine.
