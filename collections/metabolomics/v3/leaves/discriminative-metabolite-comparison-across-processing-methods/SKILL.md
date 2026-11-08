---
name: discriminative-metabolite-comparison-across-processing-methods
description: Use when a metabolomics feature table has matched patient/subject metadata
  and you want to quantify how LMM-based covariate adjustment changes which metabolites
  pass a VIP > 1 threshold relative to unprocessed data.
when_to_use_negative:
- The data lack subject/patient metadata (age, BMI, sex, etc.), so a random-intercept
  LMM per metabolite cannot be formulated
- The feature table is raw, unpre-processed MS output rather than log2-transformed,
  normalized peak data ready for multivariate modeling
- The analysis goal is prediction performance benchmarking (e.g. repeated k-fold cross-validation
  accuracy) rather than comparing VIP-selected feature sets between processing strategies
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: lmm2met
  role: Fits a per-metabolite linear mixed-effects model to re-adjust metabolite signals
    using subject metadata, producing the LMM-processed matrix
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting engine used by lmm2met
- name: mixOmics
  role: Performs PLS-DA on M0 and LMM-processed matrices and extracts VIP scores (vip)
    for feature selection at VIP > 1
- name: ropls
  role: Alternative OPLS-DA implementation via getVipVn to obtain VIP scores
- name: ggplot2
  role: Visualization of PLS-DA sample plots and VIP-score comparisons
- name: devtools
  role: Installs lmm2met from GitHub via install_github
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:461dc1f702dd46e495e421d8c783cee0472aa8c598f0d3bb42084af02e65f27a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - devtools
  - ggplot2
  - lme4
  - lmm2met
  - mixOmics
  - ropls
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# discriminative-metabolite-comparison-across-processing-methods

## Summary

Compare discriminative metabolites (VIP > 1) selected by PLS-DA on metabolomics data before (M0) versus after linear mixed-effects model (LMM) processing, quantifying how VIP scores shift for key metabolites (e.g. cholesterol, SM (d40:2), acylcarnitines).

## When to use

Use when a metabolomics feature table has matched patient/subject metadata and you want to quantify how LMM-based covariate adjustment changes which metabolites pass a VIP > 1 threshold relative to unprocessed data.

## When NOT to use

- The data lack subject/patient metadata (age, BMI, sex, etc.), so a random-intercept LMM per metabolite cannot be formulated
- The feature table is raw, unpre-processed MS output rather than log2-transformed, normalized peak data ready for multivariate modeling
- The analysis goal is prediction performance benchmarking (e.g. repeated k-fold cross-validation accuracy) rather than comparing VIP-selected feature sets between processing strategies

## Inputs

- Log2-transformed, pre-processed metabolomics/lipidomics feature matrix (e.g. 158 annotated lipids, VAT and SAT samples)
- Patient metadata table with covariates: Sex, Age, BMI, tumor Stage, Location, Tissue, and a subject/patient ID for the random intercept
- Class labels for PLS-DA (e.g. tissue type: VAT vs SAT)

## Outputs

- Count of discriminative metabolites (VIP > 1) for the LMM-processed data (reported: 69) and the M0 data (reported: 66)
- Per-metabolite DeltaVIP table comparing VIP scores between LMM- and M0-PLS-DA (e.g. cholesterol ΔVIP = 0.52, SM (d40:2) ΔVIP = 0.43, Acylcarnitine C18:1 ΔVIP = 0.32)
- PLS-DA models and sample plots colored by tissue type for both processing conditions

## How to apply

Load the log2-transformed annotated feature matrix (e.g. 158 lipids from VAT and SAT of 59 colorectal carcinoma patients, Metabolomics Workbench PR000058 / ST000081) plus metadata (Sex, Age, BMI, Stage, Location, Tissue). Fit a per-metabolite LMM with lmm2met::fitLmm (built on lme4) using those covariates as fixed effects and a patient random intercept (1|Id), retaining the unprocessed matrix as M0. Run PLS-DA with mixOmics on both matrices, extract VIP scores on the first component (mixOmics::vip, or ropls::getVipVn for OPLS-DA), and select metabolites with VIP > 1. Compute the count of discriminative metabolites in each condition and the per-metabolite DeltaVIP (LMM minus M0) to identify metabolites whose discriminative power changes after covariate adjustment.

## Related tools

- **lmm2met** (Fits a per-metabolite linear mixed-effects model to re-adjust metabolite signals using subject metadata, producing the LMM-processed matrix) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting engine used by lmm2met)
- **mixOmics** (Performs PLS-DA on M0 and LMM-processed matrices and extracts VIP scores (vip) for feature selection at VIP > 1)
- **ropls** (Alternative OPLS-DA implementation via getVipVn to obtain VIP scores)
- **ggplot2** (Visualization of PLS-DA sample plots and VIP-score comparisons)
- **devtools** (Installs lmm2met from GitHub via install_github)

## Examples

```
install_github('kwanjeeraw/lmm2met')
```

## Evaluation signals

- The LMM-processed PLS-DA should yield 69 metabolites with VIP > 1 and the M0 PLS-DA should yield 66
- Computed DeltaVIP values should match reported values: cholesterol 0.52, SM (d40:2) 0.43, Acylcarnitine C18:1 0.32, within reasonable numerical tolerance
- The same LMM formula must be used for every metabolite: fixed effects Sex, Age, BMI, Stage, Location, Tissue with random intercept (1|Id)
- Cholesterol should show a large positive DeltaVIP; before LMM fitting its signals were randomly mixed between tissue classes in the M0 PLS-DA
- Variance of each metabolite across patients should be substantially reduced after LMM fitting compared to M0

## Limitations

- Increased sensitivity after LMM processing carries the potential introduction of false positives
- Results are preliminary; further experiments are recommended to validate the findings
- The LMM method did not outperform the multilevel (ML) method for specificity on the GC-TOF MS dataset

## Evidence

- [methods] VIP > 1 feature selection: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [abstract] LMM processing step: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
- [methods] mixOmics PLS-DA: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [methods] lmm2met implementation on lme4: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [readme] install lmm2met: "#Install lmm2met
install_github('kwanjeeraw/lmm2met')"
