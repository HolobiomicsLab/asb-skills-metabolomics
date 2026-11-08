---
name: statistics-biomarker-discovery-workflow
description: 'Use when you have a metabolomics feature/quant table and want a statistically
  rigorous comparison and candidate biomarkers — cleaning and normalization, multivariate
  analysis, differential features, enrichment/pathway analysis, and biomarker selection
  with ROC.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - svr-based-intensity-normalization
  - lc-ms-feature-table-cleaning
  - metabolomics-feature-filtering-and-imputation
  - lipidomic-feature-table-preprocessing
  - pls-da-multivariate-modeling
  - plsda-supervised-classification-metabolomics
  - pca-dimension-reduction-metabolomics
  - metabolomics-principal-component-analysis
  - pairwise-group-comparison-statistics
  - metabolite-univariate-statistical-testing
  - metabolomics-feature-differential-analysis
  - cell-type-differential-abundance-testing
  - metabolomics-peak-differential-analysis
  - mummichog-pathway-enrichment-analysis
  - pathway-enrichment-analysis-fishers-exact-test
  - pathway-enrichment-analysis
  - mummichog-pathway-activity-prediction
  - feature-importance-rank-scoring
  - random-forest-classifier-training--728f93a1a0de83da
  - metabolomics-feature-importance-ranking
  - multi-feature-set-model-aggregation
  member_tools:
  - R
  - masscleaner
  - massdataset
  - tidyMass
  - caret
  - ggplot2
  - lmm2met
  - mixOmics
  - ropls
  - GetFeatistics
  - ggplot
  - MetaboAnalystR 4.0
  - OptiLCMS
  - Galaxy
  - Python
  - SECIMTools
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

# Metabolomics Statistics and Biomarker Discovery

## Summary

End-to-end metabolomics statistics: from a raw feature table to normalized data, multivariate structure, differential features, pathway context, and ranked biomarkers.


## When to use

Use when you have a metabolomics feature/quant table and want a statistically rigorous comparison and candidate biomarkers — cleaning and normalization, multivariate analysis, differential features, enrichment/pathway analysis, and biomarker selection with ROC.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — clean_normalize

**Goal:** clean + normalize the feature table

**EDAM operation:** http://edamontology.org/operation_3435

**Inputs:** feature-table · **Outputs:** feature-table

**Candidate leaf skills:** `svr-based-intensity-normalization` (primary), `lc-ms-feature-table-cleaning`, `metabolomics-feature-filtering-and-imputation`, `lipidomic-feature-table-preprocessing`

**Tools (primary):** R, masscleaner, massdataset, tidyMass

**Other candidate tools:** Galaxy-M, PLS-Toolbox, XCMS, EigenMS, MultiABLER, ProteoMM, devtools, limma

**Grounding:** 3 KB(s); DOIs: 10.1016/j.isci.2023.106881, 10.1038/s41467-022-32155-w, 10.1186/s13742-016-0115-8

### Stage 2 — multivariate

**Goal:** multivariate analysis (PCA / PLS-DA / OPLS-DA, VIP)

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `pls-da-multivariate-modeling` (primary), `plsda-supervised-classification-metabolomics`, `pca-dimension-reduction-metabolomics`, `metabolomics-principal-component-analysis`

**Tools (primary):** R, caret, ggplot2, lmm2met, mixOmics, ropls

**Other candidate tools:** POMA, POMAShiny, plotly, tidyverse, MetaboDiff

**Grounding:** 3 KB(s); DOIs: 10.1016/j.csbj.2019.04.009, 10.1093/bioinformatics/bty344, 10.1371/journal.pcbi.1009148

### Stage 3 — differential

**Goal:** differential feature analysis (univariate, volcano, FDR)

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** feature-table · **Outputs:** tsv

**Candidate leaf skills:** `pairwise-group-comparison-statistics` (primary), `metabolite-univariate-statistical-testing`, `metabolomics-feature-differential-analysis`, `cell-type-differential-abundance-testing`, `metabolomics-peak-differential-analysis`

**Tools (primary):** GetFeatistics, R, ggplot

**Other candidate tools:** dplyr, massdataset, massstat, purrr, tidymass, dbnorm, ggplot2, WaveICA, apLCMS, BiomiX, CEU Mass Mediator, EnrichR, GSEA

**Grounding:** 5 KB(s); DOIs: 10.1038/s41467-022-32155-w, 10.1038/s41598-020-70850-0, 10.1038/s41598-021-84824-3, 10.1186/s12859-024-06022-y …

### Stage 4 — enrichment_pathway

**Goal:** enrichment + pathway / functional analysis

**EDAM operation:** http://edamontology.org/operation_3928

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `mummichog-pathway-enrichment-analysis` (primary), `pathway-enrichment-analysis-fishers-exact-test`, `pathway-enrichment-analysis`, `mummichog-pathway-activity-prediction`

**Tools (primary):** MetaboAnalystR 4.0, OptiLCMS, R

**Other candidate tools:** cloud-mummichog, mummichog, Azimuth DB, metDataModel, MetaboAnalyst 6.0, MetaboAnalystR

**Grounding:** 3 KB(s); DOIs: 10.1038/s41467-024-48009-6, 10.1093/nar/gkae253, 10.1371/journal.pcbi.1003123

### Stage 5 — biomarker

**Goal:** biomarker selection + ROC / importance

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** tsv, tsv · **Outputs:** tsv

**Candidate leaf skills:** `feature-importance-rank-scoring` (primary), `random-forest-classifier-training--728f93a1a0de83da`, `metabolomics-feature-importance-ranking`, `multi-feature-set-model-aggregation`

**Tools (primary):** Galaxy, Python, SECIMTools

**Other candidate tools:** MassCube, MeTEor, R, XGBoost, caret, pROC, scikit-learn

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-025-60640-5, 10.1093/bioadv/vbae178, 10.1101/2024.08.17.608400, 10.1186/s12859-018-2134-1

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
