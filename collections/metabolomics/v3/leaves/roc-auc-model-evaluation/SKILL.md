---
name: roc-auc-model-evaluation
description: Use when a longitudinal metabolomics dataset has a two-class outcome
  label (e.g. COVID-19 recovered vs. deceased) and the goal is classifier performance
  quantified by ROC/AUC plus feature-importance ranking.
when_to_use_negative:
- The outcome is multi-class, continuous, or has no reliable binary labels — MeTEor's
  Prediction module is explicitly restricted to binary classification (LR, RF, XGB)
- Data still contains missing values, duplicate cases, or non-numeric metabolite columns
  — MeTEor requires imputation, duplicate removal, and numeric data types before modeling
- The analysis goal is unsupervised exploration or dimensionality reduction (PCA,
  TCAM) rather than supervised performance benchmarking
tools:
- name: MeTEor
  role: R Shiny app whose Prediction module implements the binary classification algorithms
    and evaluation workflow
  repo: https://github.com/scibiome/meteor
- name: R
  role: Runtime environment/package source in which MeTEor runs (data("covid_data")
    loading, remotes installation)
  repo: https://github.com/scibiome/meteor
- name: XGBoost
  role: Primary binary classification algorithm producing the 78.3% AUC severity prediction
    and feature importances
- name: pROC
  role: ROC curve and AUC computation on classifier predictions
- name: caret
  role: Stratified cross-validation (5-fold) / LOOCV splitting and train/test management
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:13f60d4738f2da34e5c6129d4f90d52db24c11cce411691b6e9ec1f8b7ffe795
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/meteor
  tools:
  - MeTEor
  - R
  - XGBoost
  - caret
  - pROC
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# roc-auc-model-evaluation

## Summary

Evaluate binary classifiers (logistic regression, random forest, or XGBoost) in MeTEor's Prediction module by computing ROC curves and AUC on held-out, stratified cross-validated predictions, benchmarked against the COVID-19 recovered-vs-deceased severity classification that reproduces More et al. (2022) at ~78.3% AUC.

## When to use

Use when a longitudinal metabolomics dataset has a two-class outcome label (e.g. COVID-19 recovered vs. deceased) and the goal is classifier performance quantified by ROC/AUC plus feature-importance ranking.

## When NOT to use

- The outcome is multi-class, continuous, or has no reliable binary labels — MeTEor's Prediction module is explicitly restricted to binary classification (LR, RF, XGB)
- Data still contains missing values, duplicate cases, or non-numeric metabolite columns — MeTEor requires imputation, duplicate removal, and numeric data types before modeling
- The analysis goal is unsupervised exploration or dimensionality reduction (PCA, TCAM) rather than supervised performance benchmarking

## Inputs

- MeTEor-format longitudinal metabolomics data table (wide format converted to long format, numeric metabolite values, rows are cases with an outcome column)
- Binary outcome labels (COVID-19 recovered vs. deceased) for the infected sub-cohort
- Metabolite feature matrix with RI-indexed features (e.g. RI1150.81, RI1532.53, RI1557.73)

## Outputs

- ROC curve and AUC value for the trained binary classifier (target ≈ 78.3% AUC)
- Ranked model feature-importance list (top 10 metabolite features)
- Fitted MeTEor XGBoost/LR/RF model with stratified 5-fold CV (or LOOCV) evaluation results

## How to apply

Load the metabolomics data via the MeTEor import module (e.g. the bundled covid_data loaded with data("covid_data")), subset to the infected category with recovered vs. deceased as the binary outcome, and select the metabolite (RI-indexed) features. In the Prediction module, train the XGBoost (XGB) binary classifier — logistic regression (LR) and random forest (RF) are the alternatives — splitting off a test set and using stratified 5-fold cross-validation (or LOOCV as the alternative) with pROC/caret for metric computation. Compute the ROC curve and AUC on held-out predictions; MeTEor's XGBoost reaches 78.3% AUC for this task, comparable to the top models of the original More et al. (2022) COVID-19 study. Finally extract the ten most important features by model importance and check whether RI1150.81, RI1532.53, and RI1557.73 appear, validating that severity-predictive capacity transfers.

## Related tools

- **MeTEor** (R Shiny app whose Prediction module implements the binary classification algorithms and evaluation workflow) — https://github.com/scibiome/meteor
- **R** (Runtime environment/package source in which MeTEor runs (data("covid_data") loading, remotes installation)) — https://github.com/scibiome/meteor
- **XGBoost** (Primary binary classification algorithm producing the 78.3% AUC severity prediction and feature importances)
- **pROC** (ROC curve and AUC computation on classifier predictions)
- **caret** (Stratified cross-validation (5-fold) / LOOCV splitting and train/test management)

## Examples

```
library(MeTEor); data("covid_data")
```

## Evaluation signals

- Computed AUC on held-out predictions approximates the reported 78.3% (within a small tolerance, stratified 5-fold CV)
- The top-10 model-importance list contains the three reported metabolites RI1150.81, RI1532.53, and RI1557.73
- Fold-wise AUCs under stratified splitting are consistent with the aggregate ROC curve (no leakage between train and test folds)
- Switching the evaluation scheme to LOOCV yields comparable AUC, confirming stability of the severity signal

## Limitations

- MetaboAnalyst's API was unavailable, requiring a workaround that affects the upstream metabolite ID mapping/pathway context
- The 78.3% benchmark applies to the bundled covid_data cohort (recovered vs. deceased); performance on other metabolomics datasets or outcome definitions is not guaranteed
- Only binary classification with LR, RF, and XGB is implemented in the Prediction module; no changelog was found in the repository

## Evidence

- [methods] You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB).: "You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB)."
- [readme] Prediction models: Logistic regression, Random Forest, XGBoost: [evidence span withheld — anchor status: external_doc]
- [readme] MeTEor` is an R Shiny application that offers the possibility to explore longitudinal metabolomics data.: [evidence span withheld — anchor status: external_doc]
