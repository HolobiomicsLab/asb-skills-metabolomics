---
name: wine-quality-supervised-classification
description: Use when a tabular dataset with a categorical class label (e.g. wine
  quality rating) must be turned into a class-prediction pipeline whose pre-processing,
  feature selection and estimator settings are optimised by exhaustive grid search
  and validated on unseen samples.
when_to_use_negative:
- The target variable is continuous and the task is regression rather than categorical
  class prediction.
- There is no class label available, so the problem is unsupervised (clustering, dimensionality
  reduction only).
- The dataset has substantial missing values that should be imputed rather than reduced
  to complete cases, since the mirrored software automatically filters missing values
  before activities.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: Supplies the Pipeline, GridSearchCV and metrics used to replicate Instant
    Clue's estimator-establishment functionality
- name: Python >=3.4
  role: Runtime in which the pandas loading, pipeline construction and grid search
    are scripted, matching Instant Clue's Python requirement
- name: pandas
  role: Loads the UCI wine quality CSV, encodes quality as a categorical label and
    drops incomplete rows
- name: Instant Clue
  role: Reference implementation whose interactive prediction pipeline dialog this
    workflow mirrors programmatically
  repo: https://github.com/hnolCol/instantclue
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:9bfbead6e5b0ae0410cf8cee9f51404d3c4882dc0b69972eaa05df07d92e4740
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# wine-quality-supervised-classification

## Summary

Builds a supervised class-prediction pipeline on the UCI wine quality table, mirroring Instant Clue's scikit-learn-based prediction module: chained pre-processing, feature selection/reduction and estimator steps exhaustively tuned by grid search and evaluated on a held-out test partition.

## When to use

Use when a tabular dataset with a categorical class label (e.g. wine quality rating) must be turned into a class-prediction pipeline whose pre-processing, feature selection and estimator settings are optimised by exhaustive grid search and validated on unseen samples.

## When NOT to use

- The target variable is continuous and the task is regression rather than categorical class prediction.
- There is no class label available, so the problem is unsupervised (clustering, dimensionality reduction only).
- The dataset has substantial missing values that should be imputed rather than reduced to complete cases, since the mirrored software automatically filters missing values before activities.

## Inputs

- Wine quality table as CSV from the UCI machine learning repository, with the quality rating column to be treated as a categorical class label
- Grid of candidate parameter settings covering pre-processing, feature selection/reduction and estimator blocks
- Train/test partition specification (disjoint held-out test set)

## Outputs

- wine_quality_model_metrics.json containing held-out accuracy, held-out F1 score and the selected pipeline settings
- Fitted classification pipeline (pre-processing → feature selection/reduction → estimator) reusable to predict class memberships of unseen data

## How to apply

Load the wine quality CSV from the UCI machine learning repository with pandas, encode the quality rating as a categorical class label, and reduce the table to complete cases, mirroring Instant Clue's automatic filtering of missing values before activities. Split samples into a training partition and a disjoint held-out test partition, then chain a pre-processing step, a feature selection/reduction step and a classification estimator (e.g. random forest) inside a single scikit-learn Pipeline. Exhaustively grid-search the pipeline's pre-processing, feature-selection and estimator parameters with GridSearchCV, fitting on the training partition only, exactly as Instant Clue users tune these three blocks over given parameters. Finally predict labels for the held-out partition, compute accuracy and F1 with scikit-learn metrics, and record both metrics plus the selected pipeline settings in a structured JSON record.

## Related tools

- **scikit-learn** (Supplies the Pipeline, GridSearchCV and metrics used to replicate Instant Clue's estimator-establishment functionality)
- **Python >=3.4** (Runtime in which the pandas loading, pipeline construction and grid search are scripted, matching Instant Clue's Python requirement)
- **pandas** (Loads the UCI wine quality CSV, encodes quality as a categorical label and drops incomplete rows)
- **Instant Clue** (Reference implementation whose interactive prediction pipeline dialog this workflow mirrors programmatically) — https://github.com/hnolCol/instantclue

## Examples

```
from sklearn.model_selection import GridSearchCV; GridSearchCV(pipe, param_grid).fit(X_train, y_train)
```

## Evaluation signals

- wine_quality_model_metrics.json exists and contains both a held-out accuracy and an F1 score in the [0, 1] range
- The selected pipeline settings for pre-processing, feature selection/reduction and estimator are recorded in the JSON output
- Grid search was fitted on the training partition only; the held-out test partition remained fully disjoint from fitting
- Grid search spanned all three pipeline blocks (pre-processing, feature selection/reduction, estimator), not just the estimator
- Rows with missing values were excluded before fitting, mirroring the source software's automatic pre-activity filtering

## Limitations

- The article only documents exhaustive grid searching over given parameters; no guarantee of globally optimal settings and grid ranges must be chosen by the analyst
- Evaluation is limited to accuracy and F1; no other metrics (e.g. AUC) are part of this mirrored workflow
- Missing values are handled by automatic filtering (complete cases) rather than imputation, which can bias the sample set
- The referenced Wine quality CSV is an external UCI dataset, not the article's own Tutorial_Data_01.xlsx or Example Data 03

## Evidence

- [results] estimator grid search: "Users can optimize pre-processing of data, feature selection/reduction and estimator parameters using exhaustive grid searching over given parameters"
- [results] scikit-learn basis: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] missing-value filtering: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
- [intro] Python availability: "the python source code is also freely available making Instant Clue editable and available on any platform with a working Python version (>=3.4)"
