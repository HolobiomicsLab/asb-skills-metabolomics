---
name: patient-metadata-driven-metabolite-modeling
description: Use when clinical metabolomics data (e.g., GC-TOF MS lipid profiles)
  contain patient metadata and subject-specific biological variation should be readjusted
  before multivariate analyses like PLS-DA, OPLS-DA, or PCA.
when_to_use_negative:
- Data are raw MS signals that still require pre-processing (noise reduction, peak
  matching, retention time correction) or have not yet been normalized, scaled, centered
  and log2-transformed
- No patient/subject metadata (tissue type, age, BMI, sex, tumor location and stage)
  is available to enter as fixed effects, so the LMM cannot be formulated
- The goal is raw-signal quality control rather than an additional data-processing
  step to readjust individual metabolite signals prior to multivariate analyses
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: R package providing fitLmm, plot(type='coeff') and getFixCoeff for LMM-based
    metabolite signal readjustment
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure on which lmm2met is implemented
- name: R
  role: Runtime environment for all analyses (installed via devtools::install_github('kwanjeeraw/lmm2met'))
- name: devtools
  role: Used to install the lmm2met package from GitHub
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:7f4c143073e440e3584397f837be70c70a711ce9d8107b2789463f8968bd1669
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - devtools
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# patient-metadata-driven-metabolite-modeling

## Summary

Fits each log2-transformed metabolite signal to a linear mixed-effects model (LMM) in which patient metadata (Sex, Age, BMI, Stage, Location, Tissue) are fixed effects and patient ID is a random intercept, using the lmm2met R package built on lme4. The fitted model readjusts metabolite abundances to reduce subject-specific variation prior to multivariate analyses such as PLS-DA.

## When to use

Use when clinical metabolomics data (e.g., GC-TOF MS lipid profiles) contain patient metadata and subject-specific biological variation should be readjusted before multivariate analyses like PLS-DA, OPLS-DA, or PCA.

## When NOT to use

- Data are raw MS signals that still require pre-processing (noise reduction, peak matching, retention time correction) or have not yet been normalized, scaled, centered and log2-transformed
- No patient/subject metadata (tissue type, age, BMI, sex, tumor location and stage) is available to enter as fixed effects, so the LMM cannot be formulated
- The goal is raw-signal quality control rather than an additional data-processing step to readjust individual metabolite signals prior to multivariate analyses

## Inputs

- Log2-transformed metabolomics data matrix (e.g., lipid abundances) with patient metadata columns (Sex, Age, BMI, Stage, Location, Tissue, Id)
- Adipose tissue lipid profiles of 59 colorectal cancer patients (VAT and SAT samples, Liesenfeld et al. 2015)
- Datasets available via Metabolomics Workbench PR000058 (ST000081) and PR000305 (ST000390)

## Outputs

- fitMet$fittedDat: processed metabolite profile matrix with fixed effects and subject-specific random effects accounted for
- fitted_metabolite_matrix.csv: exported readjusted metabolite abundance matrix
- fixed_effect_coefficients.csv: estimated metadata-factor coefficients
- Coefficient table and fixed-effect significance heatmaps (plot(fitMet, type='coeff'), getFixCoeff)
- Per-metabolite fitted LMM summaries (fitMet$completeMod)

## How to apply

Load the log2-transformed metabolomics dataframe, e.g. the adipose tissue lipid matrix with patient metadata columns plus 158 lipid abundances from 59 colorectal cancer patients (VAT and SAT). For each metabolite, fit the LMM y_ij = beta0 + mu0i + sum(beta_jk * X_ik) + epsilon_ij with the lmm2met function fitLmm, setting fixed effects fix=c('Sex','Age','BMI','Stage','Location','Tissue') and a random intercept random='(1|Id)', implemented on the lme4 fitting procedure. Inspect fitMet$completeMod[[1]] to verify convergence of the per-metabolite fits, then extract the processed matrix fitMet$fittedDat in which subject-specific random effects and fixed effects are accounted for. Render coefficient and fixed-effect significance heatmaps via plot(fitMet, type='coeff') and getFixCoeff. Rationale: patient metadata contains rich information of subject characteristics usable to model and normalize metabolite abundances, and LMM fitting systematically minimizes subject-specific variation in each metabolite.

## Related tools

- **lmm2met** (R package providing fitLmm, plot(type='coeff') and getFixCoeff for LMM-based metabolite signal readjustment) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure on which lmm2met is implemented)
- **R** (Runtime environment for all analyses (installed via devtools::install_github('kwanjeeraw/lmm2met')))
- **devtools** (Used to install the lmm2met package from GitHub)

## Examples

```
install_github('kwanjeeraw/lmm2met'); fitMet <- fitLmm(data=adipose_log2, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)')
```

## Evaluation signals

- Subject-wise metabolite signal variance is reduced after LMM fitting: median of subject-wise boxplots closer to zero and less variability within tissue type
- Variance of a metabolite across all patients is substantially reduced and outliers are corrected after fitting
- Per-metabolite models in fitMet$completeMod[[1]] converge without warnings
- PCA of LMM-fitted data shows better separation between SAT and VAT (e.g., ~6% increase in explained variation on PC1 vs. original data)
- Downstream PLS-DA/OPLS-DA on LMM-fitted data improves classification accuracy, precision, sensitivity and specificity relative to unfitted data

## Limitations

- LMM processing may introduce false positives due to increased sensitivity
- Performance gains are preliminary at the current stage; further experiments are recommended to validate the results
- On the GC-TOF MS dataset, the ML method was slightly better than the LMM method in specificity
- Some signals (e.g., cholesterol) may be indistinguishable before fitting, so improvement cannot be assessed per-feature without the readjusted matrix

## Evidence

- [methods] LMM formulation with metadata fixed effects: [evidence span withheld — anchor status: not_found]
- [methods] lmm2met implemented on lme4: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [results] Variance reduction after fitting: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [abstract] Proposed processing step: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses"
- [readme] Package installation from README: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
