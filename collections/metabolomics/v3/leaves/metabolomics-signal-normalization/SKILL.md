---
name: metabolomics-signal-normalization
description: Use when raw or pre-processed clinical metabolomics matrices (e.g. log2-transformed
  lipid or GC-TOF MS profiles from paired patient samples like CRC VAT/SAT) show subject-driven
  variation that obscures group differences.
when_to_use_negative:
- The matrix has not been pre-processed/log2-transformed — LMM fitting is described
  as an additional data-processing step applied after standard pre-processing and
  log2 transformation
- There is no repeated-measures or paired subject structure with metadata covariates
  to model (e.g. unpaired single-sample groups with no random-effect identifier)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: lmm2met
  role: R package providing fitLmm to process the metabolomics matrix with the LMM
    approach and getFixCoeff to extract fixed-effect coefficients and chi-square significances
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure on which lmm2met is implemented
- name: devtools
  role: Used to install lmm2met from GitHub with install_github
- name: R
  role: Runtime environment in which all normalization analyses were performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:f370af119d0f48f89bc4b5f291d456cc124590a72ec0f2ecbe71ebb2d70987b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - devtools
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolomics-signal-normalization

## Summary

Normalize clinical metabolomics signal matrices by fitting a linear mixed-effects model (LMM) per metabolite using patient metadata, via the R package lmm2met. This readjusts individual metabolite signals prior to multivariate analyses such as PLS-DA.

## When to use

Use when raw or pre-processed clinical metabolomics matrices (e.g. log2-transformed lipid or GC-TOF MS profiles from paired patient samples like CRC VAT/SAT) show subject-driven variation that obscures group differences.

## When NOT to use

- The matrix has not been pre-processed/log2-transformed — LMM fitting is described as an additional data-processing step applied after standard pre-processing and log2 transformation
- There is no repeated-measures or paired subject structure with metadata covariates to model (e.g. unpaired single-sample groups with no random-effect identifier)

## Inputs

- Log2-transformed metabolomics data matrix (metabolites x samples), e.g. 158 lipids from 59 CRC patients with paired VAT/SAT adipose samples (Metabolomics Workbench ST000081)
- Patient metadata table with covariates (Sex, Age, BMI, Stage, Location, Tissue) and subject identifiers

## Outputs

- LMM-fitted (normalized) metabolomics matrix with subject-wise variation reduced
- Per-metabolite fixed-effect coefficient table (coefficients and chi-square-test p-values) from getFixCoeff, e.g. cholesterol_vat_lmm_result.csv

## How to apply

Log2-transform the metabolomics matrix first, as in the paper where both datasets were log2-transformed prior to use. Install lmm2met from GitHub via devtools, then call fitLmm on each measured metabolite with fixed effects for covariates such as Sex, Age, BMI, Stage, Location and Tissue, and a patient random intercept '(1|Id)' — lmm2met wraps the lme4 fitting procedure. Extract fixed-effect coefficients and chi-square significances with getFixCoeff per metabolite (e.g. the cholesterol VAT coefficient of 0.05, p = .05 in Table S-2). The rationale is that LMM fitting systematically minimizes subject-specific variation: variance of each metabolite across patients is substantially reduced and outliers corrected, improving SAT/VAT separation in PCA (~6% more variation explained on PC1).

## Related tools

- **lmm2met** (R package providing fitLmm to process the metabolomics matrix with the LMM approach and getFixCoeff to extract fixed-effect coefficients and chi-square significances) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure on which lmm2met is implemented)
- **devtools** (Used to install lmm2met from GitHub with install_github)
- **R** (Runtime environment in which all normalization analyses were performed)

## Examples

```
install.packages('devtools'); library(devtools); install_github('kwanjeeraw/lmm2met'); fit <- fitLmm(lipids, fixed=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)')
```

## Evaluation signals

- Reproducing the tutorial result: cholesterol VAT coefficient of 0.05 with chi-square p-value 0.05 (Table S-2) after fitting with the specified fixed effects and '(1|Id)' random intercept
- Variance of each metabolite across all patients is substantially reduced after fitting, and subject-wise boxplots have medians closer to zero with less within-tissue variability
- PCA of LMM-fitted data separates SAT from VAT with ~6% more variation explained on PC1 compared to the original data
- getFixCoeff returns coefficients and chi-square-test significances for every metabolite in the input matrix without errors

## Limitations

- Increased sensitivity of the normalized data can introduce potential false positives
- The proposed method was evaluated at a current/preliminary stage and further experiments are recommended to validate the results
- For specificity on the GC-TOF MS dataset, the machine-learning method was slightly better than the LMM method

## Evidence

- [methods] We developed the R package lmm2met to process a metabolomics matrix with the LMM approach.: "We developed the R package lmm2met to process a metabolomics matrix with the LMM approach."
- [methods] log2 transformation before LMM fitting: "Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study."
- [results] variance reduction after LMM fitting: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [results] PCA separation improvement: "depicted better separation between SAT and VAT with 6% increase in explanation of variation on PC1"
- [readme] README purpose statement: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
