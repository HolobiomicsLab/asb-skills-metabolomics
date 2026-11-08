---
name: lipidomics-lcms-annotation-workflow
description: 'Use when you have untargeted lipidomics LC-MS/MS data (mzML) and want
  a class- and species-level annotated lipid feature table — preprocessing, normalization,
  lipid identification by MS/MS, retention/adduct rule validation, differential analysis,
  and a fused master table.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 6
  member_skills:
  - snakemake-workflow-rule-execution
  - targeted-lipidomics-transition-list-processing
  - lipidomics-peak-table-filtering
  - lipidomics-batch-effect-correction
  - mass-spectra-preprocessing
  - combat-batch-effect-correction
  - lc-ms-feature-table-batch-correction
  - qc-sample-based-feature-normalization
  - batch-integration-of-ms-omics-data
  - lipidblast-in-silico-spectral-matching
  - lipid-fragment-library-matching
  - rule-based-lipid-identification
  - ms2-ms3-fragment-peak-annotation
  - lipid-feature-qc-filtering
  - lipid-class-adduct-filtering
  - lipidomics-qc-filtering
  - adduct-aware-lipid-library-querying
  - between-group-fold-change-analysis
  - lipidomics-multivariate-statistical-analysis
  - metabolomics-feature-differential-analysis
  - scms-lipidomics-fold-change-analysis
  - lipidomics-annotation-appending
  - multi-dataset-annotation-aggregation
  - feature-table-assembly
  - cross-software-annotation-table-reconciliation
  member_tools:
  - MS2Query
  - OpenMS
  - SIRIUS
  - Snakemake
  - ThermoRawFileParser
  - UmetaFlow
  - pyOpenMS
  - ComBat (sva)
  - R
  - apLCMS
  - CAMERA
  - GNPS
  - LipidBlast
  - METLIN
  - MS-DIAL
  - MetaboAnnotatoR
  - RAMClustR
  - Compound Discoverer
  - LipiDex 2
  - LipiDex 2 Degreaser
  - LipiDex 2 Peak Finder
  - MetaboReport
  - LipidMatch
  - MSConvert
  - MZmine
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

# Lipidomics LC-MS/MS Annotation

## Summary

End-to-end lipidomics annotation: raw LC-MS/MS in, a confidence-graded lipid table out, with lipid-class-aware identification, normalization, and group-wise statistics.


## When to use

Use when you have untargeted lipidomics LC-MS/MS data (mzML) and want a class- and species-level annotated lipid feature table — preprocessing, normalization, lipid identification by MS/MS, retention/adduct rule validation, differential analysis, and a fused master table.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — preprocess

**Goal:** raw lipidomics mzML -> aligned feature table + MS/MS export

**EDAM operation:** http://edamontology.org/operation_3215

**Inputs:** mzML · **Outputs:** feature-table, mgf

**Candidate leaf skills:** `snakemake-workflow-rule-execution` (primary), `targeted-lipidomics-transition-list-processing`, `lipidomics-peak-table-filtering`, `lipidomics-batch-effect-correction`, `mass-spectra-preprocessing`

**Tools (primary):** MS2Query, OpenMS, SIRIUS, Snakemake, ThermoRawFileParser, UmetaFlow, pyOpenMS

**Other candidate tools:** LipidCreator, MSStats, Skyline, Compound Discoverer, LipiDex 2, LipiDex 2 Degreaser, LipiDex 2 Spectrum Searcher, ADViSELipidomics, LIQUID, LipidSearch, Shiny, LipiDetective, h5py

**Grounding:** 5 KB(s); DOIs: 10.1021/acs.analchem.4c00359, 10.1038/s41467-020-15960-z, 10.1093/bioinformatics/btac706, 10.1101/2024.10.07.617094 …

### Stage 2 — normalize

**Goal:** normalize + batch-correct the lipid feature table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** feature-table · **Outputs:** feature-table

**Candidate leaf skills:** `combat-batch-effect-correction` (primary), `lc-ms-feature-table-batch-correction`, `qc-sample-based-feature-normalization`, `batch-integration-of-ms-omics-data`

