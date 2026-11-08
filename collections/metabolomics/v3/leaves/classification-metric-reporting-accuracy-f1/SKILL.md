---
name: classification-metric-reporting-accuracy-f1
description: Use when a categorical-labeled dataset (e.g. wine quality ratings) has
  been split into training and held-out test partitions and you need to quantify prediction
  quality of a fitted pipeline before saving or reusing it.
when_to_use_negative:
- The task is unsupervised (clustering, PCA) with no class label to compare predictions
  against.
- Data still contains missing values that have not been filtered/imputed, since activities
  that cannot handle missing values require pre-filtering.
- You only have training metrics — evaluating on data the grid search already saw
  inflates accuracy and F1.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: Instant Clue
  role: reference GUI whose prediction module lets users optimize pre-processing,
    feature selection/reduction and estimator parameters via exhaustive grid search;
    pipelines can be saved and reused on unseen data
  repo: https://github.com/hnolCol/instantclue
- name: scikit-learn
  role: underlying library for the estimator, GridSearchCV and the accuracy/F1 metrics
    computed on held-out predictions
- name: Python
  role: language used to load the wine quality CSV with pandas, split partitions and
    write the metrics JSON
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-31154-6
    title: 'Instant Clue: A Software Suite for Interactive Data Visualization and
      Analysis'
  source_content_hash: sha256:8240bc958b78c90caa4122b095d2eaa6d84389b610b864ecabcc75025a5d16b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hnolCol/instantclue
  tools:
  - Instant Clue
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classification-metric-reporting-accuracy-f1

## Summary

Evaluate a supervised classification pipeline on held-out data by computing accuracy and F1 score with scikit-learn metrics, then persist the metrics and selected pipeline settings as a structured JSON record.

## When to use

Use when a categorical-labeled dataset (e.g. wine quality ratings) has been split into training and held-out test partitions and you need to quantify prediction quality of a fitted pipeline before saving or reusing it.

## When NOT to use

- The task is unsupervised (clustering, PCA) with no class label to compare predictions against.
- Data still contains missing values that have not been filtered/imputed, since activities that cannot handle missing values require pre-filtering.
- You only have training metrics — evaluating on data the grid search already saw inflates accuracy and F1.

## Inputs

- labeled tabular dataset with a categorical class label (e.g. wine quality CSV from the UCI machine learning repository)
- training partition and disjoint held-out test partition
- grid-search-optimized prediction pipeline (pre-processing + feature selection/reduction + estimator)

## Outputs

- held-out test accuracy
- held-out test F1 score
- structured JSON record of metrics plus selected pipeline settings

## How to apply

Fit the prediction pipeline — pre-processing, feature selection/reduction and classification estimator optimized by exhaustive grid search over given parameters — on the training partition only, mirroring Instant Clue's supervised-learning mechanism built on scikit-learn. Predict class memberships for the disjoint held-out test partition that was never touched during grid search. Compute accuracy and F1 score on those held-out predictions using scikit-learn metrics. Record both metrics together with the winning pre-processing, feature-selection and estimator settings so the established pipeline can be saved and reused to predict class memberships of unseen data.

## Related tools

- **Instant Clue** (reference GUI whose prediction module lets users optimize pre-processing, feature selection/reduction and estimator parameters via exhaustive grid search; pipelines can be saved and reused on unseen data) — https://github.com/hnolCol/instantclue
- **scikit-learn** (underlying library for the estimator, GridSearchCV and the accuracy/F1 metrics computed on held-out predictions)
- **Python** (language used to load the wine quality CSV with pandas, split partitions and write the metrics JSON)

## Examples

```
from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_test, pipe.predict(X_test)); f1 = f1_score(y_test, pipe.predict(X_test))
```

## Evaluation signals

- Held-out accuracy and F1 are computed on the test partition only, never on grid-search training folds
- JSON record contains both metrics and the selected pipeline settings (pre-processing, feature-selection, estimator parameters)
- Test partition is disjoint from the training partition used for GridSearchCV
- Class label encoded as categorical; complete cases retained mirroring automatic missing-value filtering
- Metrics fall in [0,1] and F1 is consistent with per-class precision/recall of the chosen estimator

## Limitations

- Instant Clue's statistical-test panel is still being extended, so some activities for systematic data interpretation are not yet implemented
- Exhaustive grid search over pipeline parameters is computationally expensive on large feature sets
- Metric interpretation depends on class balance; accuracy alone can be misleading for imbalanced categorical labels

## Evidence

- [results] supervised learning pipeline building and held-out metrics: "Users can optimize pre-processing of data, feature selection/reduction and estimator parameters using exhaustive grid searching over given parameters"
- [results] pipeline reuse on unseen data: [evidence span withheld — anchor status: not_found]
- [results] scikit-learn as metric/pipeline basis: "Instant Clue offers several functions to establish an estimator for prediction, based on the scikit-learn library"
