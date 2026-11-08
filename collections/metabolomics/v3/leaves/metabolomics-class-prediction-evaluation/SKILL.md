---
name: metabolomics-class-prediction-evaluation
description: Use when you need to benchmark whether a metabolomics data-processing
  strategy (e.g., LMM fitting vs. original data vs. multilevel PLS-DA) improves downstream
  class-prediction performance on labeled samples.
when_to_use_negative:
- The dataset lacks known class labels (e.g., malignant vs. non-malignant), so classification
  models and prediction metrics cannot be trained or assessed
- The goal is only to characterize variance reduction or separation quality after
  LMM fitting (use PCA of SAT/VAT separation instead of PLS classification)
- You need PLS-DA on the ML-processed training set from an alternative package — the
  workflow specifies mixOmics for multilevel PLS-DA, and OPLS-DA is only run on M0-
  and LMM-processed sets
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: Produces the LMM-processed training-set variant (fitLmm with (1|Id)) whose
    selected features are benchmarked for prediction performance
  repo: https://github.com/kwanjeeraw/lmm2met
- name: caret
  role: Repeated k-fold (k = 4, five repeats) train/test partitioning and training/prediction
    of the PLS classification model; computes performance metrics
- name: mixOmics
  role: PLS-DA on M0- and LMM-processed training sets and multilevel-based PLS-DA
    (ML strategy) used for VIP-based feature selection
- name: ropls
  role: Performs OPLS-DA on the M0- and LMM-processed training sets as an additional
    feature-selection strategy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:8f073f3ff94b4d4454380a98b85aed1140e83d3a63fb0dd041de9439f8b1dca1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - caret
  - lmm2met
  - mixOmics
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-class-prediction-evaluation

## Summary

Evaluates and compares the predictive power of discriminative metabolites selected from PLS-DA/OPLS-DA (with and without LMM processing) and multilevel PLS-DA by training PLS classification models and measuring accuracy, precision, sensitivity and specificity on held-out test sets.

## When to use

Use when you need to benchmark whether a metabolomics data-processing strategy (e.g., LMM fitting vs. original data vs. multilevel PLS-DA) improves downstream class-prediction performance on labeled samples.

## When NOT to use

- The dataset lacks known class labels (e.g., malignant vs. non-malignant), so classification models and prediction metrics cannot be trained or assessed
- The goal is only to characterize variance reduction or separation quality after LMM fitting (use PCA of SAT/VAT separation instead of PLS classification)
- You need PLS-DA on the ML-processed training set from an alternative package — the workflow specifies mixOmics for multilevel PLS-DA, and OPLS-DA is only run on M0- and LMM-processed sets

## Inputs

- log2-transformed metabolomics feature table with class labels (e.g., GC-TOF MS lung tissue metabolites from Metabolomics Workbench PR000305/ST000390)
- Processing-variant training sets: M0 (original), LMM-processed (lmm2met fitLmm with (1|Id) random intercept), and ML (multilevel via mixOmics)
- Patient/sample metadata with subject characteristics (e.g., Table S-1 metadata of 39 early-stage lung adenocarcinoma patients)

## Outputs

- Averaged performance table of accuracy, precision, sensitivity and specificity per processing condition (M0, LMM, ML) over 20 train/test subsets
- List of discriminative metabolites with VIP > 1 per strategy
- Comparison figure of prediction performance (analogous to Figure S-4)

## How to apply

Partition each log2-transformed metabolomics dataset (e.g., lung GC-TOF MS data from Metabolomics Workbench PR000305/ST000390) into training and test sets using repeated k-fold cross-validation (k = 4, repeated five times) via the caret package, yielding 20 train/test groups (~75%/25% split). Build PLS-DA (mixOmics) and OPLS-DA (ropls) models on the training sets, select discriminative metabolites with variable importance in projection score VIP > 1, and train a PLS classification model on those features with caret. Predict the held-out test sets and compute accuracy, precision, sensitivity and specificity, averaging estimates over the 20 train/test subsets. Validate by checking that the averaged performance table reproduces the expected pattern — for the lung GC-TOF MS data, LMM outperformed M0 and ML on all metrics except specificity, for which ML was slightly better (overall performance ~80%).

## Related tools

- **lmm2met** (Produces the LMM-processed training-set variant (fitLmm with (1|Id)) whose selected features are benchmarked for prediction performance) — https://github.com/kwanjeeraw/lmm2met
- **caret** (Repeated k-fold (k = 4, five repeats) train/test partitioning and training/prediction of the PLS classification model; computes performance metrics)
- **mixOmics** (PLS-DA on M0- and LMM-processed training sets and multilevel-based PLS-DA (ML strategy) used for VIP-based feature selection)
- **ropls** (Performs OPLS-DA on the M0- and LMM-processed training sets as an additional feature-selection strategy)

## Examples

```
install_github('kwanjeeraw/lmm2met'); folds <- createMultiFolds(y, k=4, times=5)  # then PLS-DA (mixOmics/ropls), select VIP>1 features, train and predict with caret
```

## Evaluation signals

- The workflow yields exactly 20 train/test groups (k = 4 folds repeated five times) with ~75%/25% splitting
- Only features with VIP > 1 enter the final PLS classification model
- Averaged performance metrics (accuracy, precision, sensitivity, specificity) are reported for every processing condition (M0, LMM, ML) on both UPLC-QTOF MS and GC-TOF MS datasets
- Reproduces the reported pattern: LMM superior on accuracy, precision and sensitivity on both datasets; ML slightly better than LMM for GC-TOF MS specificity; overall performance ~80%
- Predictions are always assessed on the held-out test set corresponding to each training split, never on the training data

## Limitations

- The ML method slightly outperformed LMM on specificity for the GC-TOF MS dataset
- Improved sensitivity of the LMM approach may introduce false positives
- Performance claims are qualified as being at the current (preliminary) stage; the authors recommend further experiments to validate the results

## Evidence

- [methods] The significant features were selected based on variable importance in projection score (VIP > 1): "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [methods] each dataset was partitioned into training and test sets with a repeated k-fold cross validation: "each dataset was partitioned into training and test sets with a repeated k-fold cross validation [31] (k = 4)"
- [discussion] superior accuracy, precision, sensitivity and specificity than the M0 and ML methods in both UPLC-QTOF MS and GC-TOF MS datasets: "superior accuracy, precision, sensitivity and specificity than the M0 and ML methods in both UPLC-QTOF MS and GC-TOF MS datasets"
- [discussion] except specificity of GC-TOF MS dataset, for which the ML method was slightly better than the LMM method: "except specificity of GC-TOF MS dataset, for which the ML method was slightly better than the LMM method"
- [methods] The evaluation processes were carried out using the caret package: "The evaluation processes were carried out using the caret package"
