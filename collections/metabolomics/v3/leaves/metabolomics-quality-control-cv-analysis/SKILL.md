---
name: metabolomics-quality-control-cv-analysis
description: Use when a 1H-NMR or metabonomic dataset contains QC replicate samples
  and you must quantify analytical reproducibility per metabolic feature, or decide
  which features to retain via CV < 0.30 / CV < 0.15 filtering.
when_to_use_negative:
- The dataset lacks QC replicate samples, so no across-QC coefficient of variation
  can be computed
- The goal is a downstream metabolite-phenotype association model (partial correlations
  or generalized linear models) rather than analytical reproducibility assessment
edam_operation: http://edamontology.org/operation_2945
edam_topics: []
tools:
- name: MWASTools
  role: R/Bioconductor package providing QC_CV() and QC_CV_specNMR() to compute per-feature
    CVs across QC samples and assess spectral reproducibility
  repo: https://bioconductor.org/packages/MWASTools/
- name: R
  role: Runtime environment in which MWASTools QC functions are executed (version
    >= 3.4)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btx477
    title: 'MWASTools: an R/bioconductor package for metabolome-wide association studies'
  source_content_hash: sha256:56e866b96110008dbae0706f633724f37bcd44b05f2dea108969ee3357842b72
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MWASTools
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metabolomics-quality-control-cv-analysis

## Summary

Quality-control analysis of metabolic features by computing per-feature coefficients of variation (CV) across QC replicate samples and applying FDA-established reproducibility thresholds of 0.30 (biomarker discovery) and 0.15 (quantification), as supported by the MWASTools R/Bioconductor package for metabonomic data.

## When to use

Use when a 1H-NMR or metabonomic dataset contains QC replicate samples and you must quantify analytical reproducibility per metabolic feature, or decide which features to retain via CV < 0.30 / CV < 0.15 filtering.

## When NOT to use

- The dataset lacks QC replicate samples, so no across-QC coefficient of variation can be computed
- The goal is a downstream metabolite-phenotype association model (partial correlations or generalized linear models) rather than analytical reproducibility assessment

## Inputs

- SummarizedExperiment object (e.g. metabo_SE) containing metabolic features and QC replicate samples
- Reference QC sample identifier for spectral reproducibility checks (e.g. ref_sample='QC1')

## Outputs

- Per-feature CV table (e.g. cv_table.csv) with a CV value for every metabolic feature
- Fraction of features with CV < 0.30 and fraction with CV < 0.15
- Optional QC_CV histogram and spectral reproducibility assessment from QC_CV_specNMR

## How to apply

Load the metabolomic data as a SummarizedExperiment (e.g. the metabo_SE example dataset shipped with MWASTools) so that QC replicate samples are identifiable. Run QC_CV(metabo_SE, plot_hist=FALSE) to compute the coefficient of variation of every metabolic feature across the QC replicate samples without rendering the histogram; optionally run QC_CV_specNMR(metabo_SE, ref_sample="QC1") to assess spectral reproducibility of the 1H-NMR data against a reference QC sample. Extract the per-feature CV values and compute the fraction of features passing CV < 0.30 (the FDA biomarker-discovery threshold) and CV < 0.15 (the FDA quantification threshold); in the FGENTCARD plasma case-study, 99% of features were below 0.30 and 92% below 0.15, confirming reproducibility and justifying CV-based filtering at 0.30. Write the full per-feature CV table to CSV so every feature has a recorded value and the fractions can be independently rechecked against rounding tolerance.

## Related tools

- **MWASTools** (R/Bioconductor package providing QC_CV() and QC_CV_specNMR() to compute per-feature CVs across QC samples and assess spectral reproducibility) — https://bioconductor.org/packages/MWASTools/
- **R** (Runtime environment in which MWASTools QC functions are executed (version >= 3.4))

## Examples

```
library(MWASTools); data(metabo_SE); cv <- QC_CV(metabo_SE, plot_hist = FALSE); write.csv(cv, "cv_table.csv")
```

## Evaluation signals

- Computed fraction of features with CV < 0.30 reproduces the reported ~99% (within rounding tolerance)
- Computed fraction of features with CV < 0.15 reproduces the reported ~92% (within rounding tolerance)
- cv_table.csv contains a CV value for every metabolic feature in metabo_SE (no missing rows)
- QC_CV_specNMR output is consistent with spectral reproducibility against the designated reference QC sample (e.g. 'QC1')

## Limitations

- The reported fractions (99% < 0.30, 92% < 0.15) are specific to the FGENTCARD plasma 1H-NMR case-study dataset; other datasets may show lower reproducibility fractions
- The thresholds 0.30 and 0.15 are FDA-established CV cutoffs; applying them to non-biomarker-discovery or non-quantification contexts should be justified independently
- QC_CV is designed for metabonomic data in large-scale epidemiological studies; it assumes QC replicate samples are clearly annotated in the SummarizedExperiment input

## Evidence

- [abstract] MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies: "MWASTools is an R package designed to provide an integrated pipeline to analyse metabonomic data in large-scale epidemiological studies"
- [abstract] Key functionalities of our package include: quality control analysis: "Key functionalities of our package include: quality control analysis"
- [intro] Our package provides a versatile and user-friendly MWAS pipeline with quality control (QC) analysis among its key functionalities: "quality control (QC) analysis; metabolite-phenotype association models (partial correlations, generalized linear models) adjusted for epidemiological confounders (e.g. age or gender)"
