---
name: classification-model-label-permutation-testing
description: Use when a supervised classifier has been trained on labeled data (e.g.
  500-peak LC–MS Pass/Fail labels) and you must rule out that reported test-set performance
  is an artifact of random chance rather than genuine signal in the features.
when_to_use_negative:
- When the classifier has already been validated against a real null (e.g. a previously
  reported permutation baseline for the same feature-label scheme) — re-running 100
  permutations adds no information.
- When labels are continuous/regression targets rather than categorical Pass/Fail
  classes — permuting labels and averaging F-measures for Pass/Fail classes is specific
  to classification.
- When the goal is statistical significance testing of differences between candidate
  classifiers (e.g. via Friedman–Nemenyi tests) rather than ruling out chance-level
  performance — that comparison uses a different test family.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: caret
  role: R package used to train the AdaBoost M11 classification approach on each of
    the 100 label-permuted versions of the 500-peak development dataset
- name: MetaClean
  role: Supplies the M11 peak-quality metric set and the AdaBoost global peak-quality
    classifier whose real performance is benchmarked against the random-permutation
    counterparts
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:a5db48608b8210f7005cd13d5f6b6b8e7ecf47a25984d0070ba64433e49da2b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - caret
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Classification model label permutation testing

## Summary

Validates that a trained classifier (here AdaBoost on the M11 peak-quality metric set) captures a real label-to-feature relationship by benchmarking it against 100 counterparts trained on randomly permuted Pass/Fail labels, then confirming the real model outperforms the averaged random-chance baseline on independent test sets.

## When to use

Use when a supervised classifier has been trained on labeled data (e.g. 500-peak LC–MS Pass/Fail labels) and you must rule out that reported test-set performance is an artifact of random chance rather than genuine signal in the features.

## When NOT to use

- When the classifier has already been validated against a real null (e.g. a previously reported permutation baseline for the same feature-label scheme) — re-running 100 permutations adds no information.
- When labels are continuous/regression targets rather than categorical Pass/Fail classes — permuting labels and averaging F-measures for Pass/Fail classes is specific to classification.
- When the goal is statistical significance testing of differences between candidate classifiers (e.g. via Friedman–Nemenyi tests) rather than ruling out chance-level performance — that comparison uses a different test family.

## Inputs

- Full development dataset of ~500 peaks with expert-assigned Pass (high-quality) / Fail (low-quality) labels, pre-processed via XCMS peak-picking, retention time correction, and grouping
- Trained best-performing classifier specification (AdaBoost trained on the M11 metric set) plus its training script/config for the caret package
- Four independent test sets with the same feature structure and labels: Test 1 (MW ST000695), Test 2 (ML MTBLS354), Test 3 and Test 4 (splits of ML MTBLS306)

## Outputs

- 100 randomized counterpart classifiers (one per label permutation of the development set)
- Per-test-set Accuracy, Pass F-measure, and Fail F-measure for each of the 100 counterparts
- Averaged random-chance baseline for Accuracy and Pass/Fail F-measures on each of the four test sets
- Comparison verdict: whether the real classifier's measures exceed the averaged random-permutation measures on each test set

## How to apply

Train the best-performing classifier — for MetaClean this is AdaBoost with the M11 metric set — on the full 500-peak development dataset (Metabolomics Workbench ST000726) with expert Pass/Fail labels using the caret package in R. Randomly permute the Pass/Fail labels of that same development dataset 100 times, retraining the identical AdaBoost M11 classification approach on each randomized copy. Apply every randomized counterpart classifier to the four independent test sets (Test 1 = ST000695; Test 2 = MTBLS354; Tests 3 and 4 split from MTBLS306) and compute Accuracy, Pass F-measure, and Fail F-measure for each. Average each performance measure across all 100 counterparts to obtain the random-chance baseline per test set. Evaluate the real classifier on the same four test sets with identical measures and require that its Accuracy and Pass/Fail F-measures exceed the corresponding averaged random-permutation values on every test set; failure to do so indicates the classifier's performance is attributable to random chance.

## Related tools

- **caret** (R package used to train the AdaBoost M11 classification approach on each of the 100 label-permuted versions of the 500-peak development dataset)
- **MetaClean** (Supplies the M11 peak-quality metric set and the AdaBoost global peak-quality classifier whose real performance is benchmarked against the random-permutation counterparts) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
R: `permuted <- lapply(1:100, function(i) train(quality ~ ., data=permuteLabels(dev), method='ada', trControl=ctrl)); sapply(permuted, evalOnTests)`
```

## Evaluation signals

- Confirm that exactly 100 randomized counterparts were trained, one per label permutation, using the identical AdaBoost M11 configuration.
- Verify that for each of the four test sets (ST000695, MTBLS354, and the two MTBLS306 splits) the real classifier's Accuracy, Pass F-measure, and Fail F-measure each exceed the corresponding 100-counterpart averaged random-permutation measure.
- Check that the averaging of Accuracy and Pass/Fail F-measures is computed across all 100 counterparts per test set, not on a subset.
- Ensure the performance measures are computed on the held-out independent test sets only, never on the development dataset itself.
- Confirm the label-permutation step only shuffles Pass/Fail labels of the development dataset while leaving the M11 metric values untouched.

## Limitations

- The check is a necessary-not-sufficient validation: exceeding the averaged random baseline does not quantify effect size or confidence intervals around the real classifier's performance.
- The baseline is specific to the classifier configuration tested (AdaBoost with M11); swapping metric sets or algorithms requires re-running the full 100-permutation procedure.
- Random-chance baselines were estimated by averaging performance across counterparts, so per-test-set variance across the 100 permutations is not directly reported in the article.

## Evidence

- [methods] Average performance across counterparts per test set: [evidence span withheld — anchor status: not_found]
- [results] Real classifier beats random baseline: [evidence span withheld — anchor status: not_found]
