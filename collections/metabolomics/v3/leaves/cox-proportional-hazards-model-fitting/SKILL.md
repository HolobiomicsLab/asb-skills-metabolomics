---
name: cox-proportional-hazards-model-fitting
description: Use when individual-level cohort data with time-to-death follow-up (age
  at blood sampling plus mortality status) must be modeled to estimate hazard ratios
  or to extract Cox coefficients as weights for a mortality risk score.
when_to_use_negative:
- You only have summary-level association statistics and no individual-level time-to-event
  data — use the published meta-analysis summary statistics instead of refitting Cox
  models
- 'The goal is a clinically deployable classifier: the score is built on per-cohort
  scaled biomarker values and is not yet suitable for classification of patients in
  the clinic'
- The endpoint is a non-mortality outcome or a short-time-horizon risk where the 5-10
  year all-cause mortality framing and age-as-time-scale design do not apply
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Fits the Cox proportional hazards models with age at blood sampling as the
    time scale (survival analysis), computes custom-made C-statistic discrimination
    measures, and assembles results tables
- name: STATA/SE 11.2
  role: Alternative environment for performing the cohort survival analyses alongside
    R
- name: R-package meta
  role: Implements the fixed-effect meta-analyses underlying the stepwise (forward-backward)
    biomarker selection whose ln(hazard ratio) results supply the score weights
- name: Nightingale Health NMR metabolomics platform
  role: Provides the plasma/serum metabolic biomarker concentrations that serve as
    covariates in the Cox models
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:23dd84145d56bff5b9c9ec4ede7a4d54cdc9b50c1840784f37bf3f2e30acb749
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  - R-package meta
  - STATA/SE 11.2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cox-proportional-hazards-model-fitting

## Summary

Fits Cox proportional hazards regression on individual-level cohort data to quantify biomarker or risk-factor associations with all-cause mortality and to derive weighted risk-score coefficients, using age at blood sampling as the time scale in R (survival) or STATA/SE 11.2.

## When to use

Use when individual-level cohort data with time-to-death follow-up (age at blood sampling plus mortality status) must be modeled to estimate hazard ratios or to extract Cox coefficients as weights for a mortality risk score.

## When NOT to use

- You only have summary-level association statistics and no individual-level time-to-event data — use the published meta-analysis summary statistics instead of refitting Cox models
- The goal is a clinically deployable classifier: the score is built on per-cohort scaled biomarker values and is not yet suitable for classification of patients in the clinic
- The endpoint is a non-mortality outcome or a short-time-horizon risk where the 5-10 year all-cause mortality framing and age-as-time-scale design do not apply

## Inputs

- Individual-level cohort dataset with covariates (e.g., sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes) and time-to-event follow-up keyed to age at blood sampling
- Nightingale NMR metabolic biomarker measurements (EDTA plasma or serum), log-transformed and scaled to SD units per study
- Training weights source cohort (Estonian Biobank) and independent validation cohort (FINRISK 1997, 7603 individuals, 1213 deaths)

## Outputs

- Cox regression coefficient table (hazard ratios / ln(HR) per covariate) usable as risk-score weights (Supplementary Table 4 scheme)
- Weighted linear-predictor risk score per individual (metabolic biomarker score and/or conventional risk factor score)
- Harrell C-statistics for 5- and 10-year mortality (all ages and >60 years) and IDI values for model comparison

## How to apply

Load the individual-level cohort data (e.g., the Estonian Biobank cohort with conventional risk factors: sex, body mass index, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes, prevalent CVD, prevalent cancer) together with mortality follow-up, and log-transform/scale biomarkers per study (adding 1 to biomarkers containing zeroes: x+1) before modeling. Fit a Cox proportional hazards model with age at blood sampling as the time scale so age itself is excluded from the score covariates; for the biomarker score, first select independently associated biomarkers with a stepwise (forward-backward) procedure on successive rounds of fixed-effect meta-analysis (R-package meta), applying the Bonferroni threshold of 2.21 x 10^-4 (0.05/226). Extract the fitted Cox coefficients as the score weights — for the conventional risk factor score these follow the Supplementary Table 4 scheme derived from the Estonian Biobank to avoid overestimation — and construct scores by multiplying scaled biomarkers by ln(hazard ratio) weights and summing. Validate predictive ability by applying the weighted linear predictor to an independent cohort (FINRISK 1997: 7603 individuals, 1213 deaths) and computing C-statistics for 5- and 10-year mortality and IDI, including age-stratified analyses (<60, 60-70, 70-80, 80-90, >90 years). Perform all survival analyses in R and STATA/SE 11.2; benchmark the conventional score against reference C-statistics of 0.772 and 0.790 (all ages) and the biomarker-plus-sex model against 0.837 and 0.830.

## Related tools

- **R** (Fits the Cox proportional hazards models with age at blood sampling as the time scale (survival analysis), computes custom-made C-statistic discrimination measures, and assembles results tables)
- **STATA/SE 11.2** (Alternative environment for performing the cohort survival analyses alongside R)
- **R-package meta** (Implements the fixed-effect meta-analyses underlying the stepwise (forward-backward) biomarker selection whose ln(hazard ratio) results supply the score weights)
- **Nightingale Health NMR metabolomics platform** (Provides the plasma/serum metabolic biomarker concentrations that serve as covariates in the Cox models)

## Examples

```
library(survival); fit <- coxph(Surv(age_blood, age_blood + time_to_death, death) ~ sex + bmi + sbp + chol + hdl + tg + creatinine + smoking + alcohol + diabetes, data = estonian_biobank); weights <- coef(fit)
```

## Evaluation signals

- Reference C-statistics reproduced: conventional risk factor score in FINRISK 1997 should yield ~0.772 (5-year) and 0.790 (10-year) all ages, and ~0.626 and 0.650 for individuals >60 years
- Biomarker-plus-sex model should outperform the conventional model with C-statistics of 0.837 (5-year) and 0.830 (10-year)
- Score weights must be derived exclusively from the Estonian Biobank cohort (Supplementary Table 4 scheme) — no validation-cohort mortality data may enter the weights (overestimation check)
- Biomarker covariates entering the model must be preprocessed per study: x+1 for zeroes, log-transformed, and scaled to SD units; biomarkers significant only at P < 2.21 x 10^-4 (0.05/226)
- The model's time scale is age at blood sampling and age is excluded from the covariate vector

## Limitations

- Weights trained only on the Estonian Biobank and tested only in FINRISK 1997; broader validation cohorts are needed
- The metabolic biomarker score based on per-cohort scaled biomarker values is not yet suitable for classification of patients in the clinic; a score in concentration units from individual-level data is still needed
- The NMR platform captures only a fraction of the metabolites in human serum, and the predictive value of more complete platforms was not compared
- Use of the identified metabolic profile as a mortality predictor or surrogate endpoint needs further investigation before guiding patient care

## Evidence

- [discussion] Time scale definition and endpoint modeling: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Weight derivation from Estonian Biobank to avoid overestimation: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [discussion] Software environment for survival analyses: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
- [discussion] Biomarker preprocessing before modeling: [evidence span withheld — anchor status: not_found]
- [abstract] Expected predictive performance of the fitted models: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively)"
