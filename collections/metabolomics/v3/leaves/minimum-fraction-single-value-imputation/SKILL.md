---
name: minimum-fraction-single-value-imputation
description: Use when a metabolomics feature table (e.g., the cancerCell demo dataset)
  contains NA/NaN intensities and simple single-value replacement — constant 1, 1/5
  of the feature minimum, or column mean/median — is acceptable before downstream
  statistics.
when_to_use_negative:
- The feature table already contains zero NA/NaN entries — imputation would be a no-op
  and adds no value
- Missingness is missing-at-random and sample-profile similarity is preferred over
  a single constant — use the KNN variant (VIM kNN) instead of single-value replacement
- The task is general web-app reporting or visualization with MetaboReport rather
  than filling missing values in a feature table
tools:
- name: MetaboReport
  role: Provides the preprocessing module whose missing value imputation offers the
    single-value schemes (1, 1/5 minimum, mean, median) alongside KNN
  repo: https://github.com/YonghuiDong/MetaboReport
- name: R
  role: 'Implementation language: stats functions compute column-wise mean/median
    imputations; results are written to CSV'
- name: VIM
  role: Supplies the kNN function for the k-nearest-neighbor imputation alternative
    within the same five-method imputation set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae373
    title: 'MetaboReport: from metabolomics data analysis to comprehensive reporting'
  source_content_hash: sha256:7e3664ce88de05e2f7e3d1787e62b9238ccb3822603caa4dfff25ccf7ccf8f41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YonghuiDong/MetaboReport
  tools:
  - MetaboReport
  - R
  - VIM
  - stats
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# minimum-fraction-single-value-imputation

## Summary

Applies MetaboReport's single-value missing-data imputation for metabolomics feature tables — constant 1, one-fifth-of-feature-minimum, column-wise mean, and median — to fill gaps arising from missing-not-at-random (below-detection-limit) and missing-at-random mechanisms before statistical analysis.

## When to use

Use when a metabolomics feature table (e.g., the cancerCell demo dataset) contains NA/NaN intensities and simple single-value replacement — constant 1, 1/5 of the feature minimum, or column mean/median — is acceptable before downstream statistics.

## When NOT to use

- The feature table already contains zero NA/NaN entries — imputation would be a no-op and adds no value
- Missingness is missing-at-random and sample-profile similarity is preferred over a single constant — use the KNN variant (VIM kNN) instead of single-value replacement
- The task is general web-app reporting or visualization with MetaboReport rather than filling missing values in a feature table

## Inputs

- metabolomics feature table with missing values (rows = samples, columns = features), e.g., the cancerCell demo dataset from MetaboReport
- per-feature minimum observed intensity (computed during the procedure)

## Outputs

- imputed feature table as CSV with zero NA/NaN entries, one CSV per imputation variant (constant 1, 1/5 minimum, mean, median)
- imputed table preserving the original row and feature-column dimensions

## How to apply

Load the metabolomics feature table (e.g., the cancerCell demo table shipped with MetaboReport) and profile the missingness, distinguishing below-detection-limit gaps (missing not at random) from randomly missing entries. For one-fifth-minimum imputation, compute each feature's minimum observed intensity and replace every missing entry in that feature with 1/5 of that minimum; this keeps below-detection signals small rather than inflating them to a column average. For the remaining single-value variants, replace all missing entries with the constant 1, or with column-wise mean and median computed via R's stats functions, writing each variant to a separate CSV. Rationale: constant and fraction-of-minimum schemes suit MNAR below-detection values, while mean/median suit missing-at-random gaps; the same skeleton generalizes to KNN imputation via the VIM package when sample-profile similarity is preferable. Finally, verify each imputed CSV loads without error and contains zero NA/NaN entries while preserving the original rows and feature columns.

## Related tools

- **MetaboReport** (Provides the preprocessing module whose missing value imputation offers the single-value schemes (1, 1/5 minimum, mean, median) alongside KNN) — https://github.com/YonghuiDong/MetaboReport
- **R** (Implementation language: stats functions compute column-wise mean/median imputations; results are written to CSV)
- **VIM** (Supplies the kNN function for the k-nearest-neighbor imputation alternative within the same five-method imputation set)

## Examples

```
imputed <- t(apply(ft, 1, function(x) { x[is.na(x)] <- min(x, na.rm=TRUE)/5; x })); write.csv(imputed, "one_fifth_min.csv")
```

## Evaluation signals

- Each imputed CSV loads without error and contains zero NA/NaN entries
- Output tables preserve the original rows and feature columns of the input feature table
- For one-fifth-minimum imputation, every formerly-missing entry equals exactly 1/5 of its feature's minimum observed intensity (and observed entries are unchanged)
- Constant-1 imputation yields 1 in every formerly-missing cell; mean/median variants match the column-wise stats::mean or stats::median of observed values
- Imputed values from fraction-of-minimum replacement stay small, consistent with below-detection-limit signals rather than column averages

## Limitations

- Single-value imputation is crude relative to KNN: constant and fraction-of-minimum schemes target missing-not-at-random below-detection values, while mean/median suit missing-at-random gaps — choosing the wrong variant for the missingness mechanism can bias downstream statistics
- MetaboReport exposes only run_app() publicly; analysis helpers (including imputation internals) are internal and may evolve without a compatibility contract
- The described procedure was exercised on the cancerCell demo feature table; behavior on other feature-table formats is not demonstrated in the provided context

## Evidence

- [other] MetaboReport's five imputation methods: [evidence span withheld — anchor status: not_found]
- [full_text] One-fifth-minimum procedure: [evidence span withheld — anchor status: not_found]
- [abstract] MetaboReport scope: "we introduced MetaboReport, a versatile and interactive web app that simplifies metabolomics experiment design, data preprocessing, exploration, statistical analysis, visualization, and reporting"
- [readme] Where imputation logic lives in the codebase: "`R/fct_*.R` contains pure analysis, input/output, plotting, and report functions."
- [methods] Demo dataset used for imputation workflow: "cancerCell (MetaboReport)"
