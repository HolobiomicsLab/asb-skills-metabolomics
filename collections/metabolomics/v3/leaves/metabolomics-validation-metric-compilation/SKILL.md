---
name: metabolomics-validation-metric-compilation
description: Use when reproducing or auditing a published metabolomics tool's validation
  claims from per-compound supplementary tables of LC-MS/MS results, e.g. expected-feature
  detection, SIRIUS formula and CSI:FingerID annotation accuracy.
when_to_use_negative:
- The input is raw vendor data (Thermo raw, mzML) needing preprocessing — run the
  UmetaFlow preprocessing/annotation workflow first; this skill only audits the resulting
  annotated feature tables
- The goal is benchmarking feature detection/quantification against public datasets
  (MTBLS733, MTBLS736) rather than reproducing the in-house per-compound validation
  metrics
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: UmetaFlow
  role: Workflow whose in-house validation output (feature tables with SIRIUS/CSI:FingerID
    annotations and spectral matches) is audited to compute the metrics
  repo: https://github.com/biosustain/snakemake_UmetaFlow
- name: Python
  role: Used with pandas to parse the supplementary tables and compute validation
    fractions
- name: SIRIUS
  role: 'Source of the #1-ranked molecular formula predictions whose accuracy (76%)
    is recomputed from the validation tables'
- name: CSI:FingerID
  role: 'Source of the #1-ranked structural predictions whose accuracy (~62%) is recomputed
    from the validation tables'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-023-00724-w
    title: 'UmetaFlow: an untargeted metabolomics workflow for high-throughput data
      processing and analysis'
  source_content_hash: sha256:4c0b452e641eef3a1d64a8bf942ad2cf12babe95de9a274eda5120fb675239ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biosustain/snakemake_UmetaFlow
  tools:
  - CSI:FingerID
  - Python
  - SIRIUS
  - UmetaFlow
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# metabolomics-validation-metric-compilation

## Summary

A reproducible procedure for computing validation metrics (feature-detection rate, formula-annotation accuracy, structural-annotation accuracy) from per-compound supplementary tables of the UmetaFlow in-house LC-MS/MS validation, using pandas to check computed fractions against paper-reported values (100%, 76%, ~62%, 65%).

## When to use

Use when reproducing or auditing a published metabolomics tool's validation claims from per-compound supplementary tables of LC-MS/MS results, e.g. expected-feature detection, SIRIUS formula and CSI:FingerID annotation accuracy.

## When NOT to use

- The input is raw vendor data (Thermo raw, mzML) needing preprocessing — run the UmetaFlow preprocessing/annotation workflow first; this skill only audits the resulting annotated feature tables
- The goal is benchmarking feature detection/quantification against public datasets (MTBLS733, MTBLS736) rather than reproducing the in-house per-compound validation metrics

## Inputs

- Additional file 2 (Tables S9–S11) supplementary validation tables from the UmetaFlow article (DOI 10.1186/s13321-023-00724-w)
- In-house LC-MS/MS datasets of actinomycete extracts producing known secondary metabolites (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores) and commercial standards
- UmetaFlow feature tables annotated with SIRIUS/CSI:FingerID #1-ranked predictions and spectral-library matches

## Outputs

- validation_summary.csv with computed validation fractions (feature detection, formula accuracy, structural accuracy) and per-metabolite-class breakdowns
- Comparison of computed fractions against paper-reported values (100% features detected, 76% formulas, ~62% CSI:FingerID structures, 65% total structural annotations)

## How to apply

Download the article's supplementary validation tables (Additional files 2–4, Tables S9–S11) accompanying the UmetaFlow article (DOI 10.1186/s13321-023-00724-w), which report per-compound validation of in-house LC-MS/MS data from actinomycete extracts producing known secondary metabolites and commercial standards. Load the tables into Python with pandas and parse the columns covering expected-feature detection, molecular formula annotations, CSI:FingerID structural predictions, and spectral-library matches. Compute the validation fractions: fraction of expected features detected (paper reports 100%), fraction of accurate SIRIUS molecular formulas (76%), fraction of accurate CSI:FingerID structures (~62%), and total accurate structural annotations including spectral matches (65% — spectral matches scoring above the 60 threshold count as MSI level 2, SIRIUS/CSI:FingerID #1-ranked predictions as MSI level 3). Write the computed metrics plus per-metabolite-class breakdowns (germicidins, kirromycin, desferrioxamine-pathway siderophores) to validation_summary.csv, and validate by confirming the fractions match the paper-reported values.

## Related tools

- **UmetaFlow** (Workflow whose in-house validation output (feature tables with SIRIUS/CSI:FingerID annotations and spectral matches) is audited to compute the metrics) — https://github.com/biosustain/snakemake_UmetaFlow
- **Python** (Used with pandas to parse the supplementary tables and compute validation fractions)
- **SIRIUS** (Source of the #1-ranked molecular formula predictions whose accuracy (76%) is recomputed from the validation tables)
- **CSI:FingerID** (Source of the #1-ranked structural predictions whose accuracy (~62%) is recomputed from the validation tables)

## Examples

```
import pandas as pd; tbl = pd.read_csv('Additional_file2_TableS9.tsv', sep='\t'); detection_rate = tbl['expected_feature_detected'].mean(); formula_acc = tbl['formula_correct'].mean(); structure_acc = tbl['csi_fingerid_correct'].mean(); print(detection_rate, formula_acc, structure_acc)
```

## Evaluation signals

- Computed fraction of expected features detected equals 100% on the in-house actinomycete and commercial standards datasets
- Computed fraction of accurate molecular formula annotations equals 76%
- Computed fraction of accurate CSI:FingerID structural predictions is approximately 62%
- Computed total accurate structural annotations (spectral matches scoring above 60 as MSI level 2 plus SIRIUS/CSI:FingerID predictions as MSI level 3) equals 65%
- validation_summary.csv exists and per-metabolite-class breakdowns (e.g. germicidins, kirromycin, desferrioxamine-pathway siderophores) reconcile with the totals

## Limitations

- The per-compound validation numbers live only in the supplementary files (e.g. Table S9), not the main text, so reproduction requires downloading Additional files 2–4
- The article does not state software version numbers for UmetaFlow, OpenMS, SIRIUS, CSI:FingerID, or Snakemake, which may affect reproducibility of the exact validation output
- Structural-annotation accuracy conflates two annotation levels: spectral matches above a score of 60 (MSI level 2) and SIRIUS/CSI:FingerID #1-ranked predictions (MSI level 3)

## Evidence

- [supplementary] On the in-house actinomycete and commercial standards datasets, 100% of all expected features were detected, SIRIUS accurately predicted 76% of all formulas: [evidence span withheld — anchor status: not_found]
- [abstract] UmetaFlow accurately annotated 76% of molecular formulas and 65% of structures: [evidence span withheld — anchor status: not_found]
- [readme] Spectral matching with in-house or a publicly available library: "**Spectral matching** with in-house or a publicly available library (MGF/MSP/mzML format) and annotation of the feature matrix with matches that have a score above 60 (MSI level 2)."
- [abstract] in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites: "in-house LC-MS/MS datasets of actinomycetes producing known secondary metabolites, as well as commercial standards, and it detected all expected features"
- [discussion] Benchmark performance numbers are only in supplementary files, not in main text: [evidence span withheld — anchor status: not_found]
