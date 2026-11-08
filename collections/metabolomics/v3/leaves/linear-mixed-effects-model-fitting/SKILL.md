---
name: linear-mixed-effects-model-fitting
description: Use when paired or repeated clinical metabolomics samples (e.g. VAT/SAT
  tissue pairs from 59 CRC patients) contain subject-specific variation that confounds
  metabolite signals and you need to readjust per-metabolite abundances before PCA
  or PLS-DA.
when_to_use_negative:
- The data are untransformed or not pre-processed — signals must already be pre-processed
  (noise reduction, peak matching, retention time correction) and log2-transformed
  before LMM fitting
- Samples lack patient metadata (no fixed-effect covariates such as Sex, Age, BMI,
  Stage, Location, Tissue) and have no repeated measures or patient Id for a random
  intercept
- The goal is classification-model evaluation rather than signal readjustment — the
  article evaluates predictions separately with caret under repeated 4-fold cross
  validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: R package providing fitLmm, getFixCoeff and plotting methods to fit per-metabolite
    LMMs and return the readjusted metabolite matrix
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: underlying linear model-fitting procedure on which lmm2met's LMM implementation
    is built
- name: ggplot2
  role: rendering before-vs-after subject-wise boxplots and score plots used to validate
    variance reduction
- name: devtools
  role: install_github('kwanjeeraw/lmm2met') to install the package
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:bf7d89e5de3574582cffd40327fe04943aff500af0aa65fac50071601d0d206e
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# linear-mixed-effects-model-fitting

## Summary

Fits a per-metabolite linear mixed-effects model (via lmm2met, built on lme4) in which patient metadata (e.g. Sex, Age, BMI, Stage, Location, Tissue) are fixed effects and a patient random intercept '(1|Id)' captures subject-specific variation, producing a readjusted metabolite matrix with reduced per-metabolite variance prior to multivariate analyses.

## When to use

Use when paired or repeated clinical metabolomics samples (e.g. VAT/SAT tissue pairs from 59 CRC patients) contain subject-specific variation that confounds metabolite signals and you need to readjust per-metabolite abundances before PCA or PLS-DA.

## When NOT to use

- The data are untransformed or not pre-processed — signals must already be pre-processed (noise reduction, peak matching, retention time correction) and log2-transformed before LMM fitting
- Samples lack patient metadata (no fixed-effect covariates such as Sex, Age, BMI, Stage, Location, Tissue) and have no repeated measures or patient Id for a random intercept
- The goal is classification-model evaluation rather than signal readjustment — the article evaluates predictions separately with caret under repeated 4-fold cross validation

## Inputs

- log2-transformed metabolomics data matrix with patient metadata columns (e.g. 158 annotated lipids from VAT and SAT of 59 CRC patients)
- patient metadata table (Sex, Age, BMI, tumor Stage, tumor Location, Tissue type, patient Id)
- Metabolomics Workbench study ST000081 (Project PR000058) or the packaged lmm2met 'adipose' dataset

## Outputs

- LMM-processed metabolite profile matrix fitMet$fittedDat (write to fitted_metabolite_matrix.csv)
- fixed-effect coefficient table with chi-square significances per metabolite (via getFixCoeff, e.g. fixed_effect_coefficients.csv)
- before-vs-after subject-wise boxplots and coefficient/fixeff heatmaps (ggplot2)
- per-metabolite fitted LMM objects (fitMet$completeMod) for convergence inspection

## How to apply

Load the log2-transformed metabolomics matrix with patient metadata columns (e.g. the packaged 'adipose' data: 158 lipids from SAT and VAT of 59 CRC patients, Metabolomics Workbench ST000081/PR000058). Call lmm2met::fitLmm with fix=c('Sex','Age','BMI','Stage','Location','Tissue') and random='(1|Id)', implemented on lme4's lmer fitting procedure; each metabolite y_ij is modeled as beta0 + mu0i + sum(beta_jk * X_ik) + epsilon_ij. Inspect convergence via summary(fitMet$completeMod[[1]]), then extract the processed matrix fitMet$fittedDat in which the fixed effects and subject-specific random intercepts are accounted for. Inspect fixed-effect coefficients and chi-square significance heatmaps with plot(fitMet, type='fixeff') and plot(fitMet, type='coeff') or getFixCoeff, and verify variance reduction with before-vs-after subject-wise boxplots via plot(fitMet, type='fitted', fix='Tissue'). The rationale is that patient metadata contains rich information of subject characteristics that can be used to model and normalize metabolite abundances, systematically minimizing subject-specific variation per metabolite.

## Related tools

- **lmm2met** (R package providing fitLmm, getFixCoeff and plotting methods to fit per-metabolite LMMs and return the readjusted metabolite matrix) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (underlying linear model-fitting procedure on which lmm2met's LMM implementation is built)
- **ggplot2** (rendering before-vs-after subject-wise boxplots and score plots used to validate variance reduction)
- **devtools** (install_github('kwanjeeraw/lmm2met') to install the package)

## Examples

```
fitMet <- lmm2met::fitLmm(dat, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'); processed <- fitMet$fittedDat
```

## Evaluation signals

- Per-metabolite variance across all patients is substantially reduced and outliers are corrected after fitting (Figure S-1 / Fig. 3a boxplot comparison)
- Subject-wise boxplots after LMM fitting have a median closer to zero and show less variability within tissue type (SC vs VS)
- PCA on fitMet$fittedDat shows better SAT/VAT separation with ~6 percentage-point higher explained variation on PC1 versus the original data
- Known fixed-effect estimates reproduce published values, e.g. cholesterol VAT coefficient of 0.05 with p-value .05 (chi-square test, Table S-2)
- Per-metabolite models in fitMet$completeMod converge (check summary(fitMet$completeMod[[1]]))

## Limitations

- The method requires subject metadata; missing covariates (Sex, Age, BMI, Stage, Location, Tissue, patient Id) prevent model formulation
- Increased sensitivity of downstream PLS-DA after LMM fitting carries the potential introduction of false positives
- At the current stage the method is preliminary; further experiments are recommended to validate the results
- On the GC-TOF MS dataset, specificity of the ML method was slightly better than the LMM method

## Evidence

- [methods] Fixed effects plus random intercept: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [abstract] Metadata as fixed effects: "patient metadata as it contains rich information of subject characteristics, which can be used to model and normalize metabolite abundances"
- [results] Variance reduction result: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [results] Post-fit boxplot criterion: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [discussion] False-positive caveat: "albeit with the potential introduction of false positives"
- [readme] README installation: "Install lmm2met using the following commands"
