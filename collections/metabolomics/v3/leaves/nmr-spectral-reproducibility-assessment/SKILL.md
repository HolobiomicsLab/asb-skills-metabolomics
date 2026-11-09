---
name: nmr-spectral-reproducibility-assessment
description: Use when 1H-NMR metabonomic features must be validated for analytical
  reproducibility via QC-sample CVs, or when deciding whether to filter metabolic
  features at CV < 0.30 (biomarker discovery) or CV < 0.15 (quantification).
when_to_use_negative:
- The dataset lacks QC replicate samples, so no per-feature CV can be computed across
  replicates or against a reference QC sample
- The input is already a curated feature table that has been reproducibility-filtered,
  so re-running CV thresholds would be redundant
- The goal is metabolite-phenotype association modelling (partial correlations, generalized
  linear models) rather than quality control of the spectral data
edam_operation: http://edamontology.org/operation_3218
edam_topics: []
tools:
- name: MWASTools
  role: Supplies the QC_CV and QC_CV_specNMR functions and the packaged metabo_SE
    SummarizedExperiment example dataset used to compute per-feature CVs across QC
    samples and spectral reproducibility
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Implementation environment for MWASTools (version >= 3.4) in which the QC_CV/QC_CV_specNMR
    calls and fraction computations are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:e71498f45290da2fc1c15d1c6ca081067ec8f06bba123989d15cb69798cf6e2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_3218
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# nmr-spectral-reproducibility-assessment

## Summary

Assess analytical reproducibility of 1H-NMR metabolomic data by computing per-feature coefficients of variation across QC replicate samples and spectral reproducibility against a reference QC sample, applying the FDA-established CV thresholds of 0.30 (biomarker discovery) and 0.15 (quantification).

## When to use

Use when 1H-NMR metabonomic features must be validated for analytical reproducibility via QC-sample CVs, or when deciding whether to filter metabolic features at CV < 0.30 (biomarker discovery) or CV < 0.15 (quantification).

## When NOT to use

- The dataset lacks QC replicate samples, so no per-feature CV can be computed across replicates or against a reference QC sample
- The input is already a curated feature table that has been reproducibility-filtered, so re-running CV thresholds would be redundant
- The goal is metabolite-phenotype association modelling (partial correlations, generalized linear models) rather than quality control of the spectral data

## Inputs

- SummarizedExperiment object (e.g. metabo_SE) containing a 1H-NMR feature-by-sample matrix with QC replicate samples identified
- Reference QC sample identifier (e.g. 'QC1') for spectral reproducibility assessment

## Outputs

- Per-feature CV table across QC samples (written to cv_table.csv), with one CV value for every metabolic feature in metabo_SE
- Fraction of features passing CV < 0.30 and fraction passing CV < 0.15
- Optional CV histogram plot (suppressed with plot_hist=FALSE)
- Spectral reproducibility assessment of 1H-NMR data against the reference QC sample

## How to apply

Load the packaged example SummarizedExperiment dataset metabo_SE shipped with the MWASTools R/Bioconductor package (R >= 3.4). Run QC_CV(metabo_SE, plot_hist=FALSE) to compute the coefficient of variation of every metabolic feature across the QC replicate samples without rendering the histogram plot. Run QC_CV_specNMR(metabo_SE, ref_sample="QC1") to assess spectral reproducibility of the 1H-NMR data against the reference QC sample 'QC1'. Extract the per-feature CV values and compute the fraction of features with CV < 0.30 and the fraction with CV < 0.15; in the FGENTCARD plasma case-study these fractions were ~99% and ~92% respectively, confirming dataset reproducibility and justifying CV-based filtering at 0.30.

## Related tools

- **MWASTools** (Supplies the QC_CV and QC_CV_specNMR functions and the packaged metabo_SE SummarizedExperiment example dataset used to compute per-feature CVs across QC samples and spectral reproducibility) — https://bioconductor.org/packages/MWASTools/
- **R** (Implementation environment for MWASTools (version >= 3.4) in which the QC_CV/QC_CV_specNMR calls and fraction computations are executed)

## Examples

```
QC_CV(metabo_SE, plot_hist = FALSE); QC_CV_specNMR(metabo_SE, ref_sample = "QC1")
```

## Evaluation signals

- Computed fraction of features with CV < 0.30 reproduces the reported ~99% within rounding tolerance
- Computed fraction of features with CV < 0.15 reproduces the reported ~92% within rounding tolerance
- cv_table.csv contains a CV value for every metabolic feature in metabo_SE (no missing rows)
- QC_CV is run with plot_hist=FALSE so the histogram is suppressed while CVs are still computed
- QC_CV_specNMR is run with the reference QC sample explicitly set (ref_sample="QC1")

## Limitations

- The 99%/92% pass fractions are specific to the FGENTCARD plasma 1H-NMR case-study dataset; other datasets may show different fractions against the 0.30 and 0.15 thresholds
- The thresholds are the FDA-established values for biomarker discovery (0.30) and quantification (0.15); applying them outside these contexts requires separate justification
- Spectral reproducibility assessment via QC_CV_specNMR depends on choosing an appropriate reference QC sample (here 'QC1')

## Evidence

- [abstract] package design: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [intro] package QC functionality: "Our package provides a versatile and user-friendly MWAS pipeline with the following key functionalities: quality control (QC) analysis"
- [supplementary] reproducibility context: "Assessment of analytical reproducibility of 1H NMR spectroscopy based metabonomics for large-scale epidemiological research: the INTERMAP study"
- [methods] vignette entry: "VignetteIndexEntry{MWASTools}"
