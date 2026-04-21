# QGI Paper Series — Makefile
#
# Renders every preprint and the executive whitepaper to PDF (via xelatex)
# and DOCX (via a shared reference document). Figures live in common/figures/
# and each paper.md references them relative to its own folder.
#
# Usage (from repository root):
#   make -C papers all
#   make -C papers preprints
#   make -C papers whitepaper
#   make -C papers reference    (regenerates common/reference.docx)
#   make -C papers clean

SHELL := /bin/bash

PANDOC       ?= pandoc
XELATEX      ?= xelatex
VENV_PY      ?= /tmp/qgi-paper-venv/bin/python
METADATA     := $(abspath common/_metadata.yaml)
HEADER       := $(abspath common/header.tex)
REFERENCE    := $(abspath common/reference.docx)
FIGDIR       := $(abspath common/figures)

PREPRINTS := \
  preprints/A-qag-engine/paper \
  preprints/B-qprime-embedding-position/paper \
  preprints/C-agent-memory/paper \
  preprints/D-born-rule-classifier/paper \
  preprints/E-quantum-hypergraph/paper \
  preprints/F-beyond-rag-survey/paper

WHITEPAPER := whitepaper-executive/whitepaper

PDFS  := $(addsuffix .pdf,  $(PREPRINTS) $(WHITEPAPER))
DOCXS := $(addsuffix .docx, $(PREPRINTS) $(WHITEPAPER))

PANDOC_PDF_FLAGS := \
  --pdf-engine=xelatex \
  --metadata-file=$(METADATA) \
  --include-in-header=$(HEADER) \
  --resource-path=.:$(FIGDIR):common:common/figures

PANDOC_DOCX_FLAGS := \
  --reference-doc=$(REFERENCE) \
  --metadata-file=$(METADATA) \
  --toc --toc-depth=2 \
  --resource-path=.:$(FIGDIR):common:common/figures

.PHONY: all preprints whitepaper reference clean figures check

all: preprints whitepaper

preprints: $(addsuffix .pdf, $(PREPRINTS)) $(addsuffix .docx, $(PREPRINTS))

whitepaper: $(WHITEPAPER).pdf $(WHITEPAPER).docx

reference: $(REFERENCE)

$(REFERENCE): common/build_reference_docx.py
	$(VENV_PY) common/build_reference_docx.py $@

%.pdf: %.md $(HEADER) $(METADATA)
	@echo "==> Building $@"
	cd $(dir $<) && $(PANDOC) $(notdir $<) -o $(notdir $@) $(PANDOC_PDF_FLAGS)

%.docx: %.md $(REFERENCE) $(METADATA)
	@echo "==> Building $@"
	cd $(dir $<) && $(PANDOC) $(notdir $<) -o $(notdir $@) $(PANDOC_DOCX_FLAGS)

figures:
	@echo "Figures live in $(FIGDIR); regenerate from project SVGs if needed."

check:
	@echo "Preprints:"; ls -lh $(addsuffix .pdf, $(PREPRINTS)) 2>/dev/null || true
	@echo; echo "Whitepaper:"; ls -lh $(WHITEPAPER).pdf 2>/dev/null || true

clean:
	rm -f $(PDFS) $(DOCXS)
