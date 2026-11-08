---
name: random-intercept-paired-data-modeling
description: Use when metabolomics data contain paired/repeated samples per subject
  (e.g. VAT and SAT lipid profiles from the same 59 CRC patients) and per-metabolite
  variance or outliers must be reduced using subject metadata.
when_to_use_negative:
- Data are unpaired single-sample-per-subject profiles with no repeated measurements
  or subject Id, so a random intercept '(1|Id)' cannot be defined
- No patient metadata are available to serve as fixed effects, leaving nothing for
  the LMM to adjust beyond the random intercept
- The input is already a normalized/processed feature table intended directly for
  classification, and you only need feature selection (e.g. VIP > 1) rather than signal
  readjustment
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: R package providing fitLmm to fit a linear mixed-effects model per metabolite
    and getFixCoeff to extract coefficients and significances
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure (lmer) on which lmm2met is implemented
- name: R
  role: Runtime environment in which the LMM fitting is performed
  repo: https://www.r-project.org/
- name: ggplot2
  role: Used to render before-vs-after subject-wise boxplots of fitted metabolite
    signals colored by tissue type (SC vs VS)
- name: devtools
  role: Used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met')
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:10cf24f8c44182ae76374fc375228398c4ccd4293688f8b8c2606eafe7bdba6e
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# random-intercept-paired-data-modeling

## Summary

Fit a per-metabolite linear mixed-effects model (LMM) with patient metadata as fixed effects and a subject-specific random intercept '(1|Id)', using the lmm2met R package built on lme4's lmer procedure, to systematically minimize subject-specific variation in each metabolite prior to multivariate analyses.

## When to use

Use when metabolomics data contain paired/repeated samples per subject (e.g. VAT and SAT lipid profiles from the same 59 CRC patients) and per-metabolite variance or outliers must be reduced using subject metadata.

## When NOT to use

- Data are unpaired single-sample-per-subject profiles with no repeated measurements or subject Id, so a random intercept '(1|Id)' cannot be defined
- No patient metadata are available to serve as fixed effects, leaving nothing for the LMM to adjust beyond the random intercept
- The input is already a normalized/processed feature table intended directly for classification, and you only need feature selection (e.g. VIP > 1) rather than signal readjustment

## Inputs

- Log2-transformed metabolomics feature matrix (metabolites/lipids x samples), e.g. 158 log2-transformed lipids from SAT and VAT of 59 CRC patients
- Patient metadata table (subject characteristics such as Sex, Age, BMI, Stage, Location, Tissue, and subject Id) used to formulate the LMMs

## Outputs

- LMM-fitted metabolite matrix (fitMet$fittedDat) with subject-specific variation minimized per metabolite
- Per-metabolite fixed-effect coefficients and chi-square significance values from getFixCoeff (e.g. cholesterol VAT coefficient 0.05, p = 0.05)
- Diagnostic plots: fixed-effect coefficient heatmaps (type='fixeff'), significance levels (type='coeff'), and before-vs-after subject-wise boxplots (type='fitted')

## How to apply

Load the log2-transformed metabolite/lipid matrix with its patient metadata (in the article, 158 log2-transformed lipids from SAT and VAT of 59 CRC patients, Metabolomics Workbench ST000081 or the packaged lmm2met 'adipose' data) and call lmm2met::fitLmm with fix=c('Sex','Age','BMI','Stage','Location','Tissue') and random='(1|Id)'. The function fits one LMM per metabolite using lme4's lmer; inspect the first model with summary(fitMet$completeMod[[1]]) and extract the processed matrix from fitMet$fittedDat, which readjusts each metabolite signal for the fixed effects and subject-specific random intercepts. Visualize fixed-effect coefficients and their significance via plot(fitMet, type='fixeff') and plot(fitMet, type='coeff'), or extract them programmatically with getFixCoeff (coefficients plus chi-square-test significances, e.g. cholesterol VAT coefficient 0.05, p = .05 in Table S-2). The rationale is that patient metadata contain rich subject characteristics usable to model and normalize metabolite abundances as an additional processing step before PCA/PLS-DA/OPLS-DA.

## Related tools

- **lmm2met** (R package providing fitLmm to fit a linear mixed-effects model per metabolite and getFixCoeff to extract coefficients and significances) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure (lmer) on which lmm2met is implemented)
- **R** (Runtime environment in which the LMM fitting is performed) — https://www.r-project.org/
- **ggplot2** (Used to render before-vs-after subject-wise boxplots of fitted metabolite signals colored by tissue type (SC vs VS))
- **devtools** (Used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met'))

## Examples

```
fitMet <- lmm2met::fitLmm(data, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)')
```

## Evaluation signals

- Per-metabolite variance across all patients is substantially reduced after LMM fitting, and outliers are corrected (reproduced via boxplots of signals before vs after fitting, Figure S-1 / Fig. 3a)
- Subject-wise boxplots after LMM fitting show a median closer to zero and less variability within tissue type
- Known effect sizes are recovered: cholesterol VAT fixed-effect coefficient = 0.05 with chi-square p-value = 0.05, matching Table S-2
- PCA of LMM-fitted data shows better SAT-vs-VAT separation, with a 6% increase in explained variation on PC1 relative to the original data
- summary(fitMet$completeMod[[1]]) returns a converged lme4 model for the first metabolite with the specified fixed effects and random intercept

## Limitations

- Increased sensitivity of downstream PLS-DA may come with the potential introduction of false positives
- Further experiments are recommended to validate the results, and performance claims are qualified as being at the current (preliminary) stage
- On the GC-TOF MS dataset, the LMM method did not outperform the multilevel (ML) method for specificity

## Evidence

- [methods] We developed the R package lmm2met to process a metabolomics matrix with the LMM approach.: "We developed the R package lmm2met to process a metabolomics matrix with the LMM approach."
- [methods] The package was implemented on a linear model-fitting procedure of the lme4 package: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [results] variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting: "variance of a metabolite across all patients was substantially reduced and outliers were corrected after LMM fitting"
- [abstract] we highlight the importance of patient metadata as it contains rich information of subject characteristics: "we highlight the importance of patient metadata as it contains rich information of subject characteristics"
- [readme] Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada.: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
