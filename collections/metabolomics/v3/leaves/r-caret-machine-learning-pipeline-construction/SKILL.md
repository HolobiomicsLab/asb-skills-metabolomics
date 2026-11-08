---
name: r-caret-machine-learning-pipeline-construction
description: Use when you have a labeled feature matrix (e.g., expert Pass/Fail-labeled
  LC-MS peaks) and need to train, cross-validate, and statistically rank several caret
  classification algorithms across candidate feature sets.
when_to_use_negative:
- Input is already a validated single-classifier workflow and no algorithm comparison
  is needed
- Only unlabeled features are available — the pipeline requires Pass/Fail supervision
  to train classifiers
- The goal is peak detection or alignment itself rather than post-hoc peak-quality
  classification
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: caret
  role: Unified R framework used to train and tune the eight candidate classification
    algorithms over the metric-set matrices
- name: scmamp
  role: R package used to run Friedman's test and Nemenyi's post-hoc correction on
    per-repeat cross-validation performance statistics
- name: MLmetrics
  role: Provides accuracy, balanced accuracy, sensitivity/specificity (recallPass/recallFail),
    precision/precisionFail, and Pass/Fail F-measure scoring for out-of-fold predictions
- name: MetaClean
  role: Wraps the winning classifier(s) and peak-quality metric computation for integration
    into untargeted LC-MS pipelines
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Produces the feature table and per-sample peak metrics that feed the training
    matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:a783bdc54392e9ee7fdf687ed718f1a83e2f2936047d64760ddcb23d7631421c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MLmetrics
  - MetaClean
  - XCMS
  - caret
  - scmamp
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# r-caret-machine-learning-pipeline-construction

## Summary

Construct and compare supervised classification pipelines in R using caret, pairing LC-MS peak-quality metric sets with multiple algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, Linear SVM, AdaBoost, Model-Averaged Neural Network, Random Forest) to select a best-performing peak-quality classifier via repeated cross-validation and Friedman/Nemenyi statistics.

## When to use

Use when you have a labeled feature matrix (e.g., expert Pass/Fail-labeled LC-MS peaks) and need to train, cross-validate, and statistically rank several caret classification algorithms across candidate feature sets.

## When NOT to use

- Input is already a validated single-classifier workflow and no algorithm comparison is needed
- Only unlabeled features are available — the pipeline requires Pass/Fail supervision to train classifiers
- The goal is peak detection or alignment itself rather than post-hoc peak-quality classification

## Inputs

- Labeled peak-quality metric matrix (peaks × metrics) derived from an XCMS feature table, with Pass/Fail expert labels
- Metric set matrices for M4, M7, and M11 computed from per-sample peak metrics averaged per peak
- Label vector (Pass = high-quality, Fail = low-quality) for each peak

## Outputs

- Fitted caret classifier objects for each metric-set/algorithm combination
- Cross-validation evaluation table with accuracy, balanced accuracy, sensitivity, specificity, PPV, NPV, and F-measures per class for each of the 24 candidate classifiers
- Friedman/Nemenyi statistical comparison results identifying the best classifier per metric set and the global winner

## How to apply

Build a matrix of peak-quality metrics per feature — here the M4 set (sharpness, Gaussian similarity, TPASR, local zigzag index), the M7 set (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention time consistency), and their union M11 — computed as the mean per-sample value for each peak from an XCMS feature table. Pair each metric matrix with eight caret classification algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, Linear SVM, AdaBoost, Model-Averaged Neural Network, Random Forest) to form 24 candidate classifiers. Train each with a repeated fivefold cross-validation procedure (repeated 10 times), binarizing predictions at the probability threshold that maximizes balanced accuracy within each fold. Score concatenated out-of-fold predictions on accuracy, balanced accuracy, sensitivity (recallPass), specificity (recallFail), PPV (precisionPass), NPV (precisionFail), and Pass/Fail F-measures using MLmetrics. Rank the candidates by first running an intra-metric-set round and then an inter-metric-set round of Friedman's test followed by Nemenyi's post-hoc correction (scmamp R package), which in this study selected AdaBoost with M11 as the global best classifier.

## Related tools

- **caret** (Unified R framework used to train and tune the eight candidate classification algorithms over the metric-set matrices)
- **scmamp** (R package used to run Friedman's test and Nemenyi's post-hoc correction on per-repeat cross-validation performance statistics)
- **MLmetrics** (Provides accuracy, balanced accuracy, sensitivity/specificity (recallPass/recallFail), precision/precisionFail, and Pass/Fail F-measure scoring for out-of-fold predictions)
- **MetaClean** (Wraps the winning classifier(s) and peak-quality metric computation for integration into untargeted LC-MS pipelines) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Produces the feature table and per-sample peak metrics that feed the training matrices)

## Examples

```
install.packages(c('caret','scmamp','MLmetrics')); install.packages('MetaClean'); library(caret); fit <- train(M11_matrix, labels, method = 'adaboost', trControl = trainControl(method = 'cv', number = 5))
```

## Evaluation signals

- Reproduced CV evaluation table must report all 7 performance measures (accuracy, balanced accuracy, sensitivity, specificity, PPV, NPV, F-measure) for all 24 classifiers
- Friedman–Nemenyi analysis should select AdaBoost with the M11 metric set as the global best classifier, with reported p-values in the range p = 0.001–0.005 for Pass and p = 0.0023 for Fail
- Per-classifier F-measure for Pass and Fail classes must be computable from concatenated out-of-fold predictions, not resubstitution accuracy
- Balanced-accuracy-based threshold binarization must be refit inside each fold to avoid test-set leakage
- Label-permutation sanity check: retraining the winning classifier on randomly permuted labels should degrade performance toward chance

## Limitations

- Only moderately significant differences in performance between metric-set pairs were observed under Wilcoxon Rank Sum tests, so the intra/inter metric-set comparison is sensitive to the specific candidate pool
- Candidate classifiers were conservative for the low-quality (Fail) class on Test 1 and 2, partly because development sets contained few low-quality peak examples
- Performance depends on the upstream XCMS peak-picking/retention-time-correction/grouping parameters; vendor-processed feature tables not in XCMS format are not amenable to this pipeline in its current form
- Ionization mode, analytical platform/method, and biological matrix affect peak shape and size and therefore classifier transferability across datasets

## Evidence
