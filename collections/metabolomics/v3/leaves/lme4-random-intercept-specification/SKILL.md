---
name: lme4-random-intercept-specification
description: Use when metabolomics profiles contain repeated measurements or clustered
  samples from the same patients/subjects (e.g. paired tissue types) and subject-specific
  variation must be absorbed before PLS-DA/OPLS-DA.
when_to_use_negative:
- The data matrix is already a processed feature table with no patient metadata or
  subject identifiers available to formulate fixed effects and a random intercept.
- The analysis goal is raw-signal quality control of MS data (noise reduction, peak
  matching, retention time correction) rather than adjustment of individual metabolite
  signals prior to multivariate analyses.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: lmm2met
  role: R package providing fitLmm, plot(type='coeff') and getFixCoeff for fitting
    per-metabolite LMMs with random intercepts
  repo: https://github.com/kwanjeeraw/lmm2met
- name: lme4
  role: underlying linear model-fitting procedure on which the lmm2met LMM estimation
    is implemented
- name: R
  role: runtime environment in which all LMM fitting analyses were performed
  repo: https://www.r-project.org/
- name: devtools
  role: used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met')
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:fa54505e39ccab7e9a311c250e42f50416283cb4e57027f32ebd92251c9c268f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - devtools
  - lme4
  - lmm2met
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# lme4-random-intercept-specification

## Summary

Specify patient/subject random intercepts, e.g. random='(1|Id)', when fitting per-metabolite linear mixed-effects models with the lmm2met package, which wraps the lme4 fitting procedure to readjust metabolite signals in clinical metabolomics data prior to multivariate analyses.

## When to use

Use when metabolomics profiles contain repeated measurements or clustered samples from the same patients/subjects (e.g. paired tissue types) and subject-specific variation must be absorbed before PLS-DA/OPLS-DA.

## When NOT to use

- The data matrix is already a processed feature table with no patient metadata or subject identifiers available to formulate fixed effects and a random intercept.
- The analysis goal is raw-signal quality control of MS data (noise reduction, peak matching, retention time correction) rather than adjustment of individual metabolite signals prior to multivariate analyses.

## Inputs

- log2-transformed metabolomics dataframe with patient metadata columns and metabolite/lipid abundance columns (e.g. adipose tissue lipid profiles of 59 colorectal cancer patients, 158 lipid abundance columns)
- metadata table with subject identifier (Id) and covariates such as Sex, Age, BMI, Stage, Location, Tissue

## Outputs

- fitMet$fittedDat: LMM-processed metabolite profile matrix (e.g. written to fitted_metabolite_matrix.csv)
- per-metabolite fitted model summaries (fitMet$completeMod) for convergence inspection
- fixed-effect coefficient table (e.g. fixed_effect_coefficients.csv) and coefficient/fixed-effect significance heatmaps

## How to apply

For each metabolite column of the log2-transformed metabolomics dataframe (metadata columns plus abundance columns), call the lmm2met function fitLmm with the LMM y_ij = beta0 + mu0i + sum(beta_jk * X_ik) + epsilon_ij, passing the metadata covariates as fixed effects (e.g. fix=c('Sex','Age','BMI','Stage','Location','Tissue')) and random='(1|Id)' so that a random intercept per patient Id is estimated on top of the lme4 fitting procedure. Inspect fitMet$completeMod[[1]] to verify that per-metabolite fits converged. Extract the processed matrix fitMet$fittedDat, in which subject-specific random effects and the fixed effects are accounted for, and optionally render coefficient tables and fixed-effect significance heatmaps via plot(fitMet, type='coeff') and getFixCoeff. The rationale is that modeling subject characteristics as fixed effects plus a subject random intercept systematically minimizes subject-specific variation in each metabolite signal.

## Related tools

- **lmm2met** (R package providing fitLmm, plot(type='coeff') and getFixCoeff for fitting per-metabolite LMMs with random intercepts) — https://github.com/kwanjeeraw/lmm2met
- **lme4** (underlying linear model-fitting procedure on which the lmm2met LMM estimation is implemented)
- **R** (runtime environment in which all LMM fitting analyses were performed) — https://www.r-project.org/
- **devtools** (used to install lmm2met from GitHub via install_github('kwanjeeraw/lmm2met'))

## Examples

```
fitMet <- fitLmm(dat, fix=c('Sex','Age','BMI','Stage','Location','Tissue'), random='(1|Id)'); write.csv(fitMet$fittedDat, 'fitted_metabolite_matrix.csv')
```

## Evaluation signals

- fitMet$completeMod[[1]] model summary shows convergence of the first per-metabolite LMM fit.
- After LMM fitting, subject-wise boxplots have a median closer to zero and show less variability within tissue type, indicating reduced subject-wise metabolite signal variance.
- PCA of the LMM-fitted data (fitMet$fittedDat) shows better separation between SAT and VAT with a ~6% increase in explained variation on PC1 compared with the original data.
- fitted_metabolite_matrix.csv has the same number of samples and metabolite columns as the input log2-transformed dataframe but reduced variance per metabolite.

## Limitations

- Metadata factors must be recorded per patient; the paper used tissue type, age, BMI, sex, tumor location and tumor stage as fixed effects, so missing metadata columns limit the LMM formulation.
- The LMM method slightly underperformed the ML method in specificity on the GC-TOF MS dataset.
- Improved sensitivity may come with the potential introduction of false positives.

## Evidence

- [methods] The package was implemented on a linear model-fitting procedure of the lme4 package: "The package was implemented on a linear model-fitting procedure of the lme4 package"
- [abstract] we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses.: "we propose an additional data processing step using linear mixed-effects modelling to readjust an individual metabolite signal prior to multivariate analyses."
- [results] the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type: "the subject-wise boxplots after LMM fitting had a median closer to zero and showed less variability within tissue type"
- [readme] install_github('kwanjeeraw/lmm2met'): "#Install lmm2met
install_github('kwanjeeraw/lmm2met')"
