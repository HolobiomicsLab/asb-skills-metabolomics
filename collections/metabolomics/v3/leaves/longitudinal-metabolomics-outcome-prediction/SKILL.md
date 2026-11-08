---
name: longitudinal-metabolomics-outcome-prediction
description: Use when a longitudinal metabolomics dataset has a binary outcome label
  (e.g., recovered vs deceased COVID-19 categories) and the goal is outcome prediction
  with ROC/AUC evaluation and feature-importance inspection.
when_to_use_negative:
- The analysis goal is unsupervised exploration of metabolite trajectories (PCA or
  TCAM), not outcome prediction
- The outcome has more than two classes — MeTEor's Prediction module only supports
  binary classification
- The dataset still contains missing values, non-numeric metabolite columns, or is
  in wide format — run the filtering/imputation steps first
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: MeTEor
  role: R Shiny app whose Prediction module performs the binary classification (LR,
    RF, XGB) and bundles the covid_data dataset
  repo: https://github.com/scibiome/meteor
- name: XGBoost
  role: Primary classification algorithm used to predict recovered vs deceased outcomes
    and rank metabolite feature importance
- name: pROC
  role: Used to compute the ROC curve and AUC on the classifier predictions
- name: caret
  role: Supports the stratified 5-fold cross-validation / test-set splitting workflow
    in R
- name: R
  role: Runtime environment; MeTEor is an R package launchable via library(MeTEor);
    meteor() or the Docker image
  repo: https://github.com/scibiome/meteor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:f30d2646c364b6bd9109ab4e3e37a59ec8f6c6a25ce28265a93460443eea8b47
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# longitudinal-metabolomics-outcome-prediction

## Summary

Train and evaluate a binary classifier (XGBoost, logistic regression, or random forest) in MeTEor's Prediction module to predict patient outcomes from longitudinal metabolomics features, replicating severity classification such as the COVID-19 recovered-vs-deceased task.

## When to use

Use when a longitudinal metabolomics dataset has a binary outcome label (e.g., recovered vs deceased COVID-19 categories) and the goal is outcome prediction with ROC/AUC evaluation and feature-importance inspection.

## When NOT to use

- The analysis goal is unsupervised exploration of metabolite trajectories (PCA or TCAM), not outcome prediction
- The outcome has more than two classes — MeTEor's Prediction module only supports binary classification
- The dataset still contains missing values, non-numeric metabolite columns, or is in wide format — run the filtering/imputation steps first

## Inputs

- Longitudinal metabolomics data in MeTEor-compatible long format (e.g., the covid_data dataset from the MeTEor package)
- Binary outcome labels (e.g., recovered vs deceased COVID-19 category)
- Metabolite feature columns of numeric type after imputation of missing values

## Outputs

- Trained XGBoost binary classifier with ROC curve and AUC (target ≈ 78.3% for the COVID-19 replication)
- Top-10 feature-importance ranking (expected to include RI1150.81, RI1532.53, RI1557.73)
- Cross-validated performance metrics (stratified 5-fold CV or LOOCV)

## How to apply

Load the bundled dataset (e.g., data("covid_data") in the MeTEor package) and subset to the category with a binary outcome label — for the COVID-19 replication, recovered versus deceased outcomes with metabolite features such as RI1150.81, RI1532.53, and RI1557.73. Ensure the data passed through the filter steps first: duplicates removed, irrelevant columns dropped, metabolite values confirmed numeric, missing values imputed, and wide format converted to long format compatible with MeTEor. In MeTEor's Prediction module, train XGBoost (alternatives: logistic regression or random forest), splitting off a test set and evaluating with stratified 5-fold cross-validation or LOOCV. Compute the ROC curve and AUC on the predictions (e.g., with pROC in an R/caret workflow); the COVID-19 replication targets an AUC of 78.3%. Finally extract the ten most important features ranked by model importance and check that the pivotal metabolites from the original study appear among them.

## Related tools

- **MeTEor** (R Shiny app whose Prediction module performs the binary classification (LR, RF, XGB) and bundles the covid_data dataset) — https://github.com/scibiome/meteor
- **XGBoost** (Primary classification algorithm used to predict recovered vs deceased outcomes and rank metabolite feature importance)
- **pROC** (Used to compute the ROC curve and AUC on the classifier predictions)
- **caret** (Supports the stratified 5-fold cross-validation / test-set splitting workflow in R)
- **R** (Runtime environment; MeTEor is an R package launchable via library(MeTEor); meteor() or the Docker image) — https://github.com/scibiome/meteor

## Examples

```
library(MeTEor); data("covid_data"); meteor()  # then in the Prediction module: XGB classifier, stratified 5-fold CV, ROC/AUC evaluation
```

## Evaluation signals

- Computed AUC on the held-out test predictions approximates the reported 78.3% for recovered-vs-deceased classification
- Top-10 feature-importance list contains RI1150.81, RI1532.53, and RI1557.73 from the original More et al. (2022) study
- Metabolite values are all numeric type and missing values were imputed before classification
- Data was transformed from wide to long format before loading into MeTEor
- Cross-validation was stratified 5-fold (or LOOCV) rather than a single unstratified split

## Limitations

- MetaboAnalyst API workaround: the MetaboAnalyst API was unavailable, so ID conversion relies on a manual workaround — relevant if pathway mapping follows prediction
- Binary classification only: logistic regression, random forest, and XGBoost are limited to two-class outcomes
- Installation of MeTEor requires a Miniconda environment (rminiconda); on Windows the Miniconda path must avoid spaces (e.g., C:\miniconda_py_r)
- LOOCV is the alternative to 5-fold CV but is computationally heavier on large datasets

## Evidence

- [methods] You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB).: "You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB)."
- [other] XGBoost AUC comparable to top models: [evidence span withheld — anchor status: not_found]
- [readme] XGBoost install via R package: [evidence span withheld — anchor status: external_doc]
- [methods] Missing-value prerequisite: "The dataset contains missing values, which need to be addressed before conducting analysis in MeTEor."
- [methods] Format prerequisite: "The data format is being transformed from wide to long format to make it compatible with MeTEor."
