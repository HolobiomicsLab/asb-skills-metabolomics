---
name: clinical-metabolomics-data-processing
description: Use when a clinical metabolomics/lipidomics feature matrix has matched
  patient metadata (e.g. age, BMI, sex, tumor stage, tissue, repeated measures) and
  subject-specific variation must be minimized before multivariate modeling or feature
  selection.
when_to_use_negative:
- Input data lacks patient metadata or subject IDs, so no random-effects structure
  can be formulated for the LMM
- The dataset is already a fully processed/normalized feature table and the goal is
  only raw MS pre-processing (noise reduction, peak matching, retention time correction)
- Analysis involves non-clinical samples without repeated measures or biological covariates
  to model
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: lmm2met
  role: Core R package implementing the linear mixed-effects modelling data-processing
    step for normalizing clinical metabolomics data using subject metadata
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting engine on which lmm2met's LMM procedure is
    implemented
- name: mixOmics
  role: Performs PLS-DA (and multilevel PLS-DA) on the M0 and LMM-processed matrices;
    mixOmics::vip extracts VIP scores for VIP > 1 feature selection
- name: ropls
  role: Performs the OPLS-DA variant and provides VIP scores (getVipVn) for comparison
- name: caret
  role: Repeated k-fold (k = 4) cross-validation to evaluate classification accuracy,
    precision, sensitivity and specificity of features selected from LMM- vs M0-processed
    data
- name: devtools
  role: Used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met')
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:0d168a360a6a1e87bbf8a41c3a1b2824b494bfd4a170755b9fe9cbd0a6845bfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - caret
  - devtools
  - ggplot2
  - lme4
  - lmm2met
  - mixOmics
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# clinical-metabolomics-data-processing

## Summary

An additional data-processing step that readjusts individual metabolite signals in clinical metabolomics matrices by fitting a linear mixed-effects model (LMM) per metabolite using patient metadata, improving downstream multivariate analyses such as PLS-DA. Implemented in the R package lmm2met on top of lme4.

## When to use

Use when a clinical metabolomics/lipidomics feature matrix has matched patient metadata (e.g. age, BMI, sex, tumor stage, tissue, repeated measures) and subject-specific variation must be minimized before multivariate modeling or feature selection.

## When NOT to use

- Input data lacks patient metadata or subject IDs, so no random-effects structure can be formulated for the LMM
- The dataset is already a fully processed/normalized feature table and the goal is only raw MS pre-processing (noise reduction, peak matching, retention time correction)
- Analysis involves non-clinical samples without repeated measures or biological covariates to model

## Inputs

- Log2-transformed metabolomics/lipidomics feature matrix (metabolites x samples), e.g. GC-TOF MS or UPLC-QTOF MS data
- Patient metadata table with covariates (age, BMI, sex, tumor stage, tumor location, tissue type) and subject IDs
- R environment with lmm2met (installed via devtools::install_github('kwanjeeraw/lmm2met')), lme4, mixOmics, ropls, caret, ggplot2

## Outputs

- LMM-processed metabolite data matrix with subject-specific variation readjusted per metabolite
- VIP-score table of discriminative metabolites (VIP > 1) from LMM- vs M0-PLS-DA (e.g. Table S-3 with DeltaVIP per metabolite)
- PCA/PLS-DA sample-score plots colored by tissue type or class (e.g. Figure S-3)
- Cross-validated classification performance metrics (accuracy, precision, sensitivity, specificity) from caret

## How to apply

Start from log2-transformed, pre-processed metabolite/lipid abundance data (e.g. the 158 annotated lipids from VAT and SAT of 59 colorectal carcinoma patients, Metabolomics Workbench PR000058 / ST000081). Fit one LMM per metabolite with lmm2met::fitLmm (built on lme4), supplying the patient metadata as fixed effects — Sex, Age, BMI, Stage, Location, Tissue — and a random subject intercept (1|Id); keep the unprocessed matrix as the M0 baseline. The LMM-processed matrix systematically minimizes subject-specific variation, with medians closer to zero and less within-tissue variability. Compare downstream results: run PCA or PLS-DA (mixOmics) on both matrices, and select discriminative metabolites with VIP > 1 on the first component (e.g. 69 metabolites for LMM vs 66 for M0 in the CRC adipose tissue data). Assess classification gains with the caret package via repeated 4-fold cross-validation, or use ropls for the OPLS-DA variant.

## Related tools

- **lmm2met** (Core R package implementing the linear mixed-effects modelling data-processing step for normalizing clinical metabolomics data using subject metadata) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting engine on which lmm2met's LMM procedure is implemented)
- **mixOmics** (Performs PLS-DA (and multilevel PLS-DA) on the M0 and LMM-processed matrices; mixOmics::vip extracts VIP scores for VIP > 1 feature selection)
- **ropls** (Performs the OPLS-DA variant and provides VIP scores (getVipVn) for comparison)
- **caret** (Repeated k-fold (k = 4) cross-validation to evaluate classification accuracy, precision, sensitivity and specificity of features selected from LMM- vs M0-processed data)
- **devtools** (Used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met'))

## Examples

```
install.packages('devtools', repos="http://cran.rstudio.com/"); library(devtools); install_github('kwanjeeraw/lmm2met')
```

## Evaluation signals

- Variance of each metabolite across patients is substantially reduced after LMM fitting, and subject-wise boxplots show medians closer to zero with less within-tissue variability
- PCA of LMM-fitted data shows better SAT/VAT separation, with ~6% increase in explained variation on PC1 versus the original data
- VIP > 1 selection on LMM-PLS-DA yields 69 discriminative metabolites vs 66 for M0-PLS-DA in the CRC adipose tissue dataset (counts should match within numerical tolerance)
- DeltaVIP for key lipids reproduces Table S-3 values: cholesterol DeltaVIP = 0.52, SM (d40:2) DeltaVIP = 0.43, Acylcarnitine C18:1 DeltaVIP = 0.32
- Repeated 4-fold cross-validation (caret) shows improved accuracy, precision, sensitivity and specificity for LMM-processed features relative to M0

## Limitations

- The increased sensitivity of LMM processing carries the potential introduction of false positives
- The proposed method exhibited superior performance 'at current stage' — results are preliminary and further experiments are recommended to validate them
- LMM did not outperform the ML method for specificity on the GC-TOF MS dataset (ML was slightly better)
- Cholesterol signals were randomly mixed and indistinguishable before LMM fitting in the M0 dataset, illustrating failure of the unprocessed baseline rather than of the method

## Evidence

- [abstract] We propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses.: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
- [methods] The package was implemented on a linear model-fitting procedure of the lme4 package: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [methods] Both datasets were pre-processed as described previously and log2-transformed prior to use in this study.: "Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study."
- [methods] The significant features were selected based on variable importance in projection score (VIP > 1): "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [results] the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [discussion] albeit with the potential introduction of false positives: "albeit with the potential introduction of false positives"
- [readme] Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada.: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
