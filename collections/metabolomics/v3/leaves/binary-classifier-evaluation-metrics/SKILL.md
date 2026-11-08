---
name: binary-classifier-evaluation-metrics
description: Reference for scoring any binary classifier on expert Pass/Fail labels
  — e.g. MetaClean peak-quality predictions on an independent LC–MS test set such
  as ST000695.
when_to_use_negative:
- The input is already a feature/peak table that has not yet been scored by a trained
  classifier — this skill only evaluates predictions, it does not classify peaks.
- There are more than two outcome classes (Pass/Fail) — F-measures here are per-class
  for binary labels only.
- Reference labels come from a different expert or labeling scheme than the training
  set, since the article notes the ionization mode, analytical platform, and biological
  matrix affect peak shape and quality, and metrics must be interpreted against comparable
  labels.
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MLmetrics
  role: Computes Accuracy and per-class F-measures from predicted vs expert labels
- name: MetaClean
  role: Provides getPredicitons (caret predict) to produce the Pass/Fail labels being
    scored, plus calculateEvaluationMeasures for the metric computation
  repo: https://CRAN.R-project.org/package=MetaClean
- name: caret
  role: Wraps the AdaBoost.M1 model whose binarized predictions at the CV-optimal
    threshold are evaluated
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:32ff10247799ed5e87baaa16c5093aa86ebc946093eb30a9cbf208324acad19f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MLmetrics
  - MetaClean
  - caret
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# binary-classifier-evaluation-metrics

## Summary

Evaluate a binary peak-quality classifier (Pass vs Fail) using Accuracy and per-class F-measures against expert labels, and verify performance exceeds a random-label permutation baseline. In the MetaClean framework this is how the AdaBoost + M11 global classifier's generalization to independent test sets (e.g. ST000695) is quantified.

## When to use

Reference for scoring any binary classifier on expert Pass/Fail labels — e.g. MetaClean peak-quality predictions on an independent LC–MS test set such as ST000695.

## When NOT to use

- The input is already a feature/peak table that has not yet been scored by a trained classifier — this skill only evaluates predictions, it does not classify peaks.
- There are more than two outcome classes (Pass/Fail) — F-measures here are per-class for binary labels only.
- Reference labels come from a different expert or labeling scheme than the training set, since the article notes the ionization mode, analytical platform, and biological matrix affect peak shape and quality, and metrics must be interpreted against comparable labels.

## Inputs

- Per-peak Pass/Fail or probabilistic predictions from the peak-quality classifier (e.g. AdaBoost M11 outputs for the 500-peak ST000695 Test 1 matrix of 11 M11 metrics)
- Expert-assigned Pass (high-quality) / Fail (low-quality) reference labels for each peak

## Outputs

- Accuracy for the full peak set
- F-measure for the Pass (high-quality) class
- F-measure for the Fail (low-quality) class
- Permutation-test comparison showing observed metrics exceed the 100 random-labeling chance counterparts

## How to apply

Take the classifier's probabilistic predictions for each peak (e.g. from the AdaBoost M11 model trained with nIter=150 via caret's Adaboost.M1) and binarize them at the balanced-accuracy-optimal probability threshold determined during fivefold CV on the development set, using MetaClean::getPredicitons (caret predict) to assign Pass/Fail labels. Compare these predicted labels against the expert (Petrick) Pass/Fail reference labels using calculateEvaluationMeasures/MLmetrics to compute Accuracy plus separate F-measures for the Pass and Fail classes. Because Accuracy and Pass F-measure alone can mask poor failure detection — MetaClean was conservative on the Fail class for Test 1 and 2 — always report the Fail F-measure as a separate signal. Validate that the observed metrics exceed the counterparts from 100 randomly relabeled training permutations applied to the same test set, and, when comparing candidate classifiers across metric sets, use Friedman's test followed by Nemenyi's test on the collected performance statistics rather than raw metric comparison.

## Related tools

- **MLmetrics** (Computes Accuracy and per-class F-measures from predicted vs expert labels)
- **MetaClean** (Provides getPredicitons (caret predict) to produce the Pass/Fail labels being scored, plus calculateEvaluationMeasures for the metric computation) — https://CRAN.R-project.org/package=MetaClean
- **caret** (Wraps the AdaBoost.M1 model whose binarized predictions at the CV-optimal threshold are evaluated)

## Examples

```
results <- calculateEvaluationMeasures(preds, labels) # preds from getPredicitons(model, X_test); labels = expert Pass/Fail for ST000695 peaks
```

## Evaluation signals

- Reported Accuracy on Test 1 (ST000695) should be 0.81; deviations suggest misapplication of the binarization threshold or label mapping.
- Pass F-measure should be 0.87 and Fail F-measure 0.62 for the global classifier on Test 1 (same platform as the development set).
- Both class F-measures and Accuracy must exceed the 100-permutation random-chance counterparts computed by randomly permuting development labels and retraining.
- Per-peak Pass/Fail predictions should be a vector of length equal to the number of Test 1 peaks (500) after threshold binarization.
- When comparing candidates, Friedman–Nemenyi tests (e.g. p = 0.001 and 0.005 for Pass F-measure, p = 0.0023 and 0.0023 for Fail F-measure in favor of AdaBoost M11) should be used instead of raw scatter comparisons.

## Limitations

- The classifier was conservative and performed worse for the Fail class than the Pass class on Test 1 and Test 2, partly because the development set contained fewer low-quality peak examples.
- Many misclassified low-quality peaks were low-abundance and hard to distinguish from background noise; the development dataset lacked many low-abundance Pass examples.
- Metrics must be interpreted in light of the LC–MS run characteristics (ionization mode, analytical platform/method, biological matrix), which affect peak shape and ultimately integration quality.
- The article does not report exact per-dataset metric values for all 24 candidate classifiers, so replicating those specific intermediate numbers requires supplementary material.

## Evidence

- [methods] Reported Accuracy = 0.81 with F-measure Pass = 0.87 and Fail = 0.62 for Test 1 (ST000695); classifier performance must exceed the 100-permutation random-chance counterparts.: "We used an untargeted metabolomics data set, a rigorous cross-validation framework, and appropriate classification evaluation measures and statistical tests to determine the best-performing classifier"
- [methods] Expert reference labels are needed as ground truth for computing Accuracy and per-class F-measures.: "An expert (Petrick) then assigned a label of Pass (high-quality) or Fail (low-quality) to each peak"
- [methods] Friedman–Nemenyi tests are the correct statistical comparison for candidate classifier metrics.: "we applied Friedman’s statistical test, followed by Nemenyi’s test, to the collected performance statistics (Demsar 2006)"
- [methods] Random-label permutation establishes the chance baseline that observed metrics must exceed.: "randomly permuting the labels of the development dataset 100 times and training the best-performing classification approach on each of these randomized datasets"
- [discussion] Fail-class performance is systematically weaker, so the Fail F-measure must be reported separately.: "it was conservative and performed worse for the low-quality class (Fail) than the high-quality one (Pass) for Test 1 and 2"