**Tools (primary):** ComBat (sva), R, apLCMS

**Other candidate tools:** Python, Python 3, pandas, pcpfm, pyComBat, ProteoWizard, Skyline, XCMS, random forest, statTarget

**Grounding:** 3 KB(s); DOIs: 10.1016/j.aca.2018.08.002, 10.1038/s41598-020-70850-0, 10.1371/journal.pcbi.1011912

### Stage 3 — lipid_identification

**Goal:** identify lipids (class + species) from MS/MS fragmentation

**EDAM operation:** http://edamontology.org/operation_3803

**Inputs:** mgf · **Outputs:** tsv

**Candidate leaf skills:** `lipidblast-in-silico-spectral-matching` (primary), `lipid-fragment-library-matching`, `rule-based-lipid-identification`, `ms2-ms3-fragment-peak-annotation`

**Tools (primary):** CAMERA, GNPS, LipidBlast, METLIN, MS-DIAL, MetaboAnnotatoR, R, RAMClustR

**Other candidate tools:** xcms, LipidMatch, LipidMatch (GarrettLab-UF GitHub), MSConvert, LipiDex 2, LipiDex 2 Lipid Spectral Annotator, Spectrum Searcher

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.1c03032, 10.1021/acs.analchem.4c00359, 10.1186/s12859-017-1744-3

### Stage 4 — rule_validation

**Goal:** validate lipid annotations by adduct / retention-time / class rules

**EDAM operation:** http://edamontology.org/operation_3695

**Inputs:** tsv · **Outputs:** tsv

**Candidate leaf skills:** `lipid-feature-qc-filtering` (primary), `lipid-class-adduct-filtering`, `lipidomics-qc-filtering`, `adduct-aware-lipid-library-querying`

**Tools (primary):** Compound Discoverer, LipiDex 2, LipiDex 2 Degreaser, LipiDex 2 Peak Finder

**Other candidate tools:** MobiLipid, R, R Markdown, RStudio, LipiDex 2 Degreaser QC module, Metaboanalyst, LipidMatch, MZmine

**Grounding:** 3 KB(s); DOIs: 10.1021/acs.analchem.4c00359, 10.1021/acs.analchem.4c01253, 10.1186/s12859-017-1744-3

### Stage 5 — statistics

**Goal:** differential lipid analysis between sample groups

**EDAM operation:** http://edamontology.org/operation_3659

**Inputs:** feature-table, tsv · **Outputs:** tsv

**Candidate leaf skills:** `between-group-fold-change-analysis` (primary), `lipidomics-multivariate-statistical-analysis`, `metabolomics-feature-differential-analysis`, `scms-lipidomics-fold-change-analysis`

**Tools (primary):** MetaboReport, R

**Other candidate tools:** LiPydomics, Python, SciPy, scikit-learn, dbnorm, ggplot2, Geena2, HMDB, METLIN, MetaPhenotype, MetaboAnalyst 6.0

**Grounding:** 4 KB(s); DOIs: 10.1021/acs.analchem.0c02560, 10.1021/acs.analchem.4c02038, 10.1038/s41598-021-84824-3, 10.1093/bioinformatics/btae373

### Stage 6 — fusion

**Goal:** consolidate lipid annotations + stats into one master table

**EDAM operation:** http://edamontology.org/operation_3434

**Inputs:** feature-table, tsv · **Outputs:** tsv

**Candidate leaf skills:** `lipidomics-annotation-appending` (primary), `multi-dataset-annotation-aggregation`, `feature-table-assembly`, `cross-software-annotation-table-reconciliation`

**Tools (primary):** LipidMatch, MSConvert, MZmine, R

**Other candidate tools:** LipidIN, LipidIN EQ module, LipidIN LCI module, LipidIN WMYn module, MS-DIAL, CAMERA, MultiABLER, xcms, CombineSoftwareIDs.R, Greazy

**Grounding:** 3 KB(s); DOIs: 10.1016/j.isci.2023.106881, 10.1038/s41467-025-59683-5, 10.1186/s12859-017-1744-3

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.
