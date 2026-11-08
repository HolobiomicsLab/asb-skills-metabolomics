---
name: technical-variation-threshold-filtering
description: Use when QC replicate samples (e.g. pooled QC1 samples) exist in a 1H-NMR
  metabonomic dataset and you must quantify feature-level technical variation or decide
  which metabolic features to retain for downstream MWAS analysis.
when_to_use_negative:
- The dataset lacks QC replicate samples, so CV across replicates cannot be computed
- The input is already a filtered feature table and the goal is phenotype association
  modeling (partial correlations, GLMs), not QC assessment
- Assessing biological rather than technical variation, e.g. metabolite-phenotype
  associations adjusted for confounders
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MWASTools
  role: Provides the QC_CV and QC_CV_specNMR functions to compute per-feature CV across
    QC samples and spectral reproducibility, plus the example metabo_SE dataset
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which the MWASTools QC functions are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:00a21344dec2091ed52ab7464543a17167b41f983b59358ca9b034febf836bed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# technical-variation-threshold-filtering

## Summary

Assesses analytical reproducibility of metabolomic features by computing per-feature coefficient of variation (CV) across QC replicate samples and filtering with FDA-established thresholds of 0.30 (biomarker discovery) and 0.15 (quantification).

## When to use

Use when QC replicate samples (e.g. pooled QC1 samples) exist in a 1H-NMR metabonomic dataset and you must quantify feature-level technical variation or decide which metabolic features to retain for downstream MWAS analysis.

## When NOT to use

- The dataset lacks QC replicate samples, so CV across replicates cannot be computed
- The input is already a filtered feature table and the goal is phenotype association modeling (partial correlations, GLMs), not QC assessment
- Assessing biological rather than technical variation, e.g. metabolite-phenotype associations adjusted for confounders

## Inputs

- SummarizedExperiment object containing metabolic feature intensities and QC replicate samples (e.g. metabo_SE from MWASTools)
- Reference QC sample identifier for spectral reproducibility check (e.g. 'QC1')

## Outputs

- Per-feature CV table across QC samples (e.g. cv_table.csv with a CV value for every metabolic feature)
- Fraction of features with CV < 0.30 and fraction with CV < 0.15
- Optional CV histogram (QC_CV) and spectral reproducibility plot (QC_CV_specNMR)

## How to apply

Load the feature-level SummarizedExperiment (e.g. the metabo_SE example 1H-NMR plasma dataset shipped with MWASTools) and run QC_CV with plot_hist=FALSE to compute the CV of every metabolic feature across QC replicate samples. Optionally run QC_CV_specNMR with ref_sample='QC1' to check whole-spectrum reproducibility against a reference QC sample. Extract per-feature CV values and compute the fraction passing CV < 0.30 (biomarker discovery threshold) and CV < 0.15 (quantification threshold); features above the discovery threshold are candidates for CV-based filtering. In the FGENTCARD plasma case-study, 99% of features had CV < 0.30 and 92% had CV < 0.15, confirming dataset reproducibility and justifying filtering at 0.30.

## Related tools

- **MWASTools** (Provides the QC_CV and QC_CV_specNMR functions to compute per-feature CV across QC samples and spectral reproducibility, plus the example metabo_SE dataset) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which the MWASTools QC functions are executed)

## Examples

```
Rscript -e 'library(MWASTools); data(metabo_SE); cv <- QC_CV(metabo_SE, plot_hist=FALSE); QC_CV_specNMR(metabo_SE, ref_sample="QC1")'
```

## Evaluation signals

- The computed fraction of features with CV < 0.30 matches ~99% and CV < 0.15 matches ~92% in the FGENTCARD case-study (within rounding tolerance)
- cv_table.csv contains a CV value for every metabolic feature present in metabo_SE
- QC_CV is called with plot_hist=FALSE so no histogram is rendered unless explicitly requested
- QC_CV_specNMR is run against the reference QC sample 'QC1' for spectral reproducibility

## Limitations

- CV-based filtering at 0.30 is justified only when QC replicates are present and the case-study reproducibility benchmarks (~99% / ~92% pass rates) are approximately reproduced; datasets failing these rates may need stricter filtering
- The 0.30 and 0.15 thresholds are FDA-established for biomarker discovery and quantification respectively and are not dataset-specific optimization targets

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] package key functionalities include QC analysis: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis"
