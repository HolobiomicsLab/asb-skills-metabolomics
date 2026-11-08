---
name: held-out-classification-performance-evaluation
description: Use when a labeled categorical class label (e.g. UCI wine quality ratings)
  must be predicted on unseen samples and you need train/test-disjoint accuracy and
  F1 for a tuned pre-processing + feature-selection + estimator pipeline.
when_to_use_negative:
- The target is continuous (e.g. raw quality as a numeric score) and no categorical
  class label has been encoded — regression metrics, not accuracy/F1, are required.
- There is no way to create a disjoint held-out test partition (e.g. only a handful
  of samples); use cross-validated scores on the single partition instead of claiming
  held-out performance.
- The goal is unsupervised exploration (dimensional reduction, clustering) rather
  than supervised class prediction, which Instant Clue treats as a separate statistical
  activity.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: scikit-learn
  role: Builds the Pipeline (pre-processing, feature selection/reduction, classifier),
    performs exhaustive GridSearchCV on the training partition, and computes accuracy
    and F1 metrics on the held-out test partition
- name: Instant Clue
  role: Reference prediction module whose interactive dialog-based pipeline (pre-processing,
    feature selection/reduction, estimator optimized by grid search, saved for predicting
    unseen samples) is mirrored in scikit-learn
  repo: https://github.com/hnolCol/instantclue
- name: Python >=3.4
  role: Implementation environment for pandas-based loading/encoding and the scikit-learn
    pipeline; Instant Clue itself is open-source Python and runnable on any platform
    with a working Python version
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:946ae2f6587f10fc51250ed82359121722ee28bd5f147a33f03dada387e78535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - Python >=3.4
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# held-out classification performance evaluation

## Summary

Mirrors Instant Clue's supervised-learning prediction module by building a scikit-learn Pipeline (pre-processing → feature selection/reduction → classifier), tuning it with exhaustive grid search on a training partition only, and scoring the disjoint held-out test partition with accuracy and F1. It quantifies how well an interactive, dialog-driven pipeline translates to reproducible class prediction on unseen data.

## When to use

Use when a labeled categorical class label (e.g. UCI wine quality ratings) must be predicted on unseen samples and you need train/test-disjoint accuracy and F1 for a tuned pre-processing + feature-selection + estimator pipeline.

## When NOT to use

- The target is continuous (e.g. raw quality as a numeric score) and no categorical class label has been encoded — regression metrics, not accuracy/F1, are required.
- There is no way to create a disjoint held-out test partition (e.g. only a handful of samples); use cross-validated scores on the single partition instead of claiming held-out performance.
- The goal is unsupervised exploration (dimensional reduction, clustering) rather than supervised class prediction, which Instant Clue treats as a separate statistical activity.

## Inputs

- wine quality table (UCI machine learning repository CSV) with quality ratings
- categorical class label column (encoded wine quality rating)
- training partition and disjoint held-out test partition of samples
- grid of candidate pre-processing, feature-selection and estimator parameters

## Outputs

- held-out test partition accuracy score
- held-out test partition F1 score
- selected pipeline settings (pre-processing, feature-selection, estimator parameters)
- structured JSON record wine_quality_model_metrics.json

## How to apply

Load the wine quality CSV (UCI machine learning repository) with pandas, encode the quality rating as a categorical class label, and reduce to complete cases — mirroring Instant Clue's behavior of automatically filtering data before activities that cannot handle missing values, without changing the source data. Split samples into a training partition and a disjoint held-out test partition. Chain a scikit-learn Pipeline with a pre-processing step, a feature selection/reduction step and a classification estimator (e.g. random forest), mirroring the software's interactive prediction dialog. Exhaustively grid-search the pre-processing, feature-selection and estimator parameters with GridSearchCV, fitting on the training partition only. Predict labels for the held-out partition, compute accuracy and F1 with scikit-learn metrics, and record the scores plus the selected pipeline settings in a structured JSON record (wine_quality_model_metrics.json).

## Related tools

- **scikit-learn** (Builds the Pipeline (pre-processing, feature selection/reduction, classifier), performs exhaustive GridSearchCV on the training partition, and computes accuracy and F1 metrics on the held-out test partition)
- **Instant Clue** (Reference prediction module whose interactive dialog-based pipeline (pre-processing, feature selection/reduction, estimator optimized by grid search, saved for predicting unseen samples) is mirrored in scikit-learn) — https://github.com/hnolCol/instantclue
- **Python >=3.4** (Implementation environment for pandas-based loading/encoding and the scikit-learn pipeline; Instant Clue itself is open-source Python and runnable on any platform with a working Python version)

## Examples

```
from sklearn.model_selection import GridSearchCV; from sklearn.metrics import accuracy_score, f1_score; gs = GridSearchCV(pipe, param_grid, cv=5).fit(X_train, y_train); acc = accuracy_score(y_test, gs.predict(X_test))
```

## Evaluation signals

- wine_quality_model_metrics.json exists and is a structured record containing held-out accuracy, F1, and the selected pipeline settings
- GridSearchCV was fit only on the training partition — the held-out test partition labels were never exposed during tuning or fitting
- Training and test partitions are disjoint (no sample appears in both)
- Accuracy and F1 values fall within the (0, 1] range expected for a categorical classifier and are reproducible given the recorded pipeline settings
- Input table was reduced to complete cases before model building, matching the automatic missing-value filtering Instant Clue applies before activities that cannot handle NaNs

## Limitations

- Instant Clue's estimator panel is based on scikit-learn and optimized by exhaustive grid searching over given parameters, so search space size drives runtime cost
- Automatic pre-filtering for missing values happens silently before activities; the source data are unchanged, so downstream metrics reflect only complete cases
- Only metrics implemented by the toolset (accuracy, F1) are available; a broader panel of statistical tests is a planned extension, not yet implemented

## Evidence

- [results] Users can optimize pre-processing of data, feature selection/reduction and estimator parameters using exhaustive grid searching over given parameters: "Users can optimize pre-processing of data, feature selection/reduction and estimator parameters using exhaustive grid searching over given parameters"
- [results] estimator for prediction based on scikit-learn: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
- [results] automatic filtering of missing values before activities: "if an activity (each test is an activity) cannot handle missing values, the data are automatically filtered before submission to the specified activity, without changing the source data"
