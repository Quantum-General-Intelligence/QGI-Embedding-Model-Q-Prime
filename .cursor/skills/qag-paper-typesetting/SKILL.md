---
name: qag-paper-typesetting
description: >-
  Project-local skill for building the QAG technical white paper
  (paper/QAG-Technical-Paper.md) into a professionally typeset PDF
  and DOCX. Use whenever the paper text changes, the README changes
  (to keep paper in sync with the canonical vocabulary), or the figures
  are regenerated. Wraps the official anthropics/skills `docx` and
  `pdf` skills, adds a pandoc/xelatex build, a reference.docx for DOCX
  styling, and the repo-specific decisions on fonts, margins, headers,
  figure captions, table styling, and front matter.
license: Same as repository (QGI Commercial Model License v1.0 for content; Proprietary for upstream skills)
---

# QAG paper typesetting guide

## What this skill is for

The repository ships a public technical white paper describing the QGI
**QAG engine** and making the case for the **Q-Prime** embedding model.
The paper is authored once in `paper/QAG-Technical-Paper.md` (pandoc
markdown) and rendered to two artifacts:

- `paper/QAG-Technical-Paper.pdf` --- built with **pandoc + xelatex**
- `paper/QAG-Technical-Paper.docx` --- built with **pandoc --reference-doc=paper/reference.docx**

Use this skill whenever you need to rebuild the paper, tweak its
typography, or keep it aligned with the README.

## Upstream skills used

| Upstream skill | Location in this repo | Used for |
|---|---|---|
| `anthropics/skills/docx` | `.cursor/skills/docx/` | DOCX reference styling, XML unpack/repack, TOC, hyperlinks, tables. |
| `anthropics/skills/pdf`  | `.cursor/skills/pdf/`  | PDF validation (page count, metadata), post-render inspection, text extraction for QA. |

If you need operations that aren't covered below (tracked changes,
redlining, fill-in forms) --- read the upstream `SKILL.md` for the
relevant skill first.

## Toolchain expected

- `pandoc` 3.1+
- TeX Live with `xelatex`, `microtype`, `fancyhdr`, `booktabs`,
  `titlesec`, `geometry`, `hyperref`, `caption`
- `cairosvg` (inside a Python venv) --- to rasterise the source
  SVG diagrams to PNG. The project convention is a 2400-px-wide
  master PNG per figure.
- `pypdf` --- for quick validation of the output PDF.
- `python-docx` --- for programmatically building the DOCX style
  reference (`paper/reference.docx`).

The one-shot venv the build currently uses:

```bash
python3 -m venv /tmp/qgi-paper-venv
/tmp/qgi-paper-venv/bin/pip install --quiet cairosvg pillow python-docx pypdf
```

## Build recipe

1. **Sync source → PNG figures** (only if SVGs changed):

   ```python
   # /tmp/qgi-paper-venv/bin/python
   import cairosvg, os
   PAIRS = [
       ("13-rag-vs-qhp.svg",           "fig-01-rag-vs-qhp.png"),
       ("09-why-hypergraph.svg",       "fig-02-why-hypergraph.png"),
       ("08-hilbert-space-signals.svg","fig-03-hilbert-space-signals.png"),
       ("10-hybrid-search.svg",        "fig-04-hybrid-search.png"),
       ("14-trust-and-audit.svg",      "fig-05-trust-and-audit.png"),
       ("15-enterprise-use-cases.svg", "fig-06-enterprise-use-cases.png"),
   ]
   for svg, png in PAIRS:
       cairosvg.svg2png(url=svg, write_to=f"paper/figures/{png}", output_width=2400)
   ```

2. **Render PDF** via pandoc → xelatex:

   ```bash
   cd paper
   pandoc QAG-Technical-Paper.md \
     -o QAG-Technical-Paper.pdf \
     --pdf-engine=xelatex \
     --resource-path=.:figures
   ```

   All LaTeX / font / preamble configuration is carried by the YAML
   front matter of the markdown --- do not duplicate it in a separate
   template unless the front matter grows beyond ~40 lines.

3. **Render DOCX** via pandoc + reference doc:

   ```bash
   cd paper
   pandoc QAG-Technical-Paper.md \
     -o QAG-Technical-Paper.docx \
     --reference-doc=reference.docx \
     --toc --toc-depth=2 \
     --resource-path=.:figures
   ```

4. **Validate**:

   ```python
   from pypdf import PdfReader
   r = PdfReader("paper/QAG-Technical-Paper.pdf")
   assert len(r.pages) >= 15          # target 15-22 pages
   assert r.metadata.title             # front-matter metadata bound
   ```

## PDF typography conventions (xelatex)

Carried as `header-includes` in the YAML front matter. Key decisions:

| Item | Choice | Why |
|---|---|---|
| Paper | A4 | Default for international audience; QGI is a DE/US corp. |
| Margins | 2.2 cm all around | Readable measure (~75 char/line at 11pt serif). |
| Body font | TeX Gyre Pagella (Palatino clone) | Warm, professional, excellent for long-form technical prose. |
| Sans font | Fira Sans | Clean, modern, pairs well with Pagella. |
| Mono font | Fira Mono / DejaVu Sans Mono | Distinct from sans; legible at small sizes. |
| Body size | 11 pt | Balance of density and readability. |
| Microtype | Enabled | Character protrusion + font expansion; visibly better justification. |
| Section headings | `titlesec`, sans-serif, bold, slightly larger | Visual hierarchy. |
| Running header | `fancyhdr` --- left: engine name; right: version | Professional. |
| TOC | on, depth 2 | Enables reader scanning without being exhaustive. |
| Links | `colorlinks=true`, RoyalBlue | Subtle, not neon blue. |
| Figures | `\linewidth`, centered, caption below | Standard for tech papers. |
| Figure captions | `caption` package, bold label, justified body | Clearer than default. |
| Tables | `booktabs` (top/mid/bottom rules, no vertical rules) | Modern, readable. |
| Listings | None (no code blocks need highlighting in this paper). | |

