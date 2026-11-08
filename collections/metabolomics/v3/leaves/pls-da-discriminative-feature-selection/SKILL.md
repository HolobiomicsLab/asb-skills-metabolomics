---
name: pls-da-discriminative-feature-selection
description: Use when you need to identify discriminative metabolites from a supervised
  PLS-DA/OPLS-DA model of a clinical metabolomics matrix and benchmark the predictive
  power of those features across data-processing strategies.
when_to_use_negative:
- You only need unsupervised exploration of the metabolomics matrix (e.g. PCA), not
  class-predictive feature selection
- The data have not been split into train/test partitions — VIP-based selection on
  the full dataset risks optimistic performance estimates
- You lack a binary class label or metadata needed to build the discriminant model
  (e.g. untargeted QC samples only)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mixOmics
  role: Performs PLS-DA and multilevel (ML) PLS-DA from which VIP > 1 discriminative
    features are selected
- name: ropls
  role: Performs OPLS-DA on M0- and LMM-processed training sets for VIP-based feature
    selection
- name: caret
  role: Creates repeated k-fold (k = 4, 5 repeats) train/test partitions and trains/predicts
    the PLS classification model on VIP-selected features
- name: lmm2met
  role: Produces the LMM-processed training sets (fitLmm with random intercept (1|Id))
    whose VIP-selected features are compared against M0 and ML
  repo: https://github.com/kwanjeeraw/lmm2met
- name: R
  role: Runtime environment in which all PLS-DA, OPLS-DA, and caret classification
    steps are executed
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:aae0547ca69966fa3ac8ffe744e70191a43692b75a064dab4b3ef078d0622b36
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# PLS-DA discriminative feature selection (VIP > 1)

## Summary

Selects discriminative metabolites from PLS-DA (and OPLS-DA) models using a variable importance in projection (VIP) score threshold of > 1, then trains a PLS classification model on those features to evaluate prediction performance. In the source article this is used to compare LMM-processed vs M0 and ML metabolomics data on GC-TOF MS and UPLC-QTOF MS datasets.

## When to use

Use when you need to identify discriminative metabolites from a supervised PLS-DA/OPLS-DA model of a clinical metabolomics matrix and benchmark the predictive power of those features across data-processing strategies.

## When NOT to use

- You only need unsupervised exploration of the metabolomics matrix (e.g. PCA), not class-predictive feature selection
- The data have not been split into train/test partitions — VIP-based selection on the full dataset risks optimistic performance estimates
- You lack a binary class label or metadata needed to build the discriminant model (e.g. untargeted QC samples only)

## Inputs

- Log2-transformed metabolomics feature matrix (GC-TOF MS lung metabolites, Metabolomics Workbench PR000305/ST000390; or UPLC-QTOF MS adipose lipids, PR000058/ST000081)
- Training-set versions processed with M0 (original), LMM (lmm2met fitLmm with (1|Id) random intercept), and ML (mixOmics multilevel)
- Train/test partitions from repeated k-fold CV (k = 4, five repeats, via caret)
- Class labels / patient metadata (e.g. malignant vs non-malignant lung tissue, n = 39 early-stage lung adenocarcinoma patients)

## Outputs

- List of discriminative metabolites per processing strategy with VIP > 1
- Fitted PLS classification models trained on VIP-selected features
- Per-subset and averaged accuracy, precision, sensitivity and specificity for LMM vs M0 vs ML (cf. Figure S-4)

## How to apply

Fit PLS-DA (mixOmics for M0-, LMM- and ML-processed training sets; ropls for OPLS-DA on M0- and LMM-processed sets) on each of the 20 repeated 4-fold train/test partitions (k = 4, repeated five times, ~75%/25% split via caret). Extract variable importance in projection scores and retain features with VIP > 1 as discriminative metabolites. Train a PLS classification model on the selected features with caret and predict the corresponding held-out test set. Compute accuracy, precision, sensitivity and specificity per subset and average over the 20 partitions to compare processing strategies (LMM vs M0 vs ML). Rationale: the article found LMM-processed data yielded features with superior prediction performance (~80% overall) except GC-TOF MS specificity, where ML was slightly better.

## Related tools

- **mixOmics** (Performs PLS-DA and multilevel (ML) PLS-DA from which VIP > 1 discriminative features are selected)
- **ropls** (Performs OPLS-DA on M0- and LMM-processed training sets for VIP-based feature selection)
- **caret** (Creates repeated k-fold (k = 4, 5 repeats) train/test partitions and trains/predicts the PLS classification model on VIP-selected features)
- **lmm2met** (Produces the LMM-processed training sets (fitLmm with random intercept (1|Id)) whose VIP-selected features are compared against M0 and ML) — https://github.com/kwanjeeraw/lmm2met
- **R** (Runtime environment in which all PLS-DA, OPLS-DA, and caret classification steps are executed) — https://www.r-project.org/

## Examples

```
install_github('kwanjeeraw/lmm2met'); # then with mixOmics/ropls/caret: plsda <- plsda(X.train, y.train); vip <- vip(plsda); feats <- which(vip > 1)
```

## Evaluation signals

- Every selected feature must have VIP > 1 in the fitted PLS-DA/OPLS-DA model
- Averaged accuracy, precision and sensitivity for the LMM-processed features must exceed M0 and ML on both the UPLC-QTOF MS and GC-TOF MS datasets
- Expected exception: on the GC-TOF MS dataset, specificity of ML should slightly exceed LMM (overall performance ~80%)
- Performance estimates must be averaged over all 20 train/test subsets from the repeated 4-fold CV, not a single split
- PLS-DA/OPLS-DA must be fit only on training sets; predictions assessed only on the corresponding held-out test sets

## Limitations

- LMM-based processing may introduce false positives due to increased sensitivity
- For GC-TOF MS specificity, the ML method slightly outperformed LMM, so LMM superiority is not universal across metrics
- Performance claims are qualified as preliminary; the authors recommend further experiments to validate the results

## Evidence

- [methods] VIP threshold used to select discriminative metabolites: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [methods] Repeated k-fold CV design and PLS-DA/OPLS-DA modeling of processed training sets: "each dataset was partitioned into training and test sets with a repeated k-fold cross validation [31] (k = 4)"
- [discussion] Superior prediction performance of LMM except GC-TOF MS specificity: "except specificity of GC-TOF MS dataset, for which the ML method was slightly better than the LMM method"
- [methods] Tooling for PLS-DA, multilevel PLS-DA, OPLS-DA and evaluation: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [discussion] Potential false positives from LMM processing: "albeit with the potential introduction of false positives"
