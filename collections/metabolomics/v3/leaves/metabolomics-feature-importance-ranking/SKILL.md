---
name: metabolomics-feature-importance-ranking
description: Use when replicating a published disease-severity classifier (e.g. recovered
  vs deceased COVID-19 outcomes from longitudinal metabolomics data) and you must
  check that model performance and the ranked important metabolite features match
  the original study.
when_to_use_negative:
- The goal is unsupervised exploration of metabolite trajectories (PCA, TCAM, cluster
  heatmaps, line plots) rather than outcome classification
- The outcome variable is not a two-class label — MeTEor's Prediction module supports
  only binary classification (LR, RF, XGB)
- Input metabolite values have not yet been checked for numeric type and missing-value
  imputation in MeTEor's filter/imputation steps
edam_operation: http://edamontology.org/operation_2939
edam_topics: []
tools:
- name: MeTEor
  role: R Shiny app whose Prediction module implements the XGBoost binary classifier;
    loads bundled covid_data for replication
  repo: https://github.com/scibiome/meteor
- name: XGBoost
  role: Algorithm selected within MeTEor's Prediction module to fit the recovered-vs-deceased
    classifier
- name: pROC
  role: Used to compute the ROC curve and AUC on the classifier's predictions
- name: caret
  role: Supports the stratified 5-fold cross-validation evaluation of the trained
    model
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae178
    title: 'MeTEor: an R Shiny app for exploring longitudinal metabolomics data'
  source_content_hash: sha256:ac76787135896d1763d824ead05aaf2bdb6407a057ea6ec60fdf6fef183c95a9
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# metabolomics-feature-importance-ranking

## Summary

Replicates supervised severity prediction from longitudinal metabolomics data by training an XGBoost binary classifier in MeTEor's Prediction module and validating the reported model via ROC/AUC plus the top-10 metabolite importance ranking.

## When to use

Use when replicating a published disease-severity classifier (e.g. recovered vs deceased COVID-19 outcomes from longitudinal metabolomics data) and you must check that model performance and the ranked important metabolite features match the original study.

## When NOT to use

- The goal is unsupervised exploration of metabolite trajectories (PCA, TCAM, cluster heatmaps, line plots) rather than outcome classification
- The outcome variable is not a two-class label — MeTEor's Prediction module supports only binary classification (LR, RF, XGB)
- Input metabolite values have not yet been checked for numeric type and missing-value imputation in MeTEor's filter/imputation steps

## Inputs

- Bundled longitudinal metabolomics dataset covid_data (loaded via data("covid_data"))
- Metabolite feature table with RI-coded variables (numeric type, long format after MeTEor preprocessing)
- Binary outcome labels (recovered vs deceased) for the COVID-19-infected sample category

## Outputs

- ROC curve with AUC score (target ≈ 78.3%)
- Ranked top-10 model feature-importance list of metabolites
- Trained MeTEor XGB binary classifier predictions on the held-out test set

## How to apply

Load MeTEor's bundled longitudinal metabolomics dataset via data("covid_data") in R, subset to the COVID-19-infected category with recovered versus deceased as binary outcome labels, and select the metabolite features (RI-coded, e.g. RI1150.81) after the standard MeTEor preprocessing (imputation, wide-to-long reformat). Train the XGBoost (XGB) binary classifier in MeTEor's Prediction module — the alternative algorithms are logistic regression (LR) and random forest (RF) — splitting off a test set and evaluating with stratified 5-fold cross-validation (or LOOCV). Compute the ROC curve and AUC on held-out predictions using pROC/caret, targeting the reported AUC of 78.3% that is comparable to the top-performing models of More et al. (2022). Extract the ten most important features ranked by model importance and confirm that RI1150.81, RI1532.53, and RI1557.73 from the original COVID-19 study appear among them; if they do, and the AUC approximates 78.3%, the replication is validated.

## Related tools

- **MeTEor** (R Shiny app whose Prediction module implements the XGBoost binary classifier; loads bundled covid_data for replication) — https://github.com/scibiome/meteor
- **XGBoost** (Algorithm selected within MeTEor's Prediction module to fit the recovered-vs-deceased classifier)
- **pROC** (Used to compute the ROC curve and AUC on the classifier's predictions)
- **caret** (Supports the stratified 5-fold cross-validation evaluation of the trained model)

## Examples

```
library(MeTEor); data("covid_data")
```

## Evaluation signals

- Computed ROC/AUC on the held-out test set approximates the reported 78.3% for recovered-vs-deceased classification
- Top-10 feature-importance list contains RI1150.81, RI1532.53, and RI1557.73 from the original More et al. (2022) COVID-19 study
- Evaluation used stratified 5-fold cross-validation (or LOOCV as the documented alternative) on a split-off test set
- Predictive capacity is comparable to the top-performing models reported in More et al. (2022)

## Limitations

- MeTEor's Prediction module only offers three algorithms for binary classification (LR, RF, XGB); no multi-class or custom model tuning is documented
- Running MeTEor locally requires a Miniconda environment (rminiconda); on Windows, Miniconda install paths containing spaces can break installation, needing the C:\miniconda_py_r workaround
- The companion MetaboAnalyst ID-conversion API was unavailable at publication time, with only a documented workaround provided

## Evidence

- [methods] Prediction module: "You can perform binary classification using three different algorithms: logistic regression (LR), random forest (RF), and XGBoost (XGB)."
- [abstract] COVID-19 replication: "we replicated the analysis of metabolomics data from a previously published study on COVID-19 patients"
- [readme] Prediction feature set: [evidence span withheld — anchor status: external_doc]
- [abstract] Tool scope: "MeTEor is an R Shiny app providing a comprehensive set of statistical analysis methods"
