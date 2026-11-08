---
name: binary-classification-evaluation-measures-calculation
description: Use when comparing multiple binary classifiers trained on the same labeled
  dataset (e.g. high-quality vs low-quality LC-MS peaks) and you need a rigorous,
  statistically defensible selection of the model using recall, precision, F-measure
  and Friedman-Nemenyi ranking.
when_to_use_negative:
- Input is a continuous-valued regression target or multi-class problem rather than
  a binary Pass/Fail label
- You need to filter peaks directly from raw chromatograms instead of evaluating already-trained
  classifiers on labeled data
- Only a single classifier is available — Friedman's/Nemenyi's tests require comparing
  multiple candidate models across repeated folds
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3524
tools:
- name: MetaClean
  role: Provides the metric matrices (M4, M7, M11) and candidate classifiers whose
    cross-validated predictions are scored; the evaluation measures are computed on
    these predictions.
  repo: https://CRAN.R-project.org/package=MetaClean
- name: caret
  role: Supplies the eight classification algorithms (Decision Tree, Logistic Regression,
    Naive Bayes, Neural Network, Linear SVM, AdaBoost, Model-Averaged Neural Network,
    Random Forest) and the fivefold cross-validation machinery used to generate out-of-fold
    predictions.
- name: scmamp
  role: Performs Friedman's test followed by Nemenyi's post-hoc correction on the
    collected per-repeat performance statistics to select the best-performing classifier.
- name: MLmetrics
  role: Computes the individual classification measures (Accuracy, Balanced Accuracy,
    recall, precision, F-measure) on the concatenated out-of-fold predictions.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:2d826778e4c5296751dbd57bda6f6bbc3838a3dca9d4072aeebcaa143ab2d511
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MLmetrics
  - MetaClean
  - caret
  - scmamp
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Binary classification evaluation measures calculation

## Summary

Compute seven standard classification performance measures (Accuracy, Balanced Accuracy, Sensitivity/recallPass, Specificity/recallFail, PPV/precisionPass, NPV/precisionFail, and Pass/Fail F-measures) on the concatenated out-of-fold predictions produced by candidate classifiers in a repeated fivefold cross-validation, then rank them statistically with Friedman's and Nemenyi's tests to select the best-performing classifier.

## When to use

Use when comparing multiple binary classifiers trained on the same labeled dataset (e.g. high-quality vs low-quality LC-MS peaks) and you need a rigorous, statistically defensible selection of the best model using recall, precision, F-measure and Friedman-Nemenyi ranking.

## When NOT to use

- Input is a continuous-valued regression target or multi-class problem rather than a binary Pass/Fail label
- You need to filter peaks directly from raw chromatograms instead of evaluating already-trained classifiers on labeled data
- Only a single classifier is available — Friedman's/Nemenyi's tests require comparing multiple candidate models across repeated folds

## Inputs

- Labeled binary dataset of 500 expert-annotated Pass/Fail LC-MS peaks from Metabolomics Workbench ST000726
- Peak-quality metric matrices (M4, M7, M11 feature sets) for each peak
- Out-of-fold predicted class probabilities from each candidate classifier under 5-fold x 10-repeat cross-validation

## Outputs

- Cross-validation evaluation table with numeric values for 7 performance measures x 24 classifiers
- Per-repeat performance statistics suitable for Friedman's test
- Friedman-Nemenyi p-values and post-hoc ranking identifying the global best-performing classifier

## How to apply

Train each candidate classifier (e.g. 24 combinations of three peak-quality metric sets M4/M7/M11 and eight caret algorithms) under a fivefold cross-validation repeated 10 times, binarizing predictions at the probability threshold that maximizes balanced accuracy in each fold. Concatenate the out-of-fold predictions and compute Accuracy, Balanced Accuracy, sensitivity (recallPass), specificity (recallFail), PPV (precisionPass), NPV (precisionFail), and F-measures for the Pass and Fail classes. Collect the per-repeat performance statistics and apply Friedman's test followed by Nemenyi's post-hoc correction (scmamp R package) in two rounds — intra-metric-set then inter-metric-set — to identify the statistically best-performing classifier. Validate by reproducing the full evaluation table (7 measures x 24 classifiers) and confirming that Friedman-Nemenyi identifies the same winner (AdaBoost with M11, p = 0.001–0.005 for Pass and p = 0.0023 for Fail) as reported.

## Related tools

- **MetaClean** (Provides the metric matrices (M4, M7, M11) and candidate classifiers whose cross-validated predictions are scored; the evaluation measures are computed on these predictions.) — https://CRAN.R-project.org/package=MetaClean
- **caret** (Supplies the eight classification algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, Linear SVM, AdaBoost, Model-Averaged Neural Network, Random Forest) and the fivefold cross-validation machinery used to generate out-of-fold predictions.)
- **scmamp** (Performs Friedman's test followed by Nemenyi's post-hoc correction on the collected per-repeat performance statistics to select the best-performing classifier.)
- **MLmetrics** (Computes the individual classification measures (Accuracy, Balanced Accuracy, recall, precision, F-measure) on the concatenated out-of-fold predictions.)

## Examples

```
library(scmamp); friedmanTest(perf_stats); nemenyiTest(perf_stats)
```

## Evaluation signals

- A full numeric table exists with 7 measures for all 24 candidate classifiers — no missing cells
- Friedman-Nemenyi analysis reproduces AdaBoost with the M11 metric set as the statistically best performer (Pass p = 0.001–0.005, Fail p = 0.0023)
- Binarization threshold per fold equals the probability threshold that maximizes balanced accuracy in that fold
- Predictions are concatenated across folds before metric computation, so every labeled peak is scored exactly once per repeat
- Both Friedman's and Nemenyi's tests are applied in the specified two-round order: intra-metric-set then inter-metric-set

## Limitations

- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one (Pass), partly due to the smaller number of low-quality peaks in the development set
- Many misclassified low-quality peaks were low-abundance and hard to distinguish from background noise; the development dataset lacked many low-abundance Pass examples
- Characteristics of the LC–MS run (ionization mode, platform, biological matrix) affect peak shape and quality, so the evaluation must account for dataset similarity when generalizing the winner

## Evidence

- [methods] binarizing predictions at the probability threshold that maximizes balanced accuracy in each fold: [evidence span withheld — anchor status: not_found]
- [methods] We employed a rigorous fivefold cross-validation (CV) procedure (Arlot and Celisse 2010) for training and assessing the performance of these candidate classifiers: "We employed a rigorous fivefold cross-validation (CV) procedure (Arlot and Celisse 2010) for training and assessing the performance of these candidate classifiers"
- [methods] we applied Friedman's statistical test, followed by Nemenyi's test, to the collected performance statistics (Demsar 2006): "we applied Friedman’s statistical test, followed by Nemenyi’s test, to the collected performance statistics (Demsar 2006)"
- [results] the performance of the 24 classifiers, i.e., combinations of three metric sets and eight classification algorithms, obtained through the CV procedure applied to the full set of peaks in the: "the performance of the 24 classifiers, i.e., combinations of three metric sets and eight classification algorithms, obtained through the CV procedure applied to the full set of peaks in the"
- [results] we identified the global peak quality classifier through two rounds of statistical comparison of candidate classifier performance, namely an intra-metric set round, followed by an inter-metric set: "we identified the global peak quality classifier through two rounds of statistical comparison of candidate classifier performance, namely an intra-metric set round, followed by an inter-metric set"
