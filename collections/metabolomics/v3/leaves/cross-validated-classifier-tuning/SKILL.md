---
name: cross-validated-classifier-tuning
description: Use when a longitudinal metabolomics dataset has a binary outcome label
  (e.g. recovered vs deceased COVID-19 patients) and you need a supervised classifier
  whose predictive performance is benchmarked via cross-validated AUC.
when_to_use_negative:
- The dataset lacks a binary outcome label — MeTEor's Prediction module explicitly
  supports binary classification only; use unsupervised options like PCA or TCAM instead
- The metabolite values are not of "numeric" type — data type checking and cleaning
  (duplicates, irrelevant columns, imputation of missing values, wide-to-long conversion)
  must be completed first
- The goal is exploratory trajectory visualization (ridge plots, line plots, cluster
  heatmaps) rather than outcome prediction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: MeTEor
  role: R Shiny app providing the Prediction module with LR, RF, and XGBoost binary
    classification and cross-validation
  repo: https://github.com/scibiome/meteor
- name: R
  role: Runtime environment; MeTEor is available as an R package and Docker image
  repo: https://github.com/scibiome/meteor
- name: XGBoost
  role: Binary classification algorithm whose recovered-vs-deceased AUC (78.3%) replicates
    More et al. (2022)
- name: pROC
  role: Computation of ROC curves and AUC on classifier predictions
- name: caret
  role: Supports stratified 5-fold cross-validation / LOOCV splitting during model
    training
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:700dac2cc028ce5de22ae6817390b80dabe200b2da030a18e81ddffc8bd5080d
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cross-validated-classifier-tuning

## Summary

Train and evaluate binary outcome classifiers (logistic regression, random forest, or XGBoost) in MeTEor's Prediction module with stratified cross-validation, using ROC/AUC on held-out predictions to validate performance against published benchmarks.

## When to use

Use when a longitudinal metabolomics dataset has a binary outcome label (e.g. recovered vs deceased COVID-19 patients) and you need a supervised classifier whose predictive performance is benchmarked via cross-validated AUC.

## When NOT to use

- The dataset lacks a binary outcome label — MeTEor's Prediction module explicitly supports binary classification only; use unsupervised options like PCA or TCAM instead
- The metabolite values are not of "numeric" type — data type checking and cleaning (duplicates, irrelevant columns, imputation of missing values, wide-to-long conversion) must be completed first
- The goal is exploratory trajectory visualization (ridge plots, line plots, cluster heatmaps) rather than outcome prediction

## Inputs

- covid_data longitudinal metabolomics dataset loaded via data("covid_data") in MeTEor
- Binary outcome labels (recovered vs deceased) from the COVID-19-infected category
- Metabolite feature matrix with numeric metabolite values (e.g. RI1150.81, RI1532.53, RI1557.73)

## Outputs

- Trained binary classifier (LR, RF, or XGBoost) with cross-validated performance
- ROC curve and AUC value (target: 78.3% for recovered-vs-deceased)
- Top-10 feature importance ranking of metabolites

## How to apply

Load the bundled longitudinal metabolomics dataset via data("covid_data") in R, subset to the COVID-19-infected category with recovered versus deceased as the binary outcome, and select the metabolite features for modeling. In MeTEor's Prediction module choose XGBoost (XGB) among the three implemented algorithms — logistic regression (LR), random forest (RF), and XGBoost (XGB) — splitting off a test set. Train with stratified 5-fold cross-validation (LOOCV is the documented alternative) and compute the ROC curve and AUC on the held-out predictions using pROC. Extract the ten most important features ranked by model importance to interpret which metabolites drive the prediction. Validation target from the replicated More et al. (2022) COVID-19 study: an AUC approximating 78.3%, with RI1150.81, RI1532.53, and RI1557.73 appearing among the top-10 critical features.

## Related tools

- **MeTEor** (R Shiny app providing the Prediction module with LR, RF, and XGBoost binary classification and cross-validation) — https://github.com/scibiome/meteor
- **R** (Runtime environment; MeTEor is available as an R package and Docker image) — https://github.com/scibiome/meteor
- **XGBoost** (Binary classification algorithm whose recovered-vs-deceased AUC (78.3%) replicates More et al. (2022))
- **pROC** (Computation of ROC curves and AUC on classifier predictions)
- **caret** (Supports stratified 5-fold cross-validation / LOOCV splitting during model training)

## Examples

```
data("covid_data"); library(MeTEor); meteor()  # then in the Prediction module: outcome=recovered_vs_deceased, algorithm=XGB, CV=5-fold
```

## Evaluation signals

- Computed AUC on held-out predictions approximates the reported 78.3% for recovered-vs-deceased classification
- Top-10 feature importance list contains RI1150.81, RI1532.53, and RI1557.73 from the original COVID-19 study
- Cross-validation is stratified 5-fold (or LOOCV), not a single unstratified split
- Outcome labels are restricted to the COVID-19-infected category with exactly two classes (recovered, deceased)
- All metabolite feature columns are numeric before training

## Limitations

- Performance is comparable to, not necessarily better than, the original More et al. (2022) models; the classifier was benchmarked on one COVID-19 metabolomics dataset
- MetaboAnalyst API unavailability required a workaround for ID conversion, which may affect metabolite annotation upstream of prediction
- Only three binary classification algorithms (LR, RF, XGB) are implemented in the Prediction module
- No changelog is available for the repository

## Evidence

- [methods] You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB).: "You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB)."
- [results] MeTEor's XGBoost classifier attains an AUC of 78.3% for predicting COVID-19 outcomes (recovered versus deceased): [evidence span withheld — anchor status: not_found]
- [methods] Subset the data to the COVID-19-infected category with recovered versus deceased as the binary outcome labels: [evidence span withheld — anchor status: not_found]
- [methods] Train the XGBoost (XGB) binary classifier in MeTEor's Prediction module, splitting off a test set and evaluating with stratified 5-fold cross-validation (or LOOCV as the alternative).: [evidence span withheld — anchor status: not_found]
- [methods] verify that RI1150.81, RI1532.53, and RI1557.73 from the original COVID-19 study appear among the critical features: [evidence span withheld — anchor status: not_found]
