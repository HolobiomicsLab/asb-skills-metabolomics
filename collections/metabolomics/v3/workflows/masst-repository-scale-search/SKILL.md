---
name: masst-repository-scale-search-workflow
description: 'Use when you have a spectrum or feature of interest and want to know
  where else it occurs across all public metabolomics data — query preparation, repository-scale
  fastMASST search, specialized microbe/plant/food MASST for ecological context, and
  co-occurrence analysis.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 4
  member_skills:
  - spectrum-usi-retrieval-from-online-repositories
  - universal-spectrum-identifier-construction
  - massive-dataset-spectrum-retrieval
  - usi-spectrum-searching
  - fast-search-api-query-execution
  - msms-spectral-batch-search-against-public-repositories
  - mass-spectral-analog-search
  - mass-spectrometry-batch-search
  - msms-spectral-search
  - mass-spectrometry-library-matching--f8a5aa74ab04f373
  - compound-to-taxon-mapping
  - human-sample-context-stratification-of-msms-matches
  - masst-job-result-parsing
  - masst-match-table-parsing
  - massive-public-dataset-spectrum-search-retrieval
  member_tools:
  - GNPS
  - Python
  - spectrum_utils
  - Fast Search API
  - MZmine
  - Python 3.10
  - microbeMASST
  - GNPS_MASST
  - jobs.py (microbe_masst)
  - Metabolights
  - NORMAN
  - metadataMASST
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/metabolomics/v3
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# MASST Repository-Scale Spectral Search (Reverse Metabolomics)

## Summary

End-to-end reverse metabolomics: take one molecule and find its public-dataset footprint with MASST, then add ecological context and co-occurrence interpretation.


## When to use

Use when you have a spectrum or feature of interest and want to know where else it occurs across all public metabolomics data — query preparation, repository-scale fastMASST search, specialized microbe/plant/food MASST for ecological context, and co-occurrence analysis.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — spectrum_prep

**Goal:** prepare a query MS/MS spectrum / USI for search

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mgf · **Outputs:** tsv

**Candidate leaf skills:** `spectrum-usi-retrieval-from-online-repositories` (primary), `universal-spectrum-identifier-construction`, `massive-dataset-spectrum-retrieval`, `usi-spectrum-searching`

**Tools (primary):** GNPS, Python, spectrum_utils

**Other candidate tools:** Metabolomics Spectrum Resolver, USI, GNPS_LCMSDashboard, Universal Spectrum Identifier (USI), mzML format (open MS data format), Fast Search API, MZmine, microbeMASST

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.analchem.9b04884, 10.1038/s41564-023-01575-9, 10.1038/s41592-021-01339-5, 10.1101/2020.05.09.086066

### Stage 2 — masst_search

**Goal:** repository-scale spectral search (fastMASST)

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `fast-search-api-query-execution` (primary), `msms-spectral-batch-search-against-public-repositories`, `mass-spectral-analog-search`, `mass-spectrometry-batch-search`, `msms-spectral-search`

**Tools (primary):** Fast Search API, MZmine, Python 3.10, microbeMASST

**Other candidate tools:** GNPS_MASST, Metabolights, NORMAN, jobs.py (Fast Search API), metadataMASST, Fast Search API (fasst), microbe_masst (jobs.py), GNPS, tissueMASST

**Grounding:** 3 KB(s); DOIs: 10.1038/s41467-025-65993-5, 10.1038/s41564-023-01575-9, 10.1101/2025.04.28.651123

### Stage 3 — specialized_masst  [OPTIONAL]

**Goal:** (optional) ecological context via microbe/plant/food MASST

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `mass-spectrometry-library-matching--f8a5aa74ab04f373` (primary), `compound-to-taxon-mapping`

**Tools (primary):** Fast Search API, GNPS_MASST, MZmine, jobs.py (microbe_masst), microbeMASST

**Other candidate tools:** GNPS libraries

**Grounding:** 1 KB(s); DOIs: 10.1038/s41564-023-01575-9

### Stage 4 — cooccurrence

**Goal:** co-occurrence / reverse-metabolomics interpretation

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `human-sample-context-stratification-of-msms-matches` (primary), `masst-job-result-parsing`, `masst-match-table-parsing`, `massive-public-dataset-spectrum-search-retrieval`

**Tools (primary):** Fast Search API, MZmine, Metabolights, NORMAN, Python 3.10, metadataMASST

**Other candidate tools:** MASST, pandas, Nextflow

**Grounding:** 2 KB(s); DOIs: 10.1038/s41564-023-01575-9, 10.1038/s41586-023-06906-8

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
