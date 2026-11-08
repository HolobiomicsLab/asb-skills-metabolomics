---
name: cross-platform-classifier-validation
description: Use when a trained peak-quality classifier must be validated on labeled
  independent test sets (e.g. ST000695, MTBLS354, MTBLS306) that differ from the development
  set in platform, matrix, or ionization mode.
when_to_use_negative:
- Input data are vendor-processed outputs in uniquely formatted files rather than
  an XCMS object — MetaClean requires XCMS object input and is not amenable to such
  formats
- No expert Pass/Fail labels are available for the test set, so Accuracy and F-measures
  cannot be computed against a reference
- The goal is selecting among candidate classifiers during training, not validating
  a single trained classifier on held-out data (use the CV + Friedman/Nemenyi skill
  instead)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3368
- http://edamontology.org/topic_3474
tools:
- name: XCMS
  role: pre-processing (peak-picking, RT correction, grouping) to produce the XCMS
    objects from which test-set peak quality metrics are extracted
- name: MetaClean
  role: computes peak quality metrics (getPeakQualityMetrics) and assigns Pass/Fail
    predictions on test sets (getPredicitons)
  repo: https://CRAN.R-project.org/package=MetaClean
- name: caret
  role: trains the AdaBoost M11 classifier (nIter=150) on the development set and
    generates predictions via predict
- name: fastAdaboost
  role: implements the Adaboost.M1 algorithm underlying the selected global classifier
- name: MLmetrics
  role: computes Accuracy and Pass/Fail F-measures against expert reference labels
    on each test set
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:132186c078940ab6dc54bf42b7dd6d72451cb76b338fcc5f38e78fad6732ddb8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MLmetrics
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  - scmamp
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3368
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-platform-classifier-validation

## Summary

Evaluates a trained MetaClean peak-quality classifier on independent test sets from the same and different analytical platforms (Test 1–4), quantifying Accuracy and Pass/Fail F-measures against expert labels and benchmarking against 100-permutation random-chance baselines to prove performance is not due to chance.

## When to use

Use when a trained peak-quality classifier must be validated on labeled independent test sets (e.g. ST000695, MTBLS354, MTBLS306) that differ from the development set in platform, matrix, or ionization mode.

## When NOT to use

- Input data are vendor-processed outputs in uniquely formatted files rather than an XCMS object — MetaClean requires XCMS object input and is not amenable to such formats
- No expert Pass/Fail labels are available for the test set, so Accuracy and F-measures cannot be computed against a reference
- The goal is selecting among candidate classifiers during training, not validating a single trained classifier on held-out data (use the CV + Friedman/Nemenyi skill instead)

## Inputs

- Pre-trained AdaBoost M11 classifier (caret/fastAdaboost model object trained on ST000726 with 11 M11 metrics)
- Independent test-set XCMS objects (ST000695, MTBLS354, MTBLS306 split by ionization mode)
- Test-set peak quality metric matrices (N peaks x 11 M11 metrics)
- Expert Pass/Fail reference labels for each test set
- Balanced-accuracy-optimal probability threshold from cross-validation

## Outputs

- Per-peak Pass/Fail predictions for each test set
- Evaluation measures (Accuracy, Pass F-measure, Fail F-measure) per test set, e.g. results_table.csv
- Averaged random-chance baselines over 100 label-permuted counterpart classifiers
- Comparison table showing real classifier vs. random-chance performance per test set

## How to apply

Train the global classifier (AdaBoost with the M11 metric set, Adaboost.M1 via caret with nIter=150) on the full 500-peak development dataset (Metabolomics Workbench ST000726) using expert Pass/Fail labels from Petrick. Generate the test-set peak quality metric matrices (e.g. 500 peaks x 11 M11 metrics for Test 1/ST000695; Test 2 = MTBLS354; Tests 3 and 4 = MTBLS306 split by positive/negative ionization mode) via MetaClean::getPeakQualityMetrics on the XCMS object. Assign Pass/Fail predictions with MetaClean::getPredicitons (caret predict), binarizing probabilistic outputs at the balanced-accuracy-optimal probability threshold determined during fivefold CV on the development set. Compute Accuracy, Pass F-measure, and Fail F-measure against expert reference labels with calculateEvaluationMeasures/MLmetrics. As a control, permute the development-set labels 100 times, retrain AdaBoost M11 on each randomized dataset, apply all 100 counterparts to the same test sets, average their measures, and confirm the real classifier exceeds the averaged random-chance values on every test set.

## Related tools

- **XCMS** (pre-processing (peak-picking, RT correction, grouping) to produce the XCMS objects from which test-set peak quality metrics are extracted)
- **MetaClean** (computes peak quality metrics (getPeakQualityMetrics) and assigns Pass/Fail predictions on test sets (getPredicitons)) — https://CRAN.R-project.org/package=MetaClean
- **caret** (trains the AdaBoost M11 classifier (nIter=150) on the development set and generates predictions via predict)
- **fastAdaboost** (implements the Adaboost.M1 algorithm underlying the selected global classifier)
- **MLmetrics** (computes Accuracy and Pass/Fail F-measures against expert reference labels on each test set)

## Examples

```
model <- fastAdaboost::adaboost(x = devM11, y = devLabels, nIter = 150); preds <- MetaClean::getPredicitons(model, testM11, threshold = optThresh); calculateEvaluationMeasures(preds, petrickLabels)
```

## Evaluation signals

- Test 1 (ST000695) reference performance: Accuracy = 0.81, F-measure Pass = 0.87, F-measure Fail = 0.62
- Real classifier's Accuracy and Pass/Fail F-measures must exceed the averaged values over 100 random-permutation counterparts on each of the four test sets
- Predictions must be binarized at the balanced-accuracy-optimal threshold from CV, not at 0.5 by default
- Per-peak predictions must match the expert (Petrick) Pass/Fail reference labels at the reported rates when scored with MLmetrics
- Test sets must include both a same-platform case (Test 1) and cross-platform cases (Tests 2–4) to demonstrate cross-platform generalization

## Limitations

- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality (Pass) one for Test 1 and 2, partly because the development set contained fewer low-quality peaks
- Test 2 was more dissimilar from other datasets: it had fewer peaks passing the RSD < 30% threshold and a smaller IPO peak width parameter; many misclassified low-quality peaks were low-abundance peaks indistinguishable from background noise
- Performance varies with the LC-MS run characteristics (ionization mode, analytical platform, biological matrix), which affect peak shape and integration quality and must be considered when applying the classifier

## Evidence
