---
name: cross-platform-classifier-generalization-evaluation
description: Use when a machine-learning peak-quality classifier trained on one LC-MS
  platform/dataset must be tested on independent datasets from a different platform
  or ionization mode, and cross-dataset generalization (Accuracy, Pass/Fail F-measure)
  needs to be quantified.
when_to_use_negative:
- The classifier has not yet been trained — this skill only evaluates a pre-trained
  model; use the MetaClean training workflow (fivefold CV on the development dataset)
  first.
- Only internal, same-platform validation is needed (e.g. Test 1 generated on the
  same platform as the development set) — this skill targets cross-platform transfer
  evaluation.
- Input is vendor-processed, non-XCMS-formatted output; MetaClean requires an XCMS
  object input and is not amenable to uniquely formatted vendor-processed outputs.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaClean
  role: Provides the pre-trained AdaBoost+M11 classifier and the M11 peak-quality
    metric computation applied to each cross-platform test dataset
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Pre-processes the external test datasets (peak-picking, retention time correction,
    grouping) before metric computation
- name: IPO
  role: Optimizes XCMS parameters for each test dataset so cross-platform peak tables
    are generated with comparable settings
- name: fastAdaboost
  role: Supplies the AdaBoost algorithm underlying the M11-based global peak quality
    classifier being evaluated
- name: caret
  role: Supports classifier evaluation and performance-measure computation during
    cross-platform testing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:f820b4b7990024c8b811a2f406020084c5c5f9ab41447753812e1cc812cb2e11
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - XCMS
  - caret
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-platform-classifier-generalization-evaluation

## Summary

Evaluate a pre-trained MetaClean AdaBoost+M11 peak-quality classifier on independent test sets generated on a different analytical platform (Test 2 on MTBLS354; Tests 3-4 on MTBLS306 in positive/negative ionization mode), measuring Accuracy plus Pass and Fail F-measures against expert Pass/Fail labels to demonstrate cross-platform generalization.

## When to use

Use when a machine-learning peak-quality classifier trained on one LC-MS platform/dataset must be tested on independent datasets from a different platform or ionization mode, and cross-dataset generalization (Accuracy, Pass/Fail F-measure) needs to be quantified.

## When NOT to use

- The classifier has not yet been trained — this skill only evaluates a pre-trained model; use the MetaClean training workflow (fivefold CV on the development dataset) first.
- Only internal, same-platform validation is needed (e.g. Test 1 generated on the same platform as the development set) — this skill targets cross-platform transfer evaluation.
- Input is vendor-processed, non-XCMS-formatted output; MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs.

## Inputs

- Pre-trained AdaBoost+M11 MetaClean classifier (global peak quality classifier)
- XCMS peak table / XCMS object from an external test dataset (e.g. MTBLS354, MTBLS306 positive mode, MTBLS306 negative mode)
- Peak-quality metric matrix (M11 set: 7 TargetedMSQC-derived metrics + 4 Zhang et al. metrics) for each test dataset
- Expert-assigned Pass/Fail labels for test-set peaks (ground truth)

## Outputs

- Per-test-set classification Accuracy on the external cross-platform dataset
- Pass-class F-measure and Fail-class F-measure per test set
- Comparison table/diagnostic of cross-platform performance vs. reported values (Test 2 = 0.65, Test 3 = 0.75, Test 4 = 0.79)

## How to apply

Pre-process each external test dataset (e.g. MTBLS354, and MTBLS306 in positive and negative mode) with XCMS (version 3.6.3) using IPO-optimized peak-picking, retention time correction, and grouping parameters. Compute the 11 peak-quality metrics (the M11 set: seven TargetedMSQC-derived metrics plus four Zhang et al. 2014 metrics) for every peak across samples and pooled QCs, then apply the pre-trained AdaBoost+M11 classifier to that metric matrix. Binarize the probabilistic predictions at the optimized threshold that maximized balanced accuracy during the fivefold cross-validation, and binarize against the expert Pass/Fail labels. Evaluate each test set's Accuracy and per-class F-measures (Pass F-measure and Fail F-measure) and compare against reported cross-platform values (Test 2 Accuracy = 0.65, Test 3 Accuracy = 0.75, Test 4 Accuracy = 0.79). Rationale: this establishes whether the classifier learned on one platform transfers to others, as the original MetaClean evaluation did.

## Related tools

- **MetaClean** (Provides the pre-trained AdaBoost+M11 classifier and the M11 peak-quality metric computation applied to each cross-platform test dataset) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Pre-processes the external test datasets (peak-picking, retention time correction, grouping) before metric computation)
- **IPO** (Optimizes XCMS parameters for each test dataset so cross-platform peak tables are generated with comparable settings)
- **fastAdaboost** (Supplies the AdaBoost algorithm underlying the M11-based global peak quality classifier being evaluated)
- **caret** (Supports classifier evaluation and performance-measure computation during cross-platform testing)

## Examples

```
Rscript evaluate_cross_platform.R --test MTBLS354 --classifier MetaClean_AdaBoost_M11.rds --metrics M11
```

## Evaluation signals

- Computed Accuracy on each cross-platform test set matches reported values: Test 2 = 0.65, Test 3 = 0.75, Test 4 = 0.79 (allow ~0.05 tolerance).
- Pass and Fail F-measures are reported separately for each test set and are consistent with the qualitative performance described in the article.
- All test datasets were pre-processed with XCMS version 3.6.3 using IPO-optimized parameters, and the same M11 metric set (7 TargetedMSQC + 4 Zhang) was used for feature computation.
- Classifier predictions are binarized at the threshold that maximized balanced accuracy during the original fivefold CV on the development set.
- Performance is evaluated against expert-verified Pass/Fail labels, not merely predicted quality flags.

## Limitations

- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs, limiting transfer to non-XCMS pipelines.
- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one, partly because the development dataset contained fewer Fail examples.
- Many misclassified low-quality peaks in Test 2 were low-abundance peaks difficult to distinguish from background noise; the development dataset contained few low-abundance Pass examples.
- Test 2 had fewer peaks passing the RSD < 30% threshold and a smaller IPO peak width parameter, suggesting greater dissimilarity from the development set and a lower cross-platform Accuracy.
- Ionization mode, analytical platform/method, and biological matrix affect peak shape and size, so generalization depends on how similar the test platform is to the training platform.

## Evidence

- [results] cross-platform generalization reported performance: [evidence span withheld — anchor status: not_found]
