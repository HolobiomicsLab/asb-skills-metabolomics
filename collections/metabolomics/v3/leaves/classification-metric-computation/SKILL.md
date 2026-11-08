---
name: classification-metric-computation
description: Decision support for quantifying whether an integrated multi-assay MB-PLS
  model outperforms single-block PLS models on a binary classification task (e.g.
  sex prediction), using per-repeat metrics from Monte Carlo cross-validation.
when_to_use_negative:
- Input is a single-assay feature table with no multi-block structure to compare —
  a simple single-block PLS with plain metric reporting suffices, and no MB-PLS vs
  single-block delta is meaningful.
- The outcome variable is continuous and only regression metrics (Q2, RMSE) are required
  without the classification accuracy/AUC/F1 columns.
- Only a single train/test split is needed for a quick diagnostic rather than the
  1000-repeat MCCV distribution used for delta comparison.
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mamsi
  role: Provides the MamsiPls class whose montecarlo_cv() and calculate_ci() methods
    perform the repeated-split evaluation and metric aggregation for classification.
  repo: https://github.com/kopeckylukas/py-mamsi
- name: scikit-learn
  role: Underlying dependency supplying classification metric implementations (accuracy,
    AUC, F1) used inside MamsiPls evaluation methods.
- name: pandas
  role: Loads the LC-MS feature CSVs and outcome metadata, and holds the per-repeat
    metric DataFrames returned by montecarlo_cv.
- name: mbpls
  role: Pinned dependency (mbpls==1.0.4) implementing the NIPALS-based multiblock
    PLS algorithm that MamsiPls wraps for model fitting.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:4ce481674cb25940f845d43dc1ebeb30610030bb8d721a7e0ccb9558976345e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - mamsi
  - mbpls
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# classification-metric-computation

## Summary

Computes and aggregates classification performance metrics (accuracy, AUC, F1 score) from repeated train/test splits of MB-PLS and single-block PLS models, enabling statistical comparison of multi-assay versus single-assay model performance in the MAMSI framework.

## When to use

Decision support for quantifying whether an integrated multi-assay MB-PLS model outperforms single-block PLS models on a binary classification task (e.g. sex prediction), using per-repeat metrics from Monte Carlo cross-validation.

## When NOT to use

- Input is a single-assay feature table with no multi-block structure to compare — a simple single-block PLS with plain metric reporting suffices, and no MB-PLS vs single-block delta is meaningful.
- The outcome variable is continuous and only regression metrics (Q2, RMSE) are required without the classification accuracy/AUC/F1 columns.
- Only a single train/test split is needed for a quick diagnostic rather than the 1000-repeat MCCV distribution used for delta comparison.

## Inputs

- Training and testing subsets of the multi-assay LC-MS feature tables (e.g. alz_hpos.csv, alz_lpos.csv, alz_lneg.csv) with assay-prefixed column names
- Binary-encoded outcome labels from metadata (e.g. Gender encoded as 1 for female, 0 for male in alz_metadata.csv)
- MamsiPls model instance fitted with the chosen number of latent variables (e.g. n_components=6 for classification)

## Outputs

- Per-repeat classification metrics DataFrame from montecarlo_cv (columns: accuracy, AUC, F1; or Q2/RMSE for regression)
- Aggregated metrics table from calculate_ci with mean, margin of error, and confidence interval per metric column
- MB-PLS vs single-block performance comparison table (mccv_performance_comparison.csv) with metric deltas

## How to apply

Fit the MamsiPls MB-PLS model (NIPALS algorithm, standardise=True) on concatenated assay blocks and select the number of latent variables via cross-validation with estimate_lv using the 'auc' metric (6 LVs in the AddNeuroMed serum classification example). Evaluate model performance using MamsiPls.montecarlo_cv() with 1000 MCCV repeats (test_size=0.2, random_state=42, n_jobs=-1), which returns per-repeat accuracy, AUC, and F1 score; for regression tasks the same call returns Q2 and RMSE. Aggregate the per-repeat metric DataFrame with MamsiPls.calculate_ci() to obtain mean, margin of error, and confidence interval for each metric column. Compute the MB-PLS minus single-block deltas (e.g. +0.055 accuracy, +0.054 AUC, +0.061 F1 for serum sex prediction) by fitting identical MCCV runs on each individual assay block. Validate that reproduced deltas match reported values and write the comparison table to mccv_performance_comparison.csv.

## Related tools

- **mamsi** (Provides the MamsiPls class whose montecarlo_cv() and calculate_ci() methods perform the repeated-split evaluation and metric aggregation for classification.) — https://github.com/kopeckylukas/py-mamsi
- **scikit-learn** (Underlying dependency supplying classification metric implementations (accuracy, AUC, F1) used inside MamsiPls evaluation methods.)
- **pandas** (Loads the LC-MS feature CSVs and outcome metadata, and holds the per-repeat metric DataFrames returned by montecarlo_cv.)
- **mbpls** (Pinned dependency (mbpls==1.0.4) implementing the NIPALS-based multiblock PLS algorithm that MamsiPls wraps for model fitting.)

## Examples

```
mamsipls.montecarlo_cv([hpos, lpos, lneg], y, test_size=0.2, n_repeats=1000, random_state=42, n_jobs=-1); mamsipls.calculate_ci()
```

## Evaluation signals

- Reproduced classification deltas match reported values: accuracy +0.055, AUC +0.054, F1 +0.061 for serum sex prediction (MB-PLS vs mean of single-block PLS).
- Regression-side validation deltas match reported Q2 +0.157 and RMSE −0.506 for bilirubin, confirming the same MCCV/calculate_ci pipeline is consistent across tasks.
- calculate_ci output contains mean, margin of error, and confidence interval for every metric column, not just the mean.
- montecarlo_cv was run with the documented settings (1000 repeats, test_size=0.2, random_state=42, n_jobs=-1) so per-repeat metrics are reproducible.
- The comparison table is written to mccv_performance_comparison.csv with MB-PLS and single-block rows and metric delta columns.

## Limitations

- The reported deltas are specific to the AddNeuroMed and MY Diabetes cohorts with three assays (HPOS/LPOS/LNEG or analogous blocks); different assays or outcome definitions may not reproduce the same magnitudes.
- MCCV with 1000 repeats is computationally heavy; n_jobs=-1 parallelism is required and results depend on the fixed random_state=42 seed.
- Classification performance depends on correct LV selection via estimate_lv with the 'auc' metric — an inappropriate LV count (not 6 in the reference example) will distort the comparison.

## Evidence

- [methods] montecarlo_cv method support: "Evaluate MB-PLS model using Monte Carlo Cross-Validation (MCCV)."
- [methods] calculate_ci method support: "Calculates mean, margin of error, and confidence interval for each column."
- [methods] LV estimation guidance: "A method to estimate the number of latent variables (LVs)/components in the MB-PLS model."
- [methods] reported final classification metrics: "| Accuracy            | 0.966 |
| Recall              | 1.0   |
| Specificity         | 0.943 |
| F<sub>1</sub> Score | 0.971 |
| AUC                 | 0.933 |"
- [abstract] MB-PLS outperformance claim: "The MB-PLS models outperformed single-assay models in both classification and regression tasks, indicating that modeling interblock relationships enabled an improved estimate of phenotypic outcome."
