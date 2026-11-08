---
name: lc-ms-peak-integration-quality-prediction
description: Use when an XCMS-processed untargeted LC–MS feature table (e.g., from
  Metabolomics Workbench ST000726/ST000695-style data) contains poorly integrated
  peaks you need to flag automatically, and RSD < 30% QC filtering alone is insufficient.
when_to_use_negative:
- Input is a vendor-processed output in a uniquely formatted file rather than an XCMS
  object — MetaClean requires an XCMS object input
- The data come from a substantially dissimilar analytical platform/method (e.g.,
  Test 2-like datasets with few peaks passing RSD < 30% and small IPO peak widths),
  where classifier performance degrades, especially for low-abundance Fail peaks
- The goal is pure reproducibility filtering rather than integration-quality assessment
  — a simple RSD < 30% cut-off on pooled QC samples is the cheaper alternative
edam_operation: http://edamontology.org/operation_3676
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3361
tools:
- name: MetaClean
  role: R package providing the peak-quality metrics, the trained global AdaBoost
    M11 classifier, and getPredicitons for Pass/Fail prediction
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: 'Pre-processing software that must produce the input object: peak-picking,
    retention-time correction, and grouping of peaks across samples and QCs'
- name: caret
  role: Trains the AdaBoost.M1 model (nIter=150) and powers predict calls inside getPredicitons
- name: MLmetrics
  role: Computes Accuracy and Pass/Fail F-measures against expert reference labels
    via calculateEvaluationMeasures
- name: fastAdaboost
  role: Implements the AdaBoost algorithm underlying the global classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:b41b7a117084982858c440ead8c6e076ecf7db39e40b8823c391d54aabc92476
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MLmetrics
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_3676
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lc-ms-peak-integration-quality-prediction

## Summary

Predicts Pass (high-quality) vs Fail (low-quality) labels for chromatographic peak integrations in untargeted LC–MS metabolomics data using a machine learning classifier — the MetaClean global classifier, AdaBoost trained on the 11-metric M11 peak-quality metric set — as an objective alternative to manual visual assessment or RSD filtering.

## When to use

Use when an XCMS-processed untargeted LC–MS feature table (e.g., from Metabolomics Workbench ST000726/ST000695-style data) contains poorly integrated peaks you need to flag automatically, and RSD < 30% QC filtering alone is insufficient.

## When NOT to use

- Input is a vendor-processed output in a uniquely formatted file rather than an XCMS object — MetaClean requires an XCMS object input
- The data come from a substantially dissimilar analytical platform/method (e.g., Test 2-like datasets with few peaks passing RSD < 30% and small IPO peak widths), where classifier performance degrades, especially for low-abundance Fail peaks
- The goal is pure reproducibility filtering rather than integration-quality assessment — a simple RSD < 30% cut-off on pooled QC samples is the cheaper alternative

## Inputs

- XCMS object from peak-picking, retention-time correction, and grouping of raw LC–MS data (samples plus pooled QCs)
- Peak-quality metric matrix (n_peaks x 11 M11 metrics) computed per dataset
- Pre-trained AdaBoost M11 global classifier (caret model, nIter=150, trained on ST000726)
- Balanced-accuracy-optimal probability threshold from fivefold CV on the development set
- Optional expert Pass/Fail reference labels for evaluation (e.g., ST000695 Test 1)

## Outputs

- Per-peak Pass/Fail quality labels with probabilistic predictions
- Evaluation measures: Accuracy and Pass and Fail F-measures (MLmetrics)
- Per-peak predictions and evaluation measures saved as results_table.csv

## How to apply

Pre-process raw LC–MS data with XCMS (peak-picking, retention time correction, grouping across samples and QCs), then compute the 11 M11 peak-quality metrics (adapted from Zhang et al. 2014 and TargetedMSQC/Eshghi et al. 2018) for every grouped peak, yielding a peaks x metrics matrix. Load the pre-trained global classifier — AdaBoost.M1 with the M11 metric set, trained via the caret package with nIter=150 on the full ST000726 development set. Run MetaClean::getPredicitons (caret predict) to obtain probabilistic Pass/Fail predictions, and binarize each peak at the balanced-accuracy-optimal probability threshold determined during fivefold cross-validation on the development set. To validate against expert labels (e.g., Petrick's Pass/Fail annotations on Test 1 / ST000695, 500 peaks), compute Accuracy and Pass and Fail F-measures via calculateEvaluationMeasures/MLmetrics, and confirm performance exceeds the random-chance level established by 100 label-permutation runs. Rationale: AdaBoost + M11 significantly outperformed the other 23 candidate classifiers on Pass and Fail F-measures (Friedman-Nemenyi p = 0.001–0.005), making it the classifier released in MetaClean for pipeline integration.

## Related tools

- **MetaClean** (R package providing the peak-quality metrics, the trained global AdaBoost M11 classifier, and getPredicitons for Pass/Fail prediction) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Pre-processing software that must produce the input object: peak-picking, retention-time correction, and grouping of peaks across samples and QCs)
- **caret** (Trains the AdaBoost.M1 model (nIter=150) and powers predict calls inside getPredicitons)
- **MLmetrics** (Computes Accuracy and Pass/Fail F-measures against expert reference labels via calculateEvaluationMeasures)
- **fastAdaboost** (Implements the AdaBoost algorithm underlying the global classifier)

## Examples

```
preds <- MetaClean::getPredicitons(data = test_m11_metrics, fitModel = adaboost_m11_model, probThreshold = balanced_acc_threshold); calculateEvaluationMeasures(preds, expertLabels)
```

## Evaluation signals

- On Test 1 (ST000695, 500 peaks x 11 M11 metrics), the expected reference results are Accuracy = 0.81, Pass F-measure = 0.87, Fail F-measure = 0.62
- Classifier performance must exceed the 100-permutation random-chance counterparts evaluated on the same test set
- Every predicted peak must have a finite probability score that, after binarization at the balanced-accuracy-optimal CV threshold, maps to exactly one Pass or Fail label
- Pass-class F-measure should exceed Fail-class F-measure (the classifier is conservative on low-quality peaks); a reversed pattern suggests a wrong threshold or mis-trained model
- Input matrix schema check: exactly 11 M11 metric columns per peak, computed after XCMS grouping including QC samples

## Limitations

- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one (Pass) for Test 1 and Test 2, partly because the development set contained fewer low-quality peaks
- Many misclassified low-quality peaks in Test 2 were low-abundance and difficult to distinguish from background noise; the development dataset lacked many low-abundance Pass examples
- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs
- Ionization mode, analytical platform and method, and biological matrix affect peak shape/size and thus integration quality, so these characteristics must be considered when applying the methodology
- The paper reports Test 1 performance qualitatively in places (no exact metric values for all 24 candidate classifiers or for all test sets)

## Evidence
