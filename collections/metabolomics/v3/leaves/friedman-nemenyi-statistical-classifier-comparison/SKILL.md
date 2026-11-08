---
name: friedman-nemenyi-statistical-classifier-comparison
description: Use when comparing multiple classifiers (e.g. 24 combinations of 3 metric
  sets and 8 algorithms) on repeated cross-validation folds and you need a statistically
  justified single model rather than eyeballing metric scatter plots.
when_to_use_negative:
- When only a single train/test split or one fold is available — Friedman's test requires
  repeated performance statistics across multiple CV runs (Demsar 2006), not a single
  paired comparison.
- When the goal is to compare peak quality filters (e.g. RSD < 30% filtering) rather
  than classification algorithms — that comparison is handled by separate evaluation,
  not Friedman–Nemenyi.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: scmamp
  role: R package used to perform Friedman's test followed by Nemenyi's post-hoc test
    on the collected per-repeat performance statistics
- name: caret
  role: R package used to train and cross-validate the eight classification algorithms
    paired with each peak-quality metric set, producing the 24 candidate classifiers
- name: MLmetrics
  role: R package used to compute the seven performance measures (accuracy, balanced
    accuracy, sensitivity, specificity, PPV, NPV, F-measure) from concatenated out-of-fold
    predictions
- name: MetaClean
  role: R package whose cross-validation pipeline provides the candidate classifiers
    (M4/M7/M11 metric sets × 8 algorithms) and the labeled peak dataset on which the
    Friedman–Nemenyi comparison is run
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:45ac12bb73358c5e097b3a2a979a211193278a71f19df467ed152fba2761692f
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# friedman-nemenyi-statistical-classifier-comparison

## Summary

A rigorous statistical comparison procedure, built on Friedman's test followed by Nemenyi's post-hoc correction (via the scmamp R package), that selects the best-performing classification algorithm among multiple candidates, as used in MetaClean to choose among 24 candidate peak-quality classifiers built from three peak-quality metric sets (M4, M7, M11) and eight caret algorithms.

## When to use

Use when comparing multiple classifiers (e.g. 24 combinations of 3 metric sets and 8 algorithms) on repeated cross-validation folds and you need a statistically justified single best model rather than eyeballing metric scatter plots.

## When NOT to use

- When only a single train/test split or one fold is available — Friedman's test requires repeated performance statistics across multiple CV runs (Demsar 2006), not a single paired comparison.
- When the goal is to compare peak quality filters (e.g. RSD < 30% filtering) rather than classification algorithms — that comparison is handled by separate evaluation, not Friedman–Nemenyi.

## Inputs

- Per-repeat (10 repeats × 5 folds) out-of-fold performance statistics for each candidate classifier (accuracy, balanced accuracy, sensitivity/recallPass, specificity/recallFail, PPV/precisionPass, NPV/precisionFail, F-measure for Pass and Fail)
- Candidate classifier list — here the 24 combinations of three peak-quality metric sets (M4, M7, M11) and eight caret classification algorithms (Decision Tree, Logistic Regression, Naive Bayes, Neural Network, Linear SVM, AdaBoost, Model-Averaged Neural Network, Random Forest)
- Labeled peak data (500 expert-annotated Pass/Fail peaks from Metabolomics Workbench ST000726) from which CV statistics were generated

## Outputs

- Friedman test p-values and Nemenyi post-hoc pairwise comparison results (p-values) across candidate classifiers for each performance measure
- A selected best-performing classifier per metric set (intra-set round) and a single global best classifier (inter-set round), e.g. AdaBoost with the M11 metric set
- A cross-validation evaluation table with numeric values for all 7 performance measures × 24 candidate classifiers

## How to apply

Run the fivefold cross-validation repeated 10 times for each candidate classifier on the 500-peak labeled development dataset (Metabolomics Workbench ST000726), collecting per-repeat performance statistics (accuracy, balanced accuracy, sensitivity, specificity, PPV, NPV, F-measure). First, in the intra-metric-set round, apply Friedman's test followed by Nemenyi's post-hoc test with the scmamp R package to the per-repeat statistics of the eight algorithms within each metric set (M4, M7, M11), selecting the best algorithm per set — this yielded AdaBoost for M4, Random Forest for M7, and AdaBoost for M11. Then, in the inter-metric set round, repeat Friedman–Nemenyi comparison across the three selected per-set winners to identify the global classifier; AdaBoost paired with the M11 metric set was confirmed as significantly better on Pass and Fail F-measures (p = 0.001–0.005 for Pass, p = 0.0023 for Fail). The rationale is that Friedman's test on repeated CV runs respects the paired, non-parametric nature of per-fold statistics, and Nemenyi's correction controls the family-wise error rate across multiple classifier pairs, preventing the false selection of an algorithm that merely wins by chance on a small hold-out. Validate by reproducing the full cross-validation performance table for all 7 measures × 24 candidates and confirming the same two-round Friedman–Nemenyi selection outcome.

## Related tools

- **scmamp** (R package used to perform Friedman's test followed by Nemenyi's post-hoc test on the collected per-repeat performance statistics)
- **caret** (R package used to train and cross-validate the eight classification algorithms paired with each peak-quality metric set, producing the 24 candidate classifiers)
- **MLmetrics** (R package used to compute the seven performance measures (accuracy, balanced accuracy, sensitivity, specificity, PPV, NPV, F-measure) from concatenated out-of-fold predictions)
- **MetaClean** (R package whose cross-validation pipeline provides the candidate classifiers (M4/M7/M11 metric sets × 8 algorithms) and the labeled peak dataset on which the Friedman–Nemenyi comparison is run) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
Rscript -e 'library(scmamp); friedmanTest(perf_stats); nemenyiTest(perf_stats)'
```

## Evaluation signals

- The two-round selection outcome must reproduce the paper's result: AdaBoost wins for M4, Random Forest for M7, and AdaBoost for M11 in the intra-set round, and AdaBoost with the M11 metric set is the global winner in the inter-set round.
- Friedman–Nemenyi p-values for the global winner should match the reported ranges — Pass F-measure p = 0.001–0.005 and Fail F-measure p = 0.0023 — when comparing the best classifier against the remaining candidates.
- The cross-validation table must contain numeric values for all 7 performance measures (accuracy, balanced accuracy, sensitivity, specificity, PPV, NPV, F-measure) for all 24 candidate classifiers, not qualitative descriptions.
- Per-repeat statistics must come from a rigorous fivefold cross-validation procedure repeated 10 times, with probability-threshold binarization chosen per-fold to maximize balanced accuracy, as specified in the MetaClean pipeline.
- Comparison must be applied to the paired per-repeat statistics (not aggregate means) using scmamp's Friedman/Nemenyi implementation, consistent with Demsar 2006 guidance.

## Limitations

- The procedure requires the full matrix of candidate × repeat performance statistics; if some classifiers fail to train on a fold (e.g. Neural Network or Random Forest instability), the Friedman test design is compromised.
- The selection is only as good as the underlying cross-validation — in MetaClean the development set contained few low-abundance Pass peaks, so the chosen global classifier (AdaBoost with M11) was conservative and performed worse for the low-quality (Fail) class on some test sets (Test 1 and 2).
- The Friedman–Nemenyi test treats classifiers as the sole factor and does not account for dataset-level variation beyond the repeated CV design; reapplying MetaClean to MTBLS306 in split ionization-mode configurations was used to validate generalization, but the statistical selection itself is dataset-specific.

## Evidence

- [readme] Package installation: [evidence span withheld — anchor status: not_found]
