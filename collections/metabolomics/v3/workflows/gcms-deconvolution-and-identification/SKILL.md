---
name: gcms-deconvolution-identification-workflow
description: 'Use when you have GC-MS data (mzML / CDF, typically EI) and want deconvolved,
  retention-index-validated compound identifications — spectral deconvolution of co-eluting
  peaks, EI library matching, RI calibration, and differential analysis.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - gc-ms-spectral-deconvolution
  - gcms-spectral-deconvolution
  - dda-spectra-deconvolution
  - eic-peak-shape-correlation-deconvolution
  - gcms-spectral-library-matching
  - ei-mass-spectrum-similarity-scoring
  - molecular-fingerprint-prediction-from-spectra
  - ei-mass-spectrum-library-matching
  - retention-index-based-peak-refinement
  - retention-index-constrained-compound-identification
  - nalkylpyridinium-sulfonate-ri-calibration
  - retention-index-database-matching
  - metabolomics-univariate-group-comparison
  - univariate-differential-feature-testing
  - cell-type-differential-abundance-testing
  - metabolomics-feature-statistical-testing
  - gcms-quantification-table-generation
  - gcms-deconvolution-feature-scoring
  - gc-ms-quantitative-table-matching
  - cross-sample-feature-grouping
  member_tools:
  - GNPS
  - MSHub
  - pyMolNetEnhancer
  - GcDUO
  - R
  - duoID
  - getMSP
  - getRI
  - id_rev
  - libID
  - rt2ri
  - spectramatrix
  - POMA
  - POMAShiny
  - limma
  - tidyverse
  - erah
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

# GC-MS Deconvolution and Identification

## Summary

End-to-end GC-MS annotation: deconvolve co-eluting EI spectra, match to GC-MS libraries with retention-index support, and compare groups.


## When to use

Use when you have GC-MS data (mzML / CDF, typically EI) and want deconvolved, retention-index-validated compound identifications — spectral deconvolution of co-eluting peaks, EI library matching, RI calibration, and differential analysis.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — deconvolution

**Goal:** GC-MS EI spectral deconvolution + peak detection

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf

**Candidate leaf skills:** `gc-ms-spectral-deconvolution` (primary), `gcms-spectral-deconvolution`, `dda-spectra-deconvolution`, `eic-peak-shape-correlation-deconvolution`

**Tools (primary):** GNPS, MSHub, pyMolNetEnhancer

**Other candidate tools:** ChromaTOF, GC-TOF MS, MetAlign, R, XCMS, erah, gcspikelite, osd, MS-DIAL, MS-FINDER, MZmine, MetaboAnalystR 4.0, OptiLCMS, ProteoWizard, SIRIUS, MetaboAnnotatoR, RAMClustR

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.analchem.1c03032, 10.1021/acs.analchem.6b02927, 10.1038/s41467-024-48009-6, 10.1038/s41587-020-0700-3

### Stage 2 — ei_library_match

**Goal:** identify compounds by EI spectral library matching

**EDAM operation:** http://edamontology.org/operation_3631

**Inputs:** mgf · **Outputs:** tsv

**Candidate leaf skills:** `gcms-spectral-library-matching` (primary), `ei-mass-spectrum-similarity-scoring`, `molecular-fingerprint-prediction-from-spectra`, `ei-mass-spectrum-library-matching`

**Tools (primary):** GNPS, MSHub

**Other candidate tools:** DeepEI, NEIMS, PS2MS, python, rdkit, FastEI, gensim, hnswlib

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.3c05019, 10.1038/s41467-023-39279-7, 10.1038/s41587-020-0700-3

### Stage 3 — retention_index

**Goal:** retention index calibration + RI-filtered identifications

**EDAM operation:** http://edamontology.org/operation_3695

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `retention-index-based-peak-refinement` (primary), `retention-index-constrained-compound-identification`, `nalkylpyridinium-sulfonate-ri-calibration`, `retention-index-database-matching`

**Tools (primary):** GcDUO, R, duoID, getMSP, getRI, id_rev, libID, rt2ri, spectramatrix

**Other candidate tools:** MetaboAnnotation, MetaboAnnotationTutorials, MetaboCoreUtils

**Grounding:** 2 KB(s); DOIs: 10.1093/bib/bbaf080, 10.3390/metabo12020173

### Stage 4 — statistics

**Goal:** differential GC-MS feature analysis between groups

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** feature-table, tsv · **Outputs:** tsv

**Candidate leaf skills:** `metabolomics-univariate-group-comparison` (primary), `univariate-differential-feature-testing`, `cell-type-differential-abundance-testing`, `metabolomics-feature-statistical-testing`

**Tools (primary):** POMA, POMAShiny, R, limma, tidyverse

**Other candidate tools:** ProteoWizard, Skyline, XCMS, statTarget, WaveICA, apLCMS

**Grounding:** 3 KB(s); DOIs: 10.1016/j.aca.2018.08.002, 10.1038/s41598-020-70850-0, 10.1371/journal.pcbi.1009148

### Stage 5 — fusion

**Goal:** consolidate GC-MS identifications + stats into a master table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** feature-table, tsv · **Outputs:** tsv

**Candidate leaf skills:** `gcms-quantification-table-generation` (primary), `gcms-deconvolution-feature-scoring`, `gc-ms-quantitative-table-matching`, `cross-sample-feature-grouping`

**Tools (primary):** R, erah

**Other candidate tools:** GNPS, MSHub, GC-TOF MS, MetAlign, XCMS, GcDUO, PARAFAC, multiway, rrcov3way

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.6b02927, 10.1038/s41587-020-0700-3, 10.1093/bib/bbaf080

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
