---
name: random-chance-baseline-performance-evaluation
description: Use when a trained Pass/Fail or other binary classifier's test-set performance
  must be shown to exceed chance before it is trusted for real filtering decisions.
when_to_use_negative:
- The classifier's performance has already been benchmarked against an established
  null (e.g. published random-permutation results for the same metric set and datasets)
  and no re-run is needed.
- The datasets lack labeled examples or the label space has more than two classes,
  making random label permutation ill-defined for this Pass/Fail setup.
- Only a single train/test split exists with no labeled development set to permute,
  so a randomized counterpart cannot be trained.
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: caret
  role: Trains the AdaBoost M11 classification approach on each of the 100 randomly
    permuted label versions of the development dataset and evaluates performance measures
- name: MetaClean
  role: Supplies the best-performing classifier (AdaBoost with the M11 metric set)
    and the 12 peak-quality metrics whose real performance is benchmarked against
    the random-chance counterparts
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:2357620a84986661cc3abb9e942452cbda3ca49d53ea43b93f91d5d6e122226d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - caret
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# random-chance-baseline-performance-evaluation

## Summary

Establishes a random-chance baseline by training the best-performing classifier (e.g. AdaBoost with the M11 metric set in MetaClean) on 100 randomly permuted label versions of the development dataset, then compares its real test-set performance against the averaged randomized counterparts to rule out chance.

## When to use

Use when a trained Pass/Fail or other binary classifier's test-set performance must be shown to exceed chance before it is trusted for real filtering decisions.

## When NOT to use

- The classifier's performance has already been benchmarked against an established null (e.g. published random-permutation results for the same metric set and datasets) and no re-run is needed.
- The datasets lack labeled examples or the label space has more than two classes, making random label permutation ill-defined for this Pass/Fail setup.
- Only a single train/test split exists with no labeled development set to permute, so a randomized counterpart cannot be trained.

## Inputs

- 500-peak development dataset with expert Pass/Fail labels (Metabolomics Workbench ST000726)
- Four independent test datasets (Test 1 = ST000695, Test 2 = MTBLS354, Test 3 and Test 4 from MTBLS306)
- Best-performing classifier configuration (AdaBoost with the M11 metric set)
- caret R package training workflow

## Outputs

- 100 randomized counterpart classifiers
- Averaged random-chance Accuracy, Pass F-measure, and Fail F-measure per test set
- Real classifier's Accuracy, Pass F-measure, and Fail F-measure per test set for comparison
- Verdict on whether the real classifier outperforms random chance

## How to apply

Train the best-performing classification approach — in the MetaClean case, AdaBoost paired with the M11 metric set — on the full 500-peak development dataset (Metabolomics Workbench ST000726) with expert Pass/Fail labels. Randomly permute the Pass/Fail labels of that same development dataset 100 times and train the identical classification approach on each randomized version using the caret package in R. Apply all 100 randomized counterpart classifiers to the four independent test sets (ST000695, MTBLS354, and the two MTBLS306-derived sets) and compute Accuracy, Pass F-measure, and Fail F-measure for each; average these measures across counterparts to obtain the random-chance baseline per test set. Evaluate the real global classifier on the same four test sets with identical measures and judge success by whether its Accuracy and Pass/Fail F-measures on each test set exceed the corresponding averaged random-permutation measures.

## Related tools

- **caret** (Trains the AdaBoost M11 classification approach on each of the 100 randomly permuted label versions of the development dataset and evaluates performance measures)
- **MetaClean** (Supplies the best-performing classifier (AdaBoost with the M11 metric set) and the 12 peak-quality metrics whose real performance is benchmarked against the random-chance counterparts) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
train the AdaBoost M11 classifier via caret on the full 500-peak ST000726 development dataset, then repeat with 100 label permutations and compare average test-set measures
```

## Evaluation signals

- Real classifier's Accuracy on each of the four test sets exceeds the corresponding averaged random-permutation Accuracy
- Real classifier's Pass F-measure on each test set exceeds the averaged random-permutation Pass F-measure
- Real classifier's Fail F-measure on each test set exceeds the averaged random-permutation Fail F-measure
- All 100 randomized counterparts trained on identical peak-quality metrics (M11) with identical algorithms (AdaBoost) via caret, differing only in label permutation
- Comparison uses the same measures (Accuracy, Pass F-measure, Fail F-measure) for both real and randomized classifiers

## Limitations

- Requires an expert-labeled development set (500 peaks) to permute; performance on the four independent test sets is judged against that single labeled source, so results are contingent on those labels
- The real classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality (Pass) class on Test 1 and Test 2, so the chance-comparison verdict for the Fail class may be less clear-cut
- MetaClean requires an XCMS object input, so datasets produced by other peak-picking software cannot feed this evaluation without conversion
- Dataset characteristics such as ionization mode, analytical platform/method, and biological matrix affect peak shape and quality and must be considered when generalizing the chance-comparison result

## Evidence
