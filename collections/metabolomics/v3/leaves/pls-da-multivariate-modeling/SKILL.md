---
name: pls-da-multivariate-modeling
description: Use when a log2-transformed metabolite/lipid feature matrix with known
  class labels (e.g. tissue type, disease status) needs supervised class discrimination
  and ranking of discriminative features by VIP score.
when_to_use_negative:
- The data matrix is raw/unprocessed MS output — PLS-DA requires pre-processed (noise-reduced,
  peak-matched, normalization, scaling, log2-transformed) features first
- The analysis goal is unsupervised structure exploration — use PCA instead, as the
  article does for initial SAT/VAT separation
- Class labels are unavailable or the design is not a two-class supervised comparison,
  so supervised discriminant modeling is not appropriate
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mixOmics
  role: Fits the PLS-DA (and multilevel PLS-DA) models on the M0- and LMM-processed
    metabolomics matrices and provides VIP scores via mixOmics::vip
- name: ropls
  role: Performs the OPLS-DA variant of the discriminant modeling and extracts VIP
    scores via getVipVn
- name: caret
  role: Implements repeated k-fold cross validation (k = 4) for training/test partitioning
    and evaluation of the PLS classification models built from VIP-selected features
- name: lmm2met
  role: 'Produces the LMM-processed data matrix (fixed effects: Sex, Age, BMI, Stage,
    Location, Tissue; random intercept (1|Id)) that is fed into PLS-DA and compared
    against the M0 baseline'
  repo: https://github.com/kwanjeeraw/lmm2met
- name: ggplot2
  role: Visualization of PLS-DA separation and VIP-score comparisons (e.g. Figure
    S-3)
- name: R
  role: Environment in which all PLS-DA/OPLS-DA analyses are performed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:b74d4e6c4387a5e796ef2028c486ac5d82d99a81c67ddc2b83001fcd35a9aa69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - caret
  - ggplot2
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# pls-da-multivariate-modeling

## Summary

Perform PLS-DA (and optionally OPLS-DA) on pre-processed metabolomics matrices to discriminate classes (e.g. SAT vs VAT adipose tissue), then extract VIP scores on the first component to select discriminative metabolites (VIP > 1). In this article it is applied to M0 and LMM-processed CRC adipose lipid data (158 annotated lipids, 59 patients).

## When to use

Use when a log2-transformed metabolite/lipid feature matrix with known class labels (e.g. tissue type, disease status) needs supervised class discrimination and ranking of discriminative features by VIP score.

## When NOT to use

- The data matrix is raw/unprocessed MS output — PLS-DA requires pre-processed (noise-reduced, peak-matched, normalization, scaling, log2-transformed) features first
- The analysis goal is unsupervised structure exploration — use PCA instead, as the article does for initial SAT/VAT separation
- Class labels are unavailable or the design is not a two-class supervised comparison, so supervised discriminant modeling is not appropriate

## Inputs

- Log2-transformed metabolomics/lipidomics feature matrix (samples x metabolites), e.g. 158 annotated lipids from Metabolomics Workbench PR000058 / ST000081
- Sample class labels (e.g. tissue type: SAT vs VAT; or disease status: malignant vs non-malignant)
- Optional: processed data variants to compare (M0 unprocessed vs LMM-processed matrices from lmm2met)

## Outputs

- PLS-DA sample-score plots colored by class
- Per-metabolite VIP scores on the first component
- Set of discriminative metabolites with VIP > 1 (e.g. 69 for LMM-processed, 66 for M0 CRC adipose data)
- DeltaVIP comparison table (e.g. Table S-3) of metabolite VIP shifts between data-processing variants
- Class prediction performance metrics from repeated k-fold CV (accuracy, precision, sensitivity, specificity)

## How to apply

Start from a log2-transformed, pre-processed metabolomics matrix (e.g. the 158 annotated lipids from VAT and SAT of 59 colorectal carcinoma patients) with sample class labels. Fit a PLS-DA model using mixOmics::plsda, coloring samples by class to inspect separation; optionally fit OPLS-DA with ropls. Compute variable importance in projection scores (mixOmics::vip or ropls::getVipVn) on the first component and select discriminative metabolites with VIP > 1, as done in the article where LMM-processed data gave 69 discriminative metabolites versus 66 for M0. For predictive evaluation, partition data into training/test sets with repeated k-fold cross validation (k = 4) using caret, select features by VIP, and assess class prediction on the held-out test set. Comparing VIP-selected sets between data-processing variants (M0 vs LMM) quantifies how covariate adjustment reshapes feature importance (e.g. cholesterol ΔVIP = 0.52, SM (d40:2) ΔVIP = 0.43, acylcarnitine ΔVIP = 0.32).

## Related tools

- **mixOmics** (Fits the PLS-DA (and multilevel PLS-DA) models on the M0- and LMM-processed metabolomics matrices and provides VIP scores via mixOmics::vip)
- **ropls** (Performs the OPLS-DA variant of the discriminant modeling and extracts VIP scores via getVipVn)
- **caret** (Implements repeated k-fold cross validation (k = 4) for training/test partitioning and evaluation of the PLS classification models built from VIP-selected features)
- **lmm2met** (Produces the LMM-processed data matrix (fixed effects: Sex, Age, BMI, Stage, Location, Tissue; random intercept (1|Id)) that is fed into PLS-DA and compared against the M0 baseline) — https://github.com/kwanjeeraw/lmm2met
- **ggplot2** (Visualization of PLS-DA separation and VIP-score comparisons (e.g. Figure S-3))
- **R** (Environment in which all PLS-DA/OPLS-DA analyses are performed)

## Examples

```
install_github('kwanjeeraw/lmm2met'); library(mixOmics); plsda.fit <- plsda(X, y); vip(plsda.fit)
```

## Evaluation signals

- Reproduce the discriminative metabolite counts: 69 metabolites with VIP > 1 from PLS-DA on LMM-processed CRC adipose data vs 66 from the M0 dataset
- Reproduce DeltaVIP shifts for key lipids: cholesterol 0.52, SM (d40:2) 0.43, acylcarnitine (Acylcarnitine C18:1) 0.32 within reasonable numerical tolerance (Table S-3, Figure S-3)
- Check that cholesterol, undetectable by PLS-DA separation in M0 (signals randomly mixed), becomes discriminative after LMM fitting
- Verify classification performance (accuracy, precision, sensitivity, specificity) of VIP-selected-feature PLS models via repeated k-fold CV (k = 4) using caret
- Confirm feature selection used the VIP > 1 threshold on the first component, matching the article's cutoff

## Limitations

- Increased sensitivity after LMM processing may introduce false positives among discriminative features
- Performance claims are preliminary and further experiments are recommended to validate the results
- The LMM preprocessing did not outperform the ML method in specificity on the GC-TOF MS dataset, so PLS-DA performance gains depend on the data-processing strategy and platform

## Evidence

- [methods] The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA.: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [methods] The significant features were selected based on variable importance in projection score (VIP > 1): "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [methods] each dataset was partitioned into training and test sets with a repeated k-fold cross validation [31] (k = 4): "each dataset was partitioned into training and test sets with a repeated k-fold cross validation [31] (k = 4)"
- [results] significant metabolites or discriminative metabolites (VIP > 1): "significant metabolites or discriminative metabolites (VIP > 1)"
- [results] Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable: "Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable"
