---
name: metabolomics-abundance-readjustment
description: Use when a clinical metabolomics feature table (e.g. log2-transformed
  GC-TOF MS lipid abundances) still shows subject-specific variation/outliers after
  standard normalization, and patient metadata is available to model and readjust
  each metabolite signal before multivariate analyses.
when_to_use_negative:
- The input is already an LMM-processed (readjusted) feature matrix — re-fitting would
  remove biological signal rather than subject-specific variation.
- No patient/subject metadata is available to serve as fixed effects and no repeated-measures
  structure exists, so the LMM formulation cannot be specified.
- The task is raw MS data pre-processing (noise reduction, peak matching, retention
  time correction) — this skill is an additional post-processing step applied after
  normalization and log2 transformation.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: lmm2met
  role: R package that fits the per-metabolite LMM (fitLmm) and extracts the readjusted
    metabolite matrix and fixed-effect coefficients
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: Underlying linear model-fitting procedure on which lmm2met's LMM fitting is
    implemented
- name: R
  role: Runtime environment in which lmm2met is installed (via devtools::install_github)
    and executed
  repo: https://www.r-project.org/
- name: ggplot2
  role: Plotting library used for rendering coefficient and fixed-effect significance
    heatmaps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:57727de3c4d980d7348726b288b2883491004543e59c26e26b749ff8572cfee8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - ggplot2
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabolomics-abundance-readjustment

## Summary

An additional data-processing step that fits each metabolite in a log2-transformed clinical metabolomics matrix to a linear mixed-effects model (LMM), using patient metadata (e.g. Sex, Age, BMI, Stage, Location, Tissue) as fixed effects and a patient-level random intercept to readjust metabolite abundances prior to multivariate analyses such as PLS-DA and OPLS-DA.

## When to use

Use when a clinical metabolomics feature table (e.g. log2-transformed GC-TOF MS lipid abundances) still shows subject-specific variation/outliers after standard normalization, and patient metadata is available to model and readjust each metabolite signal before multivariate analyses.

## When NOT to use

- The input is already an LMM-processed (readjusted) feature matrix — re-fitting would remove biological signal rather than subject-specific variation.
- No patient/subject metadata is available to serve as fixed effects and no repeated-measures structure exists, so the LMM formulation cannot be specified.
- The task is raw MS data pre-processing (noise reduction, peak matching, retention time correction) — this skill is an additional post-processing step applied after normalization and log2 transformation.

## Inputs

- Log2-transformed metabolomics dataframe (e.g. adipose tissue lipid abundances of 59 CRC patients with patient metadata columns plus 158 lipid abundance columns)
- Patient metadata: subject characteristics such as sex, age, BMI, tumor stage, tumor location and tissue type (SAT/VAT)

## Outputs

- Readjusted metabolite profile matrix (fitMet$fittedDat), e.g. exported as fitted_metabolite_matrix.csv
- Per-metabolite fitted LMM objects and summaries (fitMet$completeMod)
- Fixed-effect coefficient table (e.g. fixed_effect_coefficients.csv) and coefficient/significance heatmaps via plot(fitMet, type='coeff') and getFixCoeff

## How to apply

Load the log2-transformed metabolomics dataframe including patient metadata columns (e.g. the adipose tutorial data: 59 colorectal cancer patients, VAT/SAT lipid profiles with 158 lipid abundance columns). For each metabolite, fit the LMM y_ij = beta0 + mu0i + sum(beta_jk * X_ik) + epsilon_ij using the lmm2met function fitLmm with fixed effects fix=c('Sex','Age','BMI','Stage','Location','Tissue') and a random intercept random='(1|Id)', implemented on the lme4 fitting procedure. Inspect the per-metabolite model summaries (e.g. fitMet$completeMod[[1]]) to verify convergence, then extract the readjusted profile matrix fitMet$fittedDat, in which subject-specific random effects and the given fixed effects are accounted for, and write it to CSV. Visualize fixed-effect coefficient and significance heatmaps via plot(fitMet, type='coeff') / getFixCoeff and export the coefficients. The rationale is that patient metadata contains rich information of subject characteristics that can be used to model and normalize metabolite abundances, systematically minimizing subject-specific variation before feature selection (e.g. VIP > 1) and PLS classification.

## Related tools

- **lmm2met** (R package that fits the per-metabolite LMM (fitLmm) and extracts the readjusted metabolite matrix and fixed-effect coefficients) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (Underlying linear model-fitting procedure on which lmm2met's LMM fitting is implemented)
- **R** (Runtime environment in which lmm2met is installed (via devtools::install_github) and executed) — https://www.r-project.org/
- **ggplot2** (Plotting library used for rendering coefficient and fixed-effect significance heatmaps)

## Examples

```
library(devtools); install_github('kwanjeeraw/lmm2met'); fitMet <- lmm2met::fitLmm(dat, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'); write.csv(fitMet$fittedDat, 'fitted_metabolite_matrix.csv')
```

## Evaluation signals

- After LMM fitting, subject-wise boxplots of each metabolite show a median closer to zero and less variability within tissue type compared to the original data.
- Variance of each metabolite across all patients is substantially reduced and outliers are corrected relative to the pre-fitting matrix.
- Per-metabolite model summaries (fitMet$completeMod[[1]] for the first metabolite) show converged lme4 fits without warnings.
- fitMet$fittedDat has the same metabolite dimensionality as the input (e.g. 158 lipids) but with subject-specific effects accounted for; PCA of the readjusted data separates SAT and VAT better (~6% increase in explained variation on PC1).
- Fixed-effect coefficient table is non-empty and matches the specified fixed effects (Sex, Age, BMI, Stage, Location, Tissue).

## Limitations

- The method did not outperform the multilevel (ML) method in specificity for the GC-TOF MS dataset.
- Increased sensitivity may come with the potential introduction of false positives, and further experiments are recommended to validate the results.
- Requires metabolomics data already pre-processed (normalized, log2-transformed) and paired, complete patient metadata; results at the current stage are preliminary.

## Evidence

- [abstract] we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses.: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
- [methods] The package was implemented on a linear model-fitting procedure of the lme4 package: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [methods] Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study.: "Both datasets were pre-processed as described previously [21,27] and log2-transformed prior to use in this study."
- [results] the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [readme] Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada.: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
