---
name: metabolomics-classifier-training
description: Use when a metabolomics or proteomics feature table with class labels
  needs supervised classification models whose accuracy must be measured on held-out
  test samples rather than on the full data set.
when_to_use_negative:
- The goal is unsupervised exploration (PCA, clustering, EDA) rather than labeled
  class prediction
- Only two CSV files are available but no preprocessing/imputation has been done and
  the data still contain excessive missing values (>20% per study group by default)
- The requirement is evaluation on the entire data set rather than external validation
  on a held-out test set (POMAShiny's design deliberately avoids this)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: POMAShiny
  role: Web-based workflow providing the machine learning classification and prediction
    model panels for regularized regression and random forest with external test-set
    validation
  repo: https://github.com/nutrimetabolomics/POMAShiny
- name: POMA
  role: R/Bioconductor back-end package used to run the same prediction analysis reproducibly
    outside the web environment
  repo: https://github.com/nutrimetabolomics/POMA
- name: R
  role: Open-source programming language in which POMAShiny and the underlying POMA
    prediction functions are wholly written
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1009148
    title: 'POMAShiny: A user-friendly web-based workflow for metabolomics and proteomics
      data analysis'
  source_content_hash: sha256:b389048c75b83832dd2f119f4225aed7db3954e3a4e57dd2695975b9bc876e82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nutrimetabolomics/POMAShiny
  tools:
  - POMA
  - POMAShiny
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9473684210526315
n_claims: 19
---

# metabolomics-classifier-training

## Summary

Trains regularized regression (LASSO, ridge, elastic net via glmnet) and random forest classifiers on metabolomics feature data and validates them on a held-out external test set, reporting confusion matrix, accuracy and error rate — as implemented in POMAShiny's machine learning prediction workflow.

## When to use

Use when a metabolomics or proteomics feature table with class labels needs supervised classification models whose accuracy must be measured on held-out test samples rather than on the full data set.

## When NOT to use

- The goal is unsupervised exploration (PCA, clustering, EDA) rather than labeled class prediction
- Only two CSV files are available but no preprocessing/imputation has been done and the data still contain excessive missing values (>20% per study group by default)
- The requirement is evaluation on the entire data set rather than external validation on a held-out test set (POMAShiny's design deliberately avoids this)

## Inputs

- Features CSV (samples x metabolite features) from a targeted metabolomics study, e.g. ST000284
- Target/metadata CSV with class labels for each sample
- Preprocessed POMAShiny data (missing values imputed, normalized/scaled)
- Train/test split definition (random partition of samples)

## Outputs

- prediction_metrics.csv containing the confusion matrix, accuracy and error rate for every fitted model (regularized regression and random forest)
- Fitted glmnet and random forest classifiers
- Predicted class labels on the held-out test set

## How to apply

Load the metabolomics features and target (metadata) file — e.g. the ST000284 targeted metabolomics three-group study from Metabolomics Workbench — after POMAShiny preprocessing (missing-value handling with the default 20% per-group feature removal threshold, imputation, and normalization such as autoscaling). Partition samples into random training and held-out external test sets. Fit glmnet regularized regression classifiers (LASSO, ridge, elastic net) and a random forest classifier on the training split only, then predict class labels on the held-out test set for each fitted model. Compute the confusion matrix, accuracy and error rate strictly on the test samples; POMAShiny's random forest returns the confusion matrix and errors calculated using the held-out test set, giving a real measure of model accuracy rather than resubstitution metrics computed on the entire data set.

## Related tools

- **POMAShiny** (Web-based workflow providing the machine learning classification and prediction model panels for regularized regression and random forest with external test-set validation) — https://github.com/nutrimetabolomics/POMAShiny
- **POMA** (R/Bioconductor back-end package used to run the same prediction analysis reproducibly outside the web environment) — https://github.com/nutrimetabolomics/POMA
- **R** (Open-source programming language in which POMAShiny and the underlying POMA prediction functions are wholly written)

## Examples

```
git clone "https://github.com/pcastellanoescuder/POMAShiny.git" && Rscript -e "shiny::runApp(appDir = 'app')"  # then run prediction on ST000284 train/test split
```

## Evaluation signals

- prediction_metrics.csv exists and contains the confusion matrix, accuracy and error rate fields for every fitted model (glmnet variants and random forest)
- Metrics are computed strictly on the held-out test samples — no training samples leaked into the evaluation set
- Confusion matrix dimensions match the number of study groups in the target file (e.g. three groups for ST000284)
- Predicted class labels are produced for every held-out test sample by each fitted model
- Reported error rate equals 1 minus accuracy on the same test set (internal consistency check)

## Limitations

- The article does not name a specific POMAShiny or POMA software version, complicating exact reproducibility
- No changelog is available for the repository
- No benchmarking or quantitative comparison of POMAShiny's prediction workflow against other tools is reported
- POMAShiny was built with bs4Dash version 0.5.0 and is not compatible with newer bs4Dash versions due to breaking API changes

## Evidence

- [results] External validation on held-out test set: [evidence span withheld — anchor status: not_found]
- [intro] ML methods offered: "including univariate statistical methods, multivariate and dimension reduction methods, feature selection methods, regularized regression analysis approaches, machine learning classification"
- [results] Input format: "POMAShiny requires two comma-separated value (CSV) files as an input: the target (or metadata) file and the features file"
- [results] Dataset: "The example data set ST000284 consists of a targeted metabolomics three-group study"
- [results] Preprocessing threshold: "remove features of the data with more than a specific percentage of missing values in all the study groups (20% allowed by default)"
