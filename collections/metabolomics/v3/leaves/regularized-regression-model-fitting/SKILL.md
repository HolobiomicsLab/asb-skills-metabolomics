---
name: regularized-regression-model-fitting
description: Use when a targeted metabolomics feature table (e.g. ST000284 three-group
  study) with class metadata must be fitted with LASSO/ridge/elastic-net classifiers
  and accuracy must be measured on held-out test samples, not the full dataset.
when_to_use_negative:
- The feature table has not yet passed the POMAShiny preprocessing panel (missing-value
  removal with the 20% default threshold, imputation, normalization), so the model
  input would be corrupted or un-scaled
- The outcome is continuous (regression) rather than categorical classification groups,
  since the workflow evaluates classification via confusion matrix, accuracy and error
  rate
- No held-out test set can be reserved and all samples are consumed by training, because
  then the reported metrics would not constitute external validation — the exact practice
  this skill is meant to contrast against
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: POMAShiny
  role: Web-based workflow whose statistical-analysis panel exposes regularized regression
    analysis and prediction model strategies on metabolomics/proteomics data
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end package used to run the same regularized regression
    analysis reproducibly outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: glmnet
  role: Fits the LASSO, ridge and elastic-net regularized regression classifiers on
    the training split (named in the source-card workflow)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:7d68a875bd905ea17ef9c9ce0c214018ae9accab17fc0a98c7918fa3b716cb96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  - glmnet
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# regularized-regression-model-fitting

## Summary

Fits glmnet regularized regression classifiers (LASSO, ridge, elastic net) on metabolomics training data within the POMAShiny/POMA workflow and validates them on a strictly held-out external test set, reporting confusion matrix, accuracy and error rate rather than in-sample metrics.

## When to use

Use when a targeted metabolomics feature table (e.g. ST000284 three-group study) with class metadata must be fitted with LASSO/ridge/elastic-net classifiers and accuracy must be measured on held-out test samples, not the full dataset.

## When NOT to use

- The feature table has not yet passed the POMAShiny preprocessing panel (missing-value removal with the 20% default threshold, imputation, normalization), so the model input would be corrupted or un-scaled
- The outcome is continuous (regression) rather than categorical classification groups, since the workflow evaluates classification via confusion matrix, accuracy and error rate
- No held-out test set can be reserved and all samples are consumed by training, because then the reported metrics would not constitute external validation — the exact practice this skill is meant to contrast against

## Inputs

- Target (metadata) CSV file with class labels for the three-group ST000284-style study
- Features CSV file (samples x metabolite abundances) from Metabolomics Workbench
- Preprocessed POMA experiment object (post missing-value filtering, imputation and normalization)
- Train/test split specification (held-out external test sample IDs)

## Outputs

- prediction_metrics.csv containing, per fitted regularized regression model, the held-out test-set confusion matrix, accuracy and error rate
- Fitted LASSO, ridge and elastic-net classifier objects
- Predicted class labels on the held-out test samples

## How to apply

Load the dataset (e.g. ST000284, a targeted metabolomics three-group study from Metabolomics Workbench) as the two CSV inputs POMAShiny requires: a target (metadata) file and a features file, after completing the preprocessing panel (missing-value removal at the 20% default threshold, imputation, normalization). Partition samples into a training split and a held-out external test set, then fit glmnet regularized regression classifiers — LASSO, ridge and elastic net — on the training split only. Predict class labels on the held-out test set for each fitted model, and compute the confusion matrix, accuracy and error rate strictly on those test samples so no training samples leak into evaluation. Write the per-model confusion matrix, accuracy and error rate to prediction_metrics.csv; the rationale, per the source card, is that this external-validation scheme gives a real measure of model accuracy, in contrast to comparable tools that evaluate on the entire data set.

## Related tools

- **POMAShiny** (Web-based workflow whose statistical-analysis panel exposes regularized regression analysis and prediction model strategies on metabolomics/proteomics data) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end package used to run the same regularized regression analysis reproducibly outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **glmnet** (Fits the LASSO, ridge and elastic-net regularized regression classifiers on the training split (named in the source-card workflow))

## Examples

```
docker pull pcastellanoescuder/pomashiny:v.1.1.14
```

## Evaluation signals

- prediction_metrics.csv exists and contains confusion matrix, accuracy and error rate fields for every fitted model (LASSO, ridge, elastic net)
- Confusion matrix, accuracy and error rate are computed strictly on the held-out test samples — no training samples leaked into evaluation
- Class labels predicted on the test set align with the three study groups of the ST000284 targeted metabolomics dataset
- Each reported model has predictions for all held-out test samples (no missing predictions in the confusion matrix)
- Test-set error rates should be a realistic external estimate, i.e. plausibly higher than resubstitution accuracy on the full data set

## Limitations

- POMAShiny is not benchmarked or quantitatively evaluated against other tools in the article, so accuracy expectations must come from the external validation itself
- POMAShiny requires exactly two CSV inputs (target/metadata and features files); data in other formats must be converted first
- No software version number is stated for POMAShiny or POMA in the article; the README pins bs4Dash to version 0.5.0 because newer versions break the API, which constrains local deployment
- The article documents the workflow via the example datasets (ST000284 three-group, ST000336 two-group); behavior on datasets with different group structure is not demonstrated

## Evidence

- [intro] POMAShiny offers regularized regression and machine-learning classification with prediction model strategies: "feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] The example dataset used is a targeted metabolomics three-group study from Metabolomics Workbench: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [abstract] POMAShiny is backed by the POMA R/Bioconductor package enabling reproducible analysis outside the web environment: "it is based on the POMA R/Bioconductor package, which increases the reproducibility and flexibility of analyses outside the web environment"
- [readme] The POMAShiny workflow is organized into four sequential panels that must be traversed before model fitting: "POMAShiny’s workflow is structured in four sequential and well-defined panels: 1) data upload, 2) pre-processing, 3) EDA and 4) statistical analysis panels"
