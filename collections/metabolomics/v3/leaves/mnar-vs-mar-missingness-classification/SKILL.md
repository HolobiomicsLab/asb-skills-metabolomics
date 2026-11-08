---
name: mnar-vs-mar-missingness-classification
description: Use when a metabolomics feature table contains missing intensity values
  and you need to classify the missingness mechanism (MNAR vs MAR) and apply an imputation
  method before downstream statistical analysis.
when_to_use_negative:
- Input is already fully imputed or contains zero NA/NaN entries — no imputation is
  needed
- Data are raw instrument files (e.g. mzML/NetCDF) rather than a processed feature
  intensity table — peak detection/alignment must precede imputation
- The goal is reporting/visualization of completed analyses rather than preprocessing
  feature tables — use MetaboReport's other modules instead
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaboReport
  role: Provides the preprocessing module implementing the five missing value imputation
    methods (1, 1/5 min, mean, median, KNN) for metabolomics feature tables
  repo: https://github.com/YonghuiDong/MetaboReport
- name: VIM
  role: R package whose kNN function performs k-nearest neighbor imputation over sample
    profiles
- name: stats
  role: R base package whose mean/median functions are used for column-wise single-value
    imputation
- name: R
  role: Language in which the imputation workflow is implemented and in which MetaboReport
    is installed as an R package
  repo: https://github.com/YonghuiDong/MetaboReport
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:ef14d8a90d38521b1f58d549b9d714cf350059d7d6cc0c031871dac9f31057e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - VIM
  - stats
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mnar-vs-mar-missingness-classification

## Summary

Reconstructs MetaboReport's missing value imputation strategy for metabolomics feature tables, covering single-value imputation (constant 1, one-fifth-minimum, mean, median) and k-nearest neighbor (KNN) imputation, chosen according to whether gaps arise from missing-not-at-random (MNAR, e.g. below-detection-limit signals) or missing-at-random (MAR) mechanisms.

## When to use

Use when a metabolomics feature table contains missing intensity values and you need to classify the missingness mechanism (MNAR vs MAR) and apply an imputation method before downstream statistical analysis.

## When NOT to use

- Input is already fully imputed or contains zero NA/NaN entries — no imputation is needed
- Data are raw instrument files (e.g. mzML/NetCDF) rather than a processed feature intensity table — peak detection/alignment must precede imputation
- The goal is reporting/visualization of completed analyses rather than preprocessing feature tables — use MetaboReport's other modules instead

## Inputs

- Metabolomics feature table (samples x features intensity matrix), e.g. the cancerCell demo dataset from MetaboReport
- Feature table with NA/NaN entries representing missing intensity values

## Outputs

- Five imputed CSV tables, one per method: constant-1, one-fifth-minimum, column-wise mean, column-wise median, and KNN
- Missingness pattern classification (missing-at-random vs missing-not-at-random per feature/gap)

## How to apply

First load the feature table (e.g. the cancerCell demo dataset from MetaboReport in R) and profile the missing-value pattern, classifying gaps as missing-at-random versus missing-not-at-random, treating below-detection-limit signals as MNAR. For MNAR-type gaps, apply single-value imputation: replace every missing entry with the constant 1, or with 1/5 of the corresponding feature's minimum observed intensity (the 1/5-minimum rule targets left-censored, below-detection values). For MAR-type gaps, apply column-wise mean or median imputation using R's stats functions, or k-nearest neighbor imputation using the VIM package's kNN function over sample profiles. Write each imputation variant to a separate CSV so they can be compared; MetaboReport implements exactly these five methods (1, 1/5 min, mean, median, KNN) in its preprocessing module.

## Related tools

- **MetaboReport** (Provides the preprocessing module implementing the five missing value imputation methods (1, 1/5 min, mean, median, KNN) for metabolomics feature tables) — https://github.com/YonghuiDong/MetaboReport
- **VIM** (R package whose kNN function performs k-nearest neighbor imputation over sample profiles)
- **stats** (R base package whose mean/median functions are used for column-wise single-value imputation)
- **R** (Language in which the imputation workflow is implemented and in which MetaboReport is installed as an R package) — https://github.com/YonghuiDong/MetaboReport

## Examples

```
imputed <- VIM::kNN(feature_table); imputed <- as.data.frame(imputed[, 1:ncol(feature_table)]); write.csv(imputed, "knn_imputed.csv", row.names=FALSE)
```

## Evaluation signals

- Each of the five imputed CSV tables loads without error
- Each imputed table contains zero NA/NaN entries after imputation
- Each imputed table preserves the original rows and feature columns (same dimensions as the input feature table)
- The one-fifth-minimum table contains imputed values equal to 1/5 of the corresponding feature's minimum observed intensity
- The KNN-imputed table matches results from VIM::kNN applied over the same sample profiles

## Limitations

- Single-value imputation (1, mean, median) collapses variability and can bias downstream variance estimates; the constant-1 and 1/5-minimum rules are appropriate only for MNAR/below-detection-limit gaps, not for MAR gaps
- MetaboReport's public API is intentionally limited to run_app(); imputation helpers are internal, so reconstruction depends on replicating behavior rather than calling exported functions
- The choice between the five methods depends on correctly classifying the missingness mechanism; misclassification (e.g. treating MAR gaps as MNAR) yields inappropriate imputed values

## Evidence

- [methods] MetaboReport's preprocessing implements five missing value imputation methods: single value imputation (1, 1/5 minimum, mean, median) and k-nearest neighbor (KNN): [evidence span withheld — anchor status: not_found]
- [methods] applied to handle values arising from mechanisms such as missing not at random and missing at random: [evidence span withheld — anchor status: not_found]
- [readme] remotes::install_github("YonghuiDong/MetaboReport"): "remotes::install_github("YonghuiDong/MetaboReport")"
- [readme] Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting.: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [methods] cancerCell (MetaboReport): "cancerCell (MetaboReport)"
