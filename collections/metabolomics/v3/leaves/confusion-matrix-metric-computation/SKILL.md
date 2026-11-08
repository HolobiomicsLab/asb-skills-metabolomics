---
name: confusion-matrix-metric-computation
description: Use when a classification model fitted on metabolomics/proteomics feature
  data needs external validation on held-out test samples, and confusion matrix, accuracy
  and error rate must be reported without training-set leakage.
when_to_use_negative:
- The analysis goal is exploratory or univariate (EDA, univariate testing, dimension
  reduction) rather than supervised classification with external validation
- The data are regression/continuous outcomes rather than class labels, since the
  workflow reports classification confusion matrices
- No held-out test split can be defined — evaluating on the entire data set contradicts
  this skill's external-validation rationale
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: POMAShiny
  role: Web application whose machine learning prediction panel fits regularized regression
    and random forest classifiers and reports confusion matrices and errors on the
    held-out test set
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor package backing POMAShiny's statistical methods, enabling the
    same prediction workflow outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: R
  role: Open-source language in which the POMA/POMAShiny backend and the glmnet/random
    forest model fitting are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:27b126e44c9e7f960daaa07fb6e46243ca336c6821392255865eec2446ab89f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# confusion-matrix-metric-computation

## Summary

Compute and report a confusion matrix with accuracy and error rates strictly on a held-out external test set after fitting glmnet regularized regression and random forest classifiers, following POMAShiny's machine learning prediction workflow.

## When to use

Use when a classification model fitted on metabolomics/proteomics feature data needs external validation on held-out test samples, and confusion matrix, accuracy and error rate must be reported without training-set leakage.

## When NOT to use

- The analysis goal is exploratory or univariate (EDA, univariate testing, dimension reduction) rather than supervised classification with external validation
- The data are regression/continuous outcomes rather than class labels, since the workflow reports classification confusion matrices
- No held-out test split can be defined — evaluating on the entire data set contradicts this skill's external-validation rationale

## Inputs

- features CSV file (metabolite/protein feature matrix, e.g. ST000284 targeted metabolomics three-group data)
- target (metadata) CSV file with class labels for each sample
- train/test sample partition defining the held-out external test set

## Outputs

- confusion matrix for each fitted model (LASSO, ridge, elastic net, random forest) on the held-out test set
- accuracy and error rate for each model computed on test samples only
- prediction_metrics.csv containing the confusion matrix, accuracy and error rate fields for every fitted model

## How to apply

Load the target (metadata) and features CSV files for the study of interest (e.g., the ST000284 targeted metabolomics three-group dataset) into the POMAShiny workflow. Preprocess the data (impute missing values, normalize via transformation and scaling, remove outliers) before modeling, since preprocessing precedes statistical analysis in POMAShiny's four-panel pipeline. Partition samples into random training and held-out test sets, then fit glmnet regularized regression classifiers (LASSO, ridge, elastic net) and a random forest classifier on the training split only. Predict class labels on the held-out test set for each fitted model. For each model, compute the confusion matrix, accuracy and error rate strictly from the held-out test predictions — POMAShiny's random forest returns the confusion matrix and errors calculated on the held-out test set, providing a real measure of model accuracy in contrast to tools that evaluate on the entire dataset. Write the per-model metrics to a prediction_metrics.csv file.

## Related tools

- **POMAShiny** (Web application whose machine learning prediction panel fits regularized regression and random forest classifiers and reports confusion matrices and errors on the held-out test set) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor package backing POMAShiny's statistical methods, enabling the same prediction workflow outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **R** (Open-source language in which the POMA/POMAShiny backend and the glmnet/random forest model fitting are implemented)

## Examples

```
docker pull pcastellanoescuder/pomashiny:v.1.1.14 ; docker run -d --rm -p 3838:3838 pomashiny
```

## Evaluation signals

- prediction_metrics.csv exists and contains confusion matrix, accuracy and error rate fields for every fitted model (LASSO, ridge, elastic net, random forest)
- Confusion matrix dimensions equal the number of study groups (e.g., three groups for ST000284) and total test counts equal the size of the held-out test set
- Accuracy and error rate are complementary (accuracy + error rate = 1) and are computed strictly on held-out test samples with no training samples leaked into evaluation
- Random forest confusion matrix and error match POMAShiny's reported test-set-based accuracy rather than resubstitution accuracy on the full data set

## Limitations

- The article reports no benchmarking or quantitative evaluation of POMAShiny against other tools, so metric values are not cross-validated against reference implementations
- No software version number is stated for POMAShiny or POMA; the README pins an old bs4Dash (0.5.0) that is incompatible with newer versions
- Metric quality depends on the random train/test partition of the small targeted metabolomics example datasets (ST000284, ST000336)

## Evidence

- [intro] including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] The example data set ST000284 consists of a targeted metabolomics three-group study: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [readme] POMAShiny is a web-based tool that provides a structured, flexible and user-friendly workflow for pre-processing, exploratory data analysis (EDA) and statistical analysis of metabolomics and proteomics data.: "POMAShiny is a web-based tool that provides a structured, flexible and user-friendly workflow for pre-processing, exploratory data analysis (EDA) and statistical analysis of metabolomics and"
