---
name: binary-classifier-f-measure-computation
description: Use when evaluating a binary classifier (e.g., peak Pass/Fail quality
  in untargeted LC–MS metabolomics) where both high- and low-quality classes are important
  and single accuracy is insufficient.
when_to_use_negative:
- When there is only a single well-populated class and class imbalance is absent —
  simple accuracy would suffice
- When the goal is regression or ranking rather than discrete binary Pass/Fail labeling
- When ground-truth labels are unavailable (e.g., purely unsupervised peak-filtering)
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: caret
  role: R package used to train the AdaBoost classification approach and to compute/extract
    classifier performance measures
- name: MetaClean
  role: R package providing the peak-quality classifiers whose Accuracy and Pass/Fail
    F-measures are being evaluated
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:8dac46610b8bc99b0eaed2b74b552402d68e3a9ff431afcba0aab7b888bce00e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - caret
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# binary-classifier-f-measure-computation

## Summary

Computes Accuracy and per-class (Pass/Fail) F-measures to evaluate binary peak-quality classifiers in the MetaClean framework, where classifier predictions of high- vs. low-quality peaks are compared against expert Pass/Fail labels.

## When to use

Use when evaluating a binary classifier (e.g., peak Pass/Fail quality in untargeted LC–MS metabolomics) where both high- and low-quality classes are important and single accuracy is insufficient.

## When NOT to use

- When there is only a single well-populated class and class imbalance is absent — simple accuracy would suffice
- When the goal is regression or ranking rather than discrete binary Pass/Fail labeling
- When ground-truth labels are unavailable (e.g., purely unsupervised peak-filtering)

## Inputs

- Predicted binary labels from a trained peak-quality classifier (Pass/Fail per peak)
- Expert-assigned ground-truth labels (Pass/Fail per peak) for the same evaluation peaks
- Trained classifier object (e.g., caret AdaBoost model using the M11 metric set)

## Outputs

- Overall Accuracy value for the classifier on the evaluation set
- F-measure for the Pass (high-quality) class
- F-measure for the Fail (low-quality) class

## How to apply

Fit the binary classifier (e.g., AdaBoost with the M11 metric set trained via caret on the 500-peak development dataset ST000726) on labeled peaks where experts assigned Pass (high-quality) or Fail (low-quality) labels. Apply the fitted model to held-out data (e.g., Test 1 = ST000695, Test 2 = MTBLS354, Test 3/4 = MTBLS306) and compare predicted labels to the expert labels. Compute Accuracy (fraction correctly labeled) and per-class F-measures separately for the Pass and Fail classes, using the F-measure as the harmonic mean of precision and recall for each class. Because the Fail class is often rarer and harder to classify (the classifier was conservative on Test 1 and 2), report Pass and Fail F-measures independently rather than relying only on accuracy. Compare results against a random-chance baseline (e.g., averaged over 100 label-permutation classifiers) or against the RSD < 30% QC-filtering alternative to judge whether the classifier adds real value.

## Related tools

- **caret** (R package used to train the AdaBoost classification approach and to compute/extract classifier performance measures)
- **MetaClean** (R package providing the peak-quality classifiers whose Accuracy and Pass/Fail F-measures are being evaluated) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
library(MetaClean); preds <- MetaClean_predict(model = adaboost_m11_model, xcmsObj = test_xcms); f1 <- caret::confusionMatrix(preds, expert_labels, mode = "everything")
```

## Evaluation signals

- F-measure for Pass and Fail classes should be computed separately — a classifier with high Accuracy but low Fail F-measure (as on Test 1 and 2) is flagged as conservative
- Real classifier's Accuracy and Pass/Fail F-measures should exceed the averaged random-permutation baseline across 100 permuted classifiers on each test set
- Compare against RSD < 30% QC-filtering performance to verify the ML classifier outperforms the traditional filtering alternative
- Confirm per-class F-measures are reported (not just a single aggregate) when class sizes differ between Pass and Fail

## Limitations

- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one (Pass) on Test 1 and Test 2, partly because the development set contained fewer low-quality peaks
- Many misclassified low-quality peaks in Test 2 were low abundance and hard to distinguish from background noise; the development dataset had few low-abundance Pass examples
- Exact numeric performance values for Test 1 (Accuracy and Pass/Fail F-measures) are described only qualitatively in the paper

## Evidence

- [methods] average the performance measure values across all 100 counterparts to obtain the random-chance performance for each test set: [evidence span withheld — anchor status: not_found]
