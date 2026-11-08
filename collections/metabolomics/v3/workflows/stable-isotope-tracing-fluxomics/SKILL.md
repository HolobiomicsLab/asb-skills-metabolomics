---
name: stable-isotope-tracing-fluxomics-workflow
description: 'Use when you have LC-MS data from a stable-isotope (e.g. 13C / 15N)
  tracing experiment and want labelling / flux information — detect features, extract
  per-feature isotopologue distributions, correct for natural isotope abundance, and
  compute mass-isotopomer distributions and fractional labelling enrichment across
  conditions or timepoints.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - isotopologue-peak-quantification
  - isotope-label-tracing-analysis
  - stable-isotope-lc-ms-annotation
  - retention-time-shift-correction-isotope-labelling
  - isotope-hypothesis-enumeration-from-mass-traces
  - isotope-adduct-ion-grouping
  - isotopologue-intensity-forward-calculation
  - stable-isotope-lcms-labeling-data-processing
  - isotopologue-abundance-normalization
  - isotopic-enrichment-calculation
  - stable-isotope-labeling-data-quantification
  - mass-distribution-vector-enrichment-calculation
  - isotopomer-distribution-comparison
  - metabolomics-data-summarization
  - isotopologue-labelling-visualization-of-metabolic-flux
  - metabolomics-html-report-generation
  member_tools:
  - MassSpecWavelet
  - PAFFT
  - TarMet
  - airPLS
  - enviPat
  - shiny
  - FeatureFinderMetabo
  - OpenMS
  - ElemCor
  - MATLAB
  - MDT
  - ULS
  - IsoCor
  coverage_gaps: []
  derived_from_workflows:
  - coll_idsl_ipa_cq
  - coll_corems
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Stable-Isotope Tracing (isotopologue extraction -> labelling analysis)

## Summary

Labelled LC-MS in, a labelling table out: isotopologue extraction, natural-abundance correction, and mass-isotopomer-distribution / enrichment analysis.


## When to use

Use when you have LC-MS data from a stable-isotope (e.g. 13C / 15N) tracing experiment and want labelling / flux information — detect features, extract per-feature isotopologue distributions, correct for natural isotope abundance, and compute mass-isotopomer distributions and fractional labelling enrichment across conditions or timepoints.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw labelled LC-MS -> aligned feature table (all isotopologues)

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table

**Candidate leaf skills:** `isotopologue-peak-quantification` (primary), `isotope-label-tracing-analysis`, `stable-isotope-lc-ms-annotation`, `retention-time-shift-correction-isotope-labelling`

**Tools (primary):** MassSpecWavelet, PAFFT, TarMet, airPLS, enviPat, shiny

**Other candidate tools:** MSnbase, Metaboseek, Metaboseek (LabelFinderModule), xcms, R, X(13)CMS, devtools, geoRge, CAMERA

**Grounding:** 4 KB(s); DOIs: 10.1007/s11306-018-1363-7, 10.1021/ac202450g, 10.1021/acs.analchem.5b03628, 10.1038/s41467-022-28391-9

### Stage 2 — isotopologue_extract

**Goal:** feature table -> per-metabolite isotopologue intensity distributions

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `isotope-hypothesis-enumeration-from-mass-traces` (primary), `isotope-adduct-ion-grouping`, `isotopologue-intensity-forward-calculation`

**Tools (primary):** FeatureFinderMetabo, OpenMS

**Other candidate tools:** Python 3, asari, khipu, mass2chem, metDataModel, jsonpickle, moiety_modeling, numpy, scipy

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.2c05810, 10.1074/mcp.m113.031278, 10.1186/s12859-019-3096-7

### Stage 3 — natural_abundance_correction

**Goal:** correct isotopologue distributions for natural isotope abundance

**EDAM operation:** http://edamontology.org/operation_3435

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `stable-isotope-lcms-labeling-data-processing` (primary), `isotopologue-abundance-normalization`, `isotopic-enrichment-calculation`, `stable-isotope-labeling-data-quantification`

**Tools (primary):** ElemCor, MATLAB, MDT, ULS

**Other candidate tools:** TarMet, enviPat, shiny, scipy.optimize.nnls, R, isoSCAN

**Grounding:** 3 KB(s); DOIs: 10.1007/s11306-018-1363-7, 10.1021/acs.analchem.0c02998, 10.1186/s12859-019-2669-9

### Stage 4 — labelling_analysis

**Goal:** corrected distributions -> mass-isotopomer distribution / fractional enrichment

**EDAM operation:** http://edamontology.org/operation_3799

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `mass-distribution-vector-enrichment-calculation` (primary), `isotopomer-distribution-comparison`

**Tools (primary):** ElemCor, IsoCor, MATLAB, MDT, ULS

**Other candidate tools:** MetaboLabPy, metabolabpytools, numpy, pandas

**Grounding:** 2 KB(s); DOIs: 10.1186/s12859-019-2669-9, 10.3390/metabo15010048

### Stage 5 — report

**Goal:** consolidate labelling / enrichment results into a tracing report table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `metabolomics-data-summarization` (primary), `isotopologue-labelling-visualization-of-metabolic-flux`, `metabolomics-html-report-generation`

**Tools (primary):** TarMet, shiny

**Other candidate tools:** MassSpecWavelet, PAFFT, enviPat, plotly, MetaboReport, R, R Shiny, ggplot2, golem

**Grounding:** 2 KB(s); DOIs: 10.1007/s11306-018-1363-7, 10.1093/bioinformatics/btae373

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
