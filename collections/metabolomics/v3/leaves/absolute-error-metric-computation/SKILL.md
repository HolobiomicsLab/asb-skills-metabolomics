---
name: absolute-error-metric-computation
description: Use when a regression model (e.g. random forest on ECFP fingerprints)
  must be evaluated against continuous targets such as retention times in seconds,
  and headline accuracy must be reported as mean and median absolute error on a held-out
  split.
when_to_use_negative:
- Input is already a per-molecule error table and only aggregation/plotting is needed
- The evaluation target is candidate ranking or top-k identification accuracy rather
  than continuous RT error in seconds
- The model outputs classification probabilities instead of continuous retention time
  predictions
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: RandomForestRegressor used to fit fingerprints to retention time and produce
    the predictions whose absolute errors are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:b6426e5e4b935a3530dfceea03c2c4daca4a366820ee73df98837c1530167545
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# absolute-error-metric-computation

## Summary

Computes per-molecule absolute prediction errors (|predicted − experimental|) and their mean and median as the accuracy metrics for a regression model, following the METLIN SMRT paper's evaluation of random forest vs deep learning retention time prediction.

## When to use

Use when a regression model (e.g. random forest on ECFP fingerprints) must be evaluated against continuous targets such as retention times in seconds, and headline accuracy must be reported as mean and median absolute error on a held-out split.

## When NOT to use

- Input is already a per-molecule error table and only aggregation/plotting is needed
- The evaluation target is candidate ranking or top-k identification accuracy rather than continuous RT error in seconds
- The model outputs classification probabilities instead of continuous retention time predictions

## Inputs

- ECFP molecular fingerprints for METLIN SMRT molecules (feature matrix)
- Experimental reverse-phase chromatography retention times in seconds (target vector)
- 75/25 random train/validation split assignment

## Outputs

- Per-molecule absolute errors in seconds for the validation set
- Mean and median absolute error values (baseline: 66 s mean, 42 s median)
- rf_rt_errors.csv with the reproduced metrics

## How to apply

Split the fingerprinted molecules into a 75% training set and 25% validation set by random assignment. Train scikit-learn's RandomForestRegressor with the ECFP fingerprints as input features and retention time (seconds) as the target, then predict RT for the 25% validation set. For each molecule compute the absolute error |predicted − experimental| in seconds; report the mean and median of these per-molecule errors (the METLIN SMRT baseline gives 66 s and 42 s). Save the metrics (e.g. as rf_rt_errors.csv) so they can be compared against the deep learning model's error distribution. Rationale: mean and median absolute errors are scale-interpretable (seconds) and robust summaries that let the fingerprint-based random forest be benchmarked directly against the deep RT predictor.

## Related tools

- **scikit-learn** (RandomForestRegressor used to fit fingerprints to retention time and produce the predictions whose absolute errors are computed)

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rf = RandomForestRegressor().fit(X_train, y_train); errs = abs(rf.predict(X_val) - y_val); print(errs.mean(), errs.median())
```

## Evaluation signals

- Reproduced mean absolute error on the 25% validation set should be close to 66 s
- Reproduced median absolute error should be close to 42 s
- All per-molecule errors must be non-negative and expressed in seconds
- Validation set must comprise exactly 25% of the 80,038-molecule fingerprint set via random assignment
- Metrics file rf_rt_errors.csv exists and contains the two reported summary values

## Limitations

- Random forest on fingerprints is a lower-accuracy baseline relative to the paper's deep learning model, so its metrics should not be presented as state-of-the-art RT prediction
- Absolute errors in seconds are dataset-scale dependent (reverse-phase chromatography RTs), so they are not comparable across chromatographic methods without rescaling

## Evidence

- [methods] Train a random forest regression model using the fingerprints as input features and RT as the target variable with scikit-learn's RandomForestRegressor: [evidence span withheld — anchor status: not_found]
- [methods] Predict RT for the validation set and compute the absolute error: [evidence span withheld — anchor status: not_found]
- [results] Random forest regression using fingerprints yielded lower accuracy: [evidence span withheld — anchor status: not_found]
- [abstract] SMRT dataset covers 80,038 small molecules: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
