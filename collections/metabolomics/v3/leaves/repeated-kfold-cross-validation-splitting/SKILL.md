---
name: repeated-kfold-cross-validation-splitting
description: Use when a log2-transformed metabolomics data matrix must be split into
  paired training/test sets so prediction performance (accuracy, precision, sensitivity,
  specificity) can be estimated and averaged over multiple resampled partitions.
when_to_use_negative:
- Input is raw MS data requiring pre-processing (noise reduction, peak matching, retention
  time correction) — split only after log2 transformation of the processed feature
  table
- A single hold-out split or leave-one-out design is specified by the study protocol
  instead of repeated k-fold resampling
- The dataset is being used only for unsupervised exploration (e.g., PCA separation
  of SAT and VAT), where train/test splitting is not needed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: caret
  role: creates the repeated k-fold (k = 4, 5 repeats) training/test partitions used
    for PLS classification model evaluation
- name: R
  role: environment in which all splitting, processing, and evaluation analyses were
    performed
  repo: https://www.r-project.org/
- name: lmm2met
  role: processes the training sets (LMM strategy with (1|Id) random intercept) after
    splitting, before model building
  repo: https://github.com/kwanjeeraw/lmm2met
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:dfe4e885bfb5b0e51e3c6eb0578d2aa8f3261be3098c7fa0e8819da41f122114
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - lmm2met
  - mixOmics
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# repeated-kfold-cross-validation-splitting

## Summary

Partitions a metabolomics dataset into training and test sets using repeated k-fold cross-validation (k = 4, repeated five times) via the R caret package, yielding 20 train/test groups at roughly 75%/25% splitting so that downstream PLS-DA model performance can be averaged over many partitions.

## When to use

Use when a log2-transformed metabolomics data matrix must be split into paired training/test sets so prediction performance (accuracy, precision, sensitivity, specificity) can be estimated and averaged over multiple resampled partitions.

## When NOT to use

- Input is raw MS data requiring pre-processing (noise reduction, peak matching, retention time correction) — split only after log2 transformation of the processed feature table
- A single hold-out split or leave-one-out design is specified by the study protocol instead of repeated k-fold resampling
- The dataset is being used only for unsupervised exploration (e.g., PCA separation of SAT and VAT), where train/test splitting is not needed

## Inputs

- log2-transformed metabolomics data matrix (e.g., lung GC-TOF MS metabolites or adipose UPLC-QTOF MS lipids)
- sample class labels / patient metadata for stratification
- R environment with the caret package installed

## Outputs

- 20 training/test set pairs per dataset (~75%/25% split per pair)
- per-split predicted classes for held-out test sets
- averaged performance metrics (accuracy, precision, sensitivity, specificity) across the 20 subsets

## How to apply

Use the caret package in R to partition each log2-transformed metabolomics matrix (e.g., the lung GC-TOF MS data from Metabolomics Workbench PR000305/ST000390) into training and test sets with repeated k-fold cross-validation, k = 4, repeated five times, producing 20 training/test groups with approximately 75%/25% splitting. For each split, process the training set according to the comparison strategy (M0 original, LMM via lmm2met fitLmm with a (1|Id) random intercept, or ML multilevel via mixOmics) before model building. Train PLS classification models on VIP > 1 discriminative features selected from PLS-DA/OPLS-DA of each training set, then predict the corresponding held-out test set. Rationale: averaging performance metrics (accuracy, precision, sensitivity, specificity) over the 20 train/test subsets gives a stable estimate of predictive power, enabling the LMM vs M0 vs ML comparison reported in Figure S-4.

## Related tools

- **caret** (creates the repeated k-fold (k = 4, 5 repeats) training/test partitions used for PLS classification model evaluation)
- **R** (environment in which all splitting, processing, and evaluation analyses were performed) — https://www.r-project.org/
- **lmm2met** (processes the training sets (LMM strategy with (1|Id) random intercept) after splitting, before model building) — https://github.com/kwanjeeraw/lmm2met

## Examples

```
library(caret); folds <- createMultiFolds(y, k = 4, times = 5); for (i in seq_along(folds)) { train <- dat[folds[[i]], ]; test <- dat[-folds[[i]], ] }
```

## Evaluation signals

- Number of train/test groups equals 20 (k = 4 folds x 5 repetitions) per dataset
- Each training set is approximately 75% of samples and each test set approximately 25%
- Metrics (accuracy, precision, sensitivity, specificity) are reported as averages over all 20 subsets
- Resulting averaged performance table reproduces the Figure S-4 pattern: LMM superior to M0 and ML on accuracy, precision, and sensitivity, with ML slightly better than LMM only on GC-TOF MS specificity
- Splitting is done on log2-transformed data as described in the methods

## Limitations

- Repeated k-fold estimates are specific to these datasets; further experiments are recommended to validate the results
- On the GC-TOF MS dataset the LMM method did not outperform the ML method for specificity, so averaged metrics can favor one strategy on some metrics and another on others
- LMM-based training-set processing may introduce false positives due to increased sensitivity

## Evidence

- [methods] repeated k-fold partition parameters: "each dataset was partitioned into training and test sets with a repeated k-fold cross validation [31] (k = 4)"
- [methods] caret used for evaluation: "The evaluation processes were carried out using the caret package"
- [methods] VIP threshold for feature selection: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [methods] log2 transformation precondition: "Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study."
- [discussion] validation gap caveat: "Therefore, further experiments are recommended to validate the results."
