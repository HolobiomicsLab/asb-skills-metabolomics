---
name: metabolomics-missing-value-qc-and-imputation
description: Use when a Nightingale 1H-NMR metabolomics matrix (CSV/TSV) contains
  missing or zero metabolite values that must be QC-filtered and imputed before computing
  metaboAge, surrogate, or published clinical scores in MiMIR.
when_to_use_negative:
- The input is phenotypic data or a non-Nightingale metabolomics platform, where Nightingale-specific
  column names, plus-one offsets, and MEAN/SD scaling parameters do not apply.
- You only need to binarize phenotypes or calibrate already-computed surrogate scores
  (e.g. via binarize_all_pheno or plattCalibration), which operate downstream of metabolite
  QC.
- A single global QC/imputation is desired across all 24 models; MiMIR instead runs
  per-model QC in parallel with model-specific parameters.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MiMIR
  role: Provides QCprep, QCprep_surrogates, subset_samples_sd, and impute_miss functions
    for Nightingale 1H-NMR sample-level QC and imputation prior to score projection
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: R
  role: Runtime environment for MiMIR functions; MiMIR requires R version 3.6+
  repo: https://github.com/DanieleBizzarri/MiMIR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:964a703fa2662c0022d261da3c1ec191bf83e0a05115b551df40f86e0f1815eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - BiocManager
  - MSPrep
  - MSnbase
  - MetaboReport
  - MiMIR
  - POMA
  - POMAShiny
  - R
  - VIM
  - devtools
  - stats
  - tidyverse
  - xtable
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metabolomics-missing-value-imputation
- missing-value-imputation-metabolomics
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolomics-missing-value-qc-and-imputation

## Summary

A sample-level QC and imputation procedure for Nightingale Health 1H-NMR metabolomics matrices, implemented in MiMIR, that removes samples exceeding missing/zero-value thresholds and SD-based outliers before scaling with bundled MEAN/SD parameters and imputing residual missing values, so pre-trained scores can be projected onto complete data.

## When to use

Use when a Nightingale 1H-NMR metabolomics matrix (CSV/TSV) contains missing or zero metabolite values that must be QC-filtered and imputed before computing metaboAge, surrogate, or published clinical scores in MiMIR.

## When NOT to use

- The input is phenotypic data or a non-Nightingale metabolomics platform, where Nightingale-specific column names, plus-one offsets, and MEAN/SD scaling parameters do not apply.
- You only need to binarize phenotypes or calibrate already-computed surrogate scores (e.g. via binarize_all_pheno or plattCalibration), which operate downstream of metabolite QC.
- A single global QC/imputation is desired across all 24 models; MiMIR instead runs per-model QC in parallel with model-specific parameters.

## Inputs

- Nightingale-format 1H-NMR metabolomics dataset (CSV or TSV with example-matching column names)
- Pre-trained model parameters bundled in MiMIR: PARAM_metaboAge and PARAM_surrogates (including per-model MEAN/SD scaling values)
- QC thresholds Nmax_miss and Nmax_zero (user-configurable)

## Outputs

- QC-filtered, scaled, and imputed metabolite matrix (samples exceeding Nmax_miss/Nmax_zero and SD outliers removed, no unimputed NA cells)
- Per-model sample retention/QC information suitable for downstream projection of the 24 scores/surrogates

## How to apply

Run QCprep (with PARAM_metaboAge) or QCprep_surrogates (with PARAM_surrogates) on the metabolite matrix, which removes samples exceeding the Nmax_miss and Nmax_zero thresholds via subset_samples_miss and subset_samples_zero, and drops SD-based outliers with subset_samples_sd using the model's bundled MEAN/SD values. Match metabolite column names to each model's required feature IDs first, using name-mapping helpers such as prep_met_for_scores (applying the plus-one offset where a model requires it), since QC thresholds and scaling parameters are defined per model. After filtering, scale the retained samples with the model's MEAN/SD parameters and apply impute_miss to fill remaining missing cells, so that apply.fit / apply.fit_surro receive a complete, correctly normalized matrix. The rationale is that each pre-trained model expects its own normalization as published, so QC and imputation must be run per model rather than once globally.

## Related tools

- **MiMIR** (Provides QCprep, QCprep_surrogates, subset_samples_sd, and impute_miss functions for Nightingale 1H-NMR sample-level QC and imputation prior to score projection) — https://github.com/DanieleBizzarri/MiMIR
- **R** (Runtime environment for MiMIR functions; MiMIR requires R version 3.6+) — https://github.com/DanieleBizzarri/MiMIR

## Examples

```
library(MiMIR); qc <- QCprep(mat, PARAM_metaboAge, Nmax_miss=3, Nmax_zero=15); qc_i <- impute_miss(qc)
```

## Evaluation signals

- Retained samples are those not exceeding the Nmax_miss and Nmax_zero thresholds and not flagged as SD-based outliers by subset_samples_sd
- After impute_miss, the matrix contains no unimputed NA cells for all retained samples
- Column names match the model's required feature IDs (e.g. after prep_met_for_scores name mapping, including plus-one offset where required)
- Downstream projection (apply.fit / apply.fit_surro) completes and yields populated values for all 24 score/surrogate columns
- QC is run per model (QCprep with PARAM_metaboAge, QCprep_surrogates with PARAM_surrogates), so each model's own normalization as published is preserved

## Limitations

- Samples with too many missing or zero values (beyond Nmax_miss / Nmax_zero) are dropped rather than imputed, reducing sample size.
- QC thresholds and MEAN/SD scaling values are model-specific; applying one model's parameters to another model's feature set is invalid.
- The tool is designed for Nightingale Health 1H-NMR metabolomics with example-matching column names; other platforms require re-mapping.

## Evidence

- [methods] Per-model QC functions with thresholds: "`QCprep(mat, PARAM_metaboAge, quiet, Nmax_zero, Nmax_miss)`"
- [methods] Surrogate QC function: "`QCprep_surrogates(mat, PARAM_surrogates, Nmax_miss, Nmax_zero, quiet)`"
- [methods] SD-based outlier removal using bundled means/SDs: "`subset_samples_sd(x, MEAN, SD, quiet)`"
- [methods] Name mapping / feature prep requirement: "`prep_met_for_scores(dat, featID, plusone, quiet)`"
- [readme] Upload format constraint from README: "Upload your metabolites with the same column names as in the example dataset (both CSV and TSV are accepted)."
- [abstract] Overall capability motivating QC before projection: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