### Known-good header-includes block

```yaml
header-includes:
  - \usepackage{microtype}
  - \usepackage{booktabs}
  - \usepackage{titlesec}
  - \usepackage{fancyhdr}
  - \usepackage{caption}
  - \usepackage{xcolor}
  - \definecolor{qgiblue}{HTML}{2E4A8F}
  - \pagestyle{fancy}
  - \fancyhf{}
  - \fancyhead[L]{\small\textsc{QGI --- QAG Engine}}
  - \fancyhead[R]{\small Public Preview 1.0}
  - \fancyfoot[C]{\small\thepage}
  - \renewcommand{\headrulewidth}{0.3pt}
  - \titleformat{\section}{\Large\bfseries\color{qgiblue}\sffamily}{\thesection}{0.6em}{}
  - \titleformat{\subsection}{\large\bfseries\sffamily}{\thesubsection}{0.5em}{}
  - \titlespacing*{\section}{0pt}{1.4em}{0.6em}
  - \titlespacing*{\subsection}{0pt}{1.0em}{0.5em}
  - \captionsetup{font=small,labelfont=bf,labelsep=endash}
  - \hypersetup{pdfborderstyle={/S/U/W 1},linkcolor=qgiblue,urlcolor=qgiblue,citecolor=qgiblue}
```

The `qgiblue` definition is the only color applied outside of link
styling --- keep the page a calm cream/white to suit compliance
audiences.

## DOCX styling conventions (reference.docx)

The DOCX render depends on `paper/reference.docx`, which declares:

- **Page**: A4, 2 cm margins all around.
- **Body**: `Normal` = Calibri 11pt (falls back to Arial on systems
  without Calibri).
- **Headings**: `Heading 1..3` = Calibri 16/14/12, bold, dark blue
  (`#2E4A8F` to match the PDF's `qgiblue`).
- **Code / monospace**: Consolas 10pt.
- **Hyperlink**: underlined, `#2E4A8F`.
- **Caption**: italic 10pt, centered.
- **TOC1, TOC2**: inherit from Normal, bold for TOC1, spacing tight.
- **Table Grid**: thin 0.5pt borders, no shading.

`reference.docx` is regenerated whenever the styles above change. The
generator script is committed at `paper/build_reference_docx.py` and
uses `python-docx`. The rules below mirror the upstream `docx` skill
(see `.cursor/skills/docx/SKILL.md`):

- **Always set page size explicitly** (python-docx defaults to Letter;
  this project uses A4 = 11906 x 16838 DXA twips).
- **Never write unicode bullets** --- let pandoc's List Bullet /
  List Number styles do the work.
- **Tables**: when pandoc generates a table, our reference's `Table
  Grid` style is applied automatically. Do not hand-edit cell widths
  after the fact; adjust column widths in markdown instead.

## Keeping the paper in sync with the README

The README defines canonical vocabulary for the engine:

- Engine components: *Memory, Ingestion, Intelligence Signal,
  Validation, Trace*.
- Intelligence signals: *Relevance, Overlap, Conflict, Redundancy,
  Predicate*.
- Graph terminology: *QHG, Node, Hyperedge, Dependency Edge, Conflict
  Edge, Graph Builder*.
- Rule terminology: *QNR2, Rule, CNL, DSL, AST*.
- Boundaries: *Execution and Decision are NOT in this repo; Signal IS
  in this repo*.

When the README is edited, the paper's §2 (*The QAG engine in one
picture*), §4 (*Hilbert Space Compacting*), §5 (*The seven intelligence
signals*), and the Glossary must be reviewed for drift. The paper may
elaborate beyond the README but must not **contradict** it.

Checklist before committing a re-render:

1. [ ] Every signal name in the README appears in the paper in the
   same form (casing and hyphenation).
2. [ ] Every term in the README's *Terminology* section has an entry
   in the paper's Glossary, or an in-text definition with the same
   expansion.
3. [ ] The *Boundaries* section is reflected in the paper's §7
   (*Trust, determinism, and the audit trail*) discussion of
   advisory-only signals.
4. [ ] The Contact section in the paper uses `contact@qgi.dev` for
   public inquiries; private routing to `sam@qgi.dev` is only for the
   README's Contact table.
5. [ ] `pypdf` page count ≥ 15.
6. [ ] DOCX opens in LibreOffice/Word without "repair" prompt.

## Troubleshooting

- **xelatex "Package inputenc Error: Unicode character..."** --- the
  paper contains a character outside the current font. Either add it
  to the font's character range or use `\newunicodechar{…}{…}`.
- **Pandoc "image not found"** --- ensure `--resource-path=.:figures`
  is passed.
- **DOCX figures missing** --- the reference doc may not include
  `Image` paragraph style. Remove `--reference-doc` temporarily to
  confirm, then add the style to `reference.docx`.
- **PDF table overfull** --- long cell content; switch the table to a
  `longtable` or reduce column text.
- **TOC stale** --- pandoc builds the TOC in a single pass. Re-run
  `pandoc` a second time if page numbers in the TOC look wrong.
