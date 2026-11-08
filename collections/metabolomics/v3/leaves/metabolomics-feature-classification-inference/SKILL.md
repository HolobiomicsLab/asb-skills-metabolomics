---
name: metabolomics-feature-classification-inference
description: Use when a pre-trained peak-quality classifier must be applied to an
  independent XCMS-derived test set from the same analytical platform to flag low-quality
  peak integrations and score classifier performance.
when_to_use_negative:
- Raw LC-MS data that has not yet been peak-picked, retention-time corrected, and
  grouped by XCMS — no peak quality metric matrix exists to classify yet
- Vendor-processed outputs in unique formats, since MetaClean requires an XCMS object
  input and is not amenable to them
- Datasets from a substantially different analytical platform (e.g., Test 2-like data
  with few RSD<30% peaks and small IPO peak widths) without acknowledging the generalization
  risk
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MetaClean
  role: Provides the peak-quality classification workflow, including getPredicitons
    for assigning Pass/Fail labels to test peaks from their metric matrix
  repo: https://CRAN.R-project.org/package=MetaClean
- name: caret
  role: Trains (Adaboost.M1, nIter=150) and predicts with the AdaBoost classifier;
    getPredicitons wraps caret predict
- name: fastAdaboost
  role: Implements the AdaBoost algorithm underlying the best-performing M11 classifier
- name: MLmetrics
  role: Computes Accuracy and Pass/Fail F-measures when scoring predictions against
    expert reference labels
- name: XCMS
  role: Pre-processing source of the feature table (peak-picking, retention time correction,
    grouping) from which the peak quality metric matrix is derived
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:734c1feb8fc1123c5e2524731ab908e50d5ef6a1b59244759ae50d2f7d1e38ec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MLmetrics
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# metabolomics-feature-classification-inference

## Summary

Applies a trained peak-quality classifier (AdaBoost with the 11-metric M11 metric set, nIter=150 via caret) to an independent LC-MS feature table to assign Pass/Fail quality labels to each peak, then evaluates Accuracy and Pass/Fail F-measures against expert reference labels.

## When to use

Use when a pre-trained peak-quality classifier must be applied to an independent XCMS-derived test set from the same analytical platform to flag low-quality peak integrations and score classifier performance.

## When NOT to use

- Raw LC-MS data that has not yet been peak-picked, retention-time corrected, and grouped by XCMS — no peak quality metric matrix exists to classify yet
- Vendor-processed outputs in unique formats, since MetaClean requires an XCMS object input and is not amenable to them
- Datasets from a substantially different analytical platform (e.g., Test 2-like data with few RSD<30% peaks and small IPO peak widths) without acknowledging the generalization risk

## Inputs

- Pre-trained AdaBoost M11 peak-quality classifier (caret, nIter=150, Adaboost.M1)
- Peak quality metric matrix for the test dataset (e.g., 500 peaks x 11 M11 metrics derived from XCMS-processed LC-MS data, Metabolomics Workbench ST000695)
- Expert Pass/Fail reference labels for the test peaks (e.g., labels assigned by Petrick)
- Balanced-accuracy-optimal probability threshold from cross-validation on the development set (ST000726)

## Outputs

- Per-peak Pass/Fail quality predictions for the test dataset
- Evaluation measures: Accuracy, Pass F-measure, Fail F-measure (expected ~0.81, 0.87, 0.62 for Test 1/ST000695)
- results_table.csv containing per-peak predictions and evaluation measures

## How to apply

Load the pre-trained AdaBoost M11 model (trained on the full ST000726 development set with nIter=150 using Adaboost.M1 via the caret package) together with the test dataset's peak quality metric matrix (e.g., 500 peaks x 11 M11 metrics for ST000695). Run probabilistic predictions with MetaClean::getPredicitons (caret predict) and binarize them at the balanced-accuracy-optimal probability threshold determined during fivefold cross-validation on the development set to obtain per-peak Pass/Fail labels. Evaluate those labels against the expert (Petrick) Pass/Fail reference labels using calculateEvaluationMeasures/MLmetrics, computing Accuracy and F-measures for both the Pass and Fail classes. Validate that performance exceeds the counterpart statistics from 100 label-permutation random-chance runs on the same test set, and save the per-peak predictions and evaluation measures as results_table.csv.

## Related tools

- **MetaClean** (Provides the peak-quality classification workflow, including getPredicitons for assigning Pass/Fail labels to test peaks from their metric matrix) — https://CRAN.R-project.org/package=MetaClean
- **caret** (Trains (Adaboost.M1, nIter=150) and predicts with the AdaBoost classifier; getPredicitons wraps caret predict)
- **fastAdaboost** (Implements the AdaBoost algorithm underlying the best-performing M11 classifier)
- **MLmetrics** (Computes Accuracy and Pass/Fail F-measures when scoring predictions against expert reference labels)
- **XCMS** (Pre-processing source of the feature table (peak-picking, retention time correction, grouping) from which the peak quality metric matrix is derived)

## Examples

```
preds <- MetaClean::getPredicitons(model = adaboost_m11_model, data = test1_metric_matrix); labels <- ifelse(preds >= optimal_threshold, 'Pass', 'Fail')
```

## Evaluation signals

- Accuracy on Test 1 (ST000695) should equal 0.81, with F-measure Pass = 0.87 and F-measure Fail = 0.62 against the Petrick expert labels
- Classifier performance must exceed the 100-permutation random-chance counterparts evaluated on the same test set
- Prediction binarization must use the balanced-accuracy-optimal threshold fixed during development-set CV, not a re-tuned threshold
- Output results_table.csv must contain one Pass/Fail prediction per test peak (500 peaks x 11 M11 metrics in) plus the three evaluation measures

## Limitations

- The global classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one for Test 1 and 2, partly because the development set contained fewer low-quality peaks
- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs
- Ionization mode, analytical platform and method, and biological matrix affect peak shape and size, so performance may not transfer across platforms (Test 2-4 vs Test 1)

## Evidence
