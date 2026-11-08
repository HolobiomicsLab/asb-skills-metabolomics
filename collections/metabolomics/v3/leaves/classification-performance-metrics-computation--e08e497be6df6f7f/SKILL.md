---
name: classification-performance-metrics-computation
description: Use when class predictions from a PLS/OPLS-DA-derived classifier on held-out
  test folds (e.g. malignant vs non-malignant lung GC-TOF MS tissue) must be quantified
  by accuracy, precision, sensitivity and specificity.
when_to_use_negative:
- The input is a metabolite intensity matrix or fitted LMM output rather than classifier
  predictions paired with reference labels — metric computation requires predicted
  vs observed classes
- The goal is exploratory QC of the data (e.g. variance reduction or PCA separation
  after LMM fitting), not supervised prediction benchmarking
- The classifier was not trained on VIP > 1 discriminative features from a PLS-DA/OPLS-DA
  strategy, so results are not comparable to the article's M0/LMM/ML benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3368
tools:
- name: caret
  role: Trains the PLS classification model, predicts held-out test sets, and carries
    out the performance evaluation (accuracy, precision, sensitivity, specificity)
- name: lmm2met
  role: Provides the LMM-processed (fitLmm with random intercept (1|Id)) training
    sets whose classifier performance is benchmarked
  repo: https://github.com/kwanjeeraw/lmm2met
- name: mixOmics
  role: Produces multilevel (ML) PLS-DA features and PLS-DA on M0/LMM training sets
    feeding the comparison
- name: ropls
  role: Performs OPLS-DA on M0- and LMM-processed training sets to extract VIP > 1
    features for alternative classification models
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:ddf8723d7e9a85cb1b941e9cc6308eff4ba8027cc405f0f30b127e9dd99e094c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# classification-performance-metrics-computation

## Summary

Computes and compares prediction performance metrics (accuracy, precision, sensitivity, specificity) of PLS classification models built from VIP > 1 discriminative metabolites, to benchmark data-processing strategies (M0, LMM, ML) on log2-transformed metabolomics data.

## When to use

Use when class predictions from a PLS/OPLS-DA-derived classifier on held-out test folds (e.g. malignant vs non-malignant lung GC-TOF MS tissue) must be quantified by accuracy, precision, sensitivity and specificity.

## When NOT to use

- The input is a metabolite intensity matrix or fitted LMM output rather than classifier predictions paired with reference labels — metric computation requires predicted vs observed classes
- The goal is exploratory QC of the data (e.g. variance reduction or PCA separation after LMM fitting), not supervised prediction benchmarking
- The classifier was not trained on VIP > 1 discriminative features from a PLS-DA/OPLS-DA strategy, so results are not comparable to the article's M0/LMM/ML benchmark

## Inputs

- Predicted class labels from a PLS classification model (caret) on each held-out test set of the 20 train/test groups
- Ground-truth class labels for the test samples (e.g. malignant vs non-malignant lung tissue; SAT vs VAT)
- Processing-strategy identifier for each model (M0, LMM, ML) per dataset (PR000305/ST000390 lung; PR000058/ST000081 adipose)

## Outputs

- Averaged accuracy, precision, sensitivity and specificity per processing strategy (M0, LMM, ML) per dataset
- Comparison table of performance metrics across the 20 train/test subsets, comparable to Figure S-4

## How to apply

After building a PLS classification model (caret) on the discriminative metabolites (VIP > 1) extracted from each processing strategy (M0 original, LMM via lmm2met fitLmm, or ML multilevel via mixOmics), predict the corresponding held-out test set from the repeated k-fold CV split (k = 4, repeated five times, giving 20 train/test groups with ~75%/25% splitting). Compute accuracy, precision, sensitivity and specificity for each prediction, then average the estimates across the 20 train/test subsets per processing condition. Compare averaged metrics across strategies per dataset (UPLC-QTOF MS adipose and GC-TOF MS lung). In this article's validation, the LMM method showed superior accuracy, precision and sensitivity versus M0 and ML, and superior specificity on the UPLC-QTOF MS data, with a target overall performance of ~80%; on the GC-TOF MS data the ML method slightly outperformed LMM on specificity.

## Related tools

- **caret** (Trains the PLS classification model, predicts held-out test sets, and carries out the performance evaluation (accuracy, precision, sensitivity, specificity))
- **lmm2met** (Provides the LMM-processed (fitLmm with random intercept (1|Id)) training sets whose classifier performance is benchmarked) — https://github.com/kwanjeeraw/lmm2met
- **mixOmics** (Produces multilevel (ML) PLS-DA features and PLS-DA on M0/LMM training sets feeding the comparison)
- **ropls** (Performs OPLS-DA on M0- and LMM-processed training sets to extract VIP > 1 features for alternative classification models)

## Examples

```
caret::confusionMatrix(factor(pred_labels), factor(test_labels), mode = "everything")
```

## Evaluation signals

- All metrics are averaged over exactly 20 train/test groups (k = 4 repeated five times) per dataset and processing strategy
- Overall prediction performance is in the reported ~80% range for the lung GC-TOF MS data
- Directional pattern check: LMM beats M0 and ML on accuracy, precision and sensitivity on both datasets; LMM beats M0 on specificity for the UPLC-QTOF MS dataset
- Known exception reproduced: ML slightly outperforms LMM on specificity for the GC-TOF MS dataset
- Features used to train each PLS classifier are restricted to VIP > 1 metabolites from the matching PLS-DA/OPLS-DA model

## Limitations

- The LMM method did not outperform the ML method for specificity on the GC-TOF MS dataset, for which the ML method was slightly better
- Improved performance may carry a potential introduction of false positives due to increased sensitivity
- Performance claims are qualified as preliminary ('at current stage'); further experiments are recommended to validate the results

## Evidence

- [methods] The evaluation processes were carried out using the caret package: "The evaluation processes were carried out using the caret package"
- [methods] The significant features were selected based on variable importance in projection score (VIP > 1): "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [discussion] except specificity of GC-TOF MS dataset, for which the ML method was slightly better than the LMM method: "except specificity of GC-TOF MS dataset, for which the ML method was slightly better than the LMM method"
- [discussion] superior accuracy, precision, sensitivity and specificity than the M0 and ML methods in both UPLC-QTOF MS and GC-TOF MS datasets: "superior accuracy, precision, sensitivity and specificity than the M0 and ML methods in both UPLC-QTOF MS and GC-TOF MS datasets"
