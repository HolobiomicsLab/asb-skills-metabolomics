---
name: metabolomics-variance-adjustment
description: Use when clinical metabolomics matrices (e.g. log2-transformed lipid/metabolite
  features from GC-TOF MS or UPLC-QTOF MS) still show subject-specific variance and
  outliers after standard pre-processing, and normalization should account for patient
  metadata before downstream multivariate analysis.
when_to_use_negative:
- The input is raw MS data that still requires pre-processing steps such as noise
  reduction, peak matching, or retention time correction — LMM fitting is an additional
  data-processing step applied after normalization, scaling, centering and log2 transformation
- No patient/subject metadata is available to define fixed effects and a random subject
  intercept, since the method relies on subject characteristics to model and normalize
  metabolite abundances
- The goal is class prediction where a machine-learning normalization alternative
  is specifically required — note the LMM method did not outperform the ML method
  for specificity on the GC-TOF MS dataset
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: Core R package that fits a per-metabolite LMM using subject metadata and returns
    the readjusted metabolite matrix
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure on which lmm2met's lmer-based LMM
    fitting is implemented
- name: ggplot2
  role: Rendering before-vs-after boxplots of metabolite signals colored by tissue
    type to verify variance reduction
- name: devtools
  role: Used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met')
- name: R
  role: Runtime environment; all analyses were performed using R
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:f5020946309b27c2b65e0909bc3f9eb0c0cf359e129a74fa850b369f2c6a8c29
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-variance-adjustment

## Summary

Fits a linear mixed-effects model (LMM) per metabolite to readjust individual metabolite signals using patient metadata (fixed effects plus a random patient intercept), reducing per-metabolite variance and correcting outliers in clinical metabolomics data prior to multivariate analyses such as PLS-DA.

## When to use

Use when clinical metabolomics matrices (e.g. log2-transformed lipid/metabolite features from GC-TOF MS or UPLC-QTOF MS) still show subject-specific variance and outliers after standard pre-processing, and normalization should account for patient metadata before downstream multivariate analysis.

## When NOT to use

- The input is raw MS data that still requires pre-processing steps such as noise reduction, peak matching, or retention time correction — LMM fitting is an additional data-processing step applied after normalization, scaling, centering and log2 transformation
- No patient/subject metadata is available to define fixed effects and a random subject intercept, since the method relies on subject characteristics to model and normalize metabolite abundances
- The goal is class prediction where a machine-learning normalization alternative is specifically required — note the LMM method did not outperform the ML method for specificity on the GC-TOF MS dataset

## Inputs

- Log2-transformed metabolite/lipid abundance matrix (features x samples), e.g. 158 log2-transformed lipids from SAT and VAT of 59 CRC patients
- Patient metadata table (subject characteristics such as Sex, Age, BMI, Stage, Location, Tissue, patient Id) used to formulate the LMMs
- R session with lmm2met installed from github.com/kwanjeeraw/lmm2met (requires lme4)

## Outputs

- LMM-fitted (readjusted) metabolite matrix available as fitMet$fittedDat
- Per-metabolite linear mixed-effects model objects (fitMet$completeMod)
- Fixed-effect coefficient and significance heatmaps via plot(fitMet, type='fixeff') and plot(fitMet, type='coeff')
- Before-vs-after boxplots of metabolite signals (e.g. colored by tissue type SC vs VS) demonstrating variance reduction

## How to apply

Load the pre-processed, log2-transformed metabolite matrix together with patient metadata (e.g. the lmm2met 'adipose' dataset: 158 lipids from SAT and VAT of 59 CRC patients, Metabolomics Workbench ST000081). Fit a per-metabolite LMM with lmm2met::fitLmm, supplying covariates such as fix=c('Sex','Age','BMI','Stage','Location','Tissue') and a random patient intercept random='(1|Id)'; the implementation builds on lme4's lmer fitting procedure. Extract the processed matrix from fitMet$fittedDat, which accounts for fixed effects and subject-specific random intercepts, and inspect individual models via summary(fitMet$completeMod[[1]]). The rationale is that subject-specific variation in each metabolite is systematically minimized, so boxplots of signals before vs after fitting (plot(fitMet, type='fitted', fix='Tissue')) should show substantially reduced variance, corrected outliers, and subject-wise medians closer to zero within tissue type.

## Related tools

- **lmm2met** (Core R package that fits a per-metabolite LMM using subject metadata and returns the readjusted metabolite matrix) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure on which lmm2met's lmer-based LMM fitting is implemented)
- **ggplot2** (Rendering before-vs-after boxplots of metabolite signals colored by tissue type to verify variance reduction)
- **devtools** (Used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met'))
- **R** (Runtime environment; all analyses were performed using R) — https://www.r-project.org/

## Examples

```
install.packages('devtools'); library(devtools); install_github('kwanjeeraw/lmm2met'); fitMet <- lmm2met::fitLmm(data=adipose, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)')
```

## Evaluation signals

- Per-metabolite variance across all patients is substantially reduced after LMM fitting compared with the original signals, with outliers corrected
- Subject-wise boxplots after LMM fitting show a median closer to zero and less variability within tissue type (SC vs VS)
- PCA of the LMM-fitted data shows better separation between SAT and VAT, e.g. a 6% increase in explanation of variation on PC1 relative to the original data
- fitMet$fittedDat contains the readjusted matrix and fitMet$completeMod[[1]] yields a valid lmer model summary for the first metabolite
- Downstream PLS-DA on LMM-processed data shows improved classification accuracy, precision, sensitivity and specificity compared with unadjusted (M0) data

## Limitations

- The proposed method exhibited superior performance only at the current (preliminary) stage and further experiments are recommended to validate the results
- Increased sensitivity may introduce potential false positives
- For the GC-TOF MS dataset, the ML method was slightly better than the LMM method in terms of specificity
- Version 1.0 (12 September 2018); repository provides no changelog, and documentation/tutorial is hosted at the package homepage

## Evidence

- [results] variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [methods] We developed the R package lmm2met to process a metabolomics matrix with the LMM approach.: "We developed the R package lmm2met to process a metabolomics matrix with the LMM approach."
- [methods] The package was implemented on a linear model-fitting procedure of the lme4 package: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [results] the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [readme] Install lmm2met using the following commands: "Install lmm2met using the following commands"
- [abstract] we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
