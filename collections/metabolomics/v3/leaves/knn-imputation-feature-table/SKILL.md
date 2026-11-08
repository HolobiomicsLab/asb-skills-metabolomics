---
name: knn-imputation-feature-table
description: Use when a metabolomics feature table (e.g. the cancerCell MetaboReport
  demo dataset) contains missing values from MNAR (below-detection-limit) or MAR mechanisms
  and you must restore a complete, NA-free intensity matrix before statistical analysis.
when_to_use_negative:
- The input is already a complete feature table with zero missing values — imputation
  would alter observed intensities unnecessarily.
- The goal is MetaboReport's full preprocessing-to-HTML-report workflow (statistical
  analysis, visualization, reporting) rather than imputation alone — use MetaboReport's
  run_app() Shiny app instead.
- Missingness requires model-based methods beyond the five MetaboReport supports (single
  value, 1/5 min, mean, median, KNN) — these are the only methods this skill covers.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: VIM
  role: provides the kNN function used to perform k-nearest neighbor imputation over
    sample profiles
- name: R stats
  role: supplies column-wise mean and median functions for single-value imputation
- name: MetaboReport
  role: source package whose preprocessing module defines the five imputation methods
    and the cancerCell demo feature table used to exercise them
  repo: https://github.com/YonghuiDong/MetaboReport
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:6f467e4096be02f95819e205fd92ebac2905b3493d59ecc7ac858daa8ae39390
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - R stats
  - VIM
  - stats
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# knn-imputation-feature-table

## Summary

Reconstructs MetaboReport-style missing value imputation for metabolomics feature tables, covering k-nearest neighbor (KNN) imputation via the VIM package alongside single-value methods (constant 1, 1/5 minimum, mean, median) that target different missingness mechanisms (MNAR vs MAR).

## When to use

Use when a metabolomics feature table (e.g. the cancerCell MetaboReport demo dataset) contains missing values from MNAR (below-detection-limit) or MAR mechanisms and you must restore a complete, NA-free intensity matrix before statistical analysis.

## When NOT to use

- The input is already a complete feature table with zero missing values — imputation would alter observed intensities unnecessarily.
- The goal is MetaboReport's full preprocessing-to-HTML-report workflow (statistical analysis, visualization, reporting) rather than imputation alone — use MetaboReport's run_app() Shiny app instead.
- Missingness requires model-based methods beyond the five MetaboReport supports (single value, 1/5 min, mean, median, KNN) — these are the only methods this skill covers.

## Inputs

- metabolomics feature table (rows = samples, columns = features) with missing intensity values, e.g. the cancerCell demo feature table from the MetaboReport package
- missing-value pattern profile (NA/NaN locations per feature and per sample)

## Outputs

- five imputed CSV feature tables (constant 1, 1/5 minimum, mean, median, KNN), each with zero NA/NaN entries and unchanged rows and feature columns
- classification of missingness as missing-at-random vs missing-not-at-random

## How to apply

Load the metabolomics feature table in R and profile the missing-value pattern, classifying gaps as missing-at-random versus missing-not-at-random (e.g. below-detection-limit signals). Implement k-nearest neighbor imputation using the VIM package's kNN function applied over sample profiles, and contrast it with single-value alternatives: constant 1, one-fifth of each feature's minimum observed intensity, column-wise mean, and column-wise median via R's stats functions. Each method exploits a different assumption about the missingness mechanism, so run them in parallel on the same table for comparison. Write each imputed result to a separate CSV and verify that every imputed table loads without error, contains zero NA/NaN entries, and preserves the original rows and feature columns.

## Related tools

- **VIM** (provides the kNN function used to perform k-nearest neighbor imputation over sample profiles)
- **R stats** (supplies column-wise mean and median functions for single-value imputation)
- **MetaboReport** (source package whose preprocessing module defines the five imputation methods and the cancerCell demo feature table used to exercise them) — https://github.com/YonghuiDong/MetaboReport

## Examples

```
Rscript -e 'library(VIM); ft <- read.csv("feature_table.csv"); imputed <- kNN(ft); write.csv(imputed, "imputed_knn.csv", row.names=FALSE)'
```

## Evaluation signals

- Each of the five imputed CSV tables loads without error.
- Each imputed table contains zero NA/NaN entries after imputation.
- Rows and feature columns of every imputed table match the original feature table (dimensionality preserved).
- KNN-imputed values are reconstructed from the most similar sample profiles, differing from the constant single-value fills (1, 1/5 min, mean, median).
- Missingness classification distinguishes MNAR gaps (e.g. below-detection-limit signals) from MAR gaps before method selection.

## Limitations

- MetaboReport's preprocessing supports exactly five imputation methods — single value (1, 1/5 minimum, mean, median) and KNN — so other approaches are out of scope.
- Single-value imputations (constant 1, 1/5 minimum, mean, median) assume a specific missingness mechanism; applying the wrong one distorts downstream statistics.
- MetaboReport's public R API is intentionally minimal (run_app()); the analysis helpers that implement imputation are internal, so reconstructing them in R requires reimplementing the workflow as in the source card.

## Evidence

- [results] single value imputation (1, 1/5 minimum, mean, median) and k-nearest neighbor (KNN): [evidence span withheld — anchor status: not_found]
- [results] missing not at random and missing at random: [evidence span withheld — anchor status: not_found]
- [readme] Shiny app for metabolomics data preprocessing: "Shiny app for metabolomics data preprocessing, statistical analysis, visualisation, and self-contained HTML reporting."
- [methods] cancerCell demo dataset: "- `cancerCell (MetaboReport)`"
