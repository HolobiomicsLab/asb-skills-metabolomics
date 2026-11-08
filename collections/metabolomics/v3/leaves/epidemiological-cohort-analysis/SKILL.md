---
name: epidemiological-cohort-analysis
description: Use when the input is an individual-level cohort dataset (exposure biomarkers
  plus time-to-death and cause-of-death endpoints) and the goal is cause-specific
  or all-cause mortality hazard ratios per biomarker.
when_to_use_negative:
- The goal is multivariable biomarker selection over many correlated metabolites —
  that requires the paper's stepwise forward–backward meta-analysis procedure, not
  single-biomarker Cox fits
- The data are cross-sectional or lack follow-up time-to-event information, so no
  proportional hazards time scale can be defined
- The task is predictive model evaluation (C-statistics, IDI) in a held-out cohort
  rather than association testing
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Runs the per-cohort survival analyses; Cox models are fitted with the R survival
    package using age at blood sampling as the time scale
- name: STATA/SE 11.2
  role: Alternative environment in which the individual-cohort survival analyses were
    performed
- name: R-package meta
  role: Fixed-effect meta-analysis of cohort-specific Cox results when pooling across
    multiple cohorts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:d7b3afadbb83cb5db5f4adbe8cc9b44bdf5f30c0f3929cded0194ad1ca29492c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  - STATA/SE 11.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# epidemiological-cohort-analysis

## Summary

Fits Cox proportional hazards models with age at blood sampling as the time scale to estimate biomarker–mortality associations (HR, 95% CI, P) in prospective cohorts such as FINRISK 1997 (7603 individuals, 1213 deaths), optionally stratified by cause of death.

## When to use

Use when the input is an individual-level cohort dataset (exposure biomarkers plus time-to-death and cause-of-death endpoints) and the goal is cause-specific or all-cause mortality hazard ratios per biomarker.

## When NOT to use

- The goal is multivariable biomarker selection over many correlated metabolites — that requires the paper's stepwise forward–backward meta-analysis procedure, not single-biomarker Cox fits
- The data are cross-sectional or lack follow-up time-to-event information, so no proportional hazards time scale can be defined
- The task is predictive model evaluation (C-statistics, IDI) in a held-out cohort rather than association testing

## Inputs

- Individual-level cohort dataset (e.g., FINRISK 1997: 7603 individuals, 1213 deaths) with 14 metabolic biomarkers (XXL-VLDL-L, S-HDL-L, VLDL-D, PUFA/FA, Glc, Lac, His, Ile, Leu, Val, Phe, AcAce, Alb, GlycA)
- Cause-of-death endpoints coded as all-cause, cancer, cardiovascular, nonlocalized infections, and other
- Age at blood sampling (entry age) and age/status at death or censoring

## Outputs

- Table of hazard ratios, 95% confidence intervals, and P values per biomarker × cause-of-death pair (biomarker rows × cause-of-death columns, matching Table 3 layout)
- Direction-of-effect comparison across cause-specific outcomes versus all-cause mortality

## How to apply

Preprocess biomarkers as specified: add 1 to any values below the detection limit (x+1), log-transform, and scale to standard deviation units separately per study. For each biomarker and each mortality outcome (all-cause, cancer, cardiovascular, nonlocalized infections, other), fit a Cox proportional hazards model with age at blood sampling as the time scale using the R survival package (survival analyses were also run in STATA/SE 11.2). Collect hazard ratios, 95% confidence intervals, and P values per biomarker–outcome pair and assemble them into a table with biomarker rows and cause-of-death columns. The rationale is that age-at-entry time scales properly handle left-truncated cohort entry, and the 5-cause decomposition lets you test whether biomarkers represent generic immune-metabolic health adversity rather than one disease pathway. Validate by checking reproduced values against published anchors (e.g., GlycA all-cause HR 1.41, 95% CI 1.26–1.57, P 5.75e-10; PUFA/FA all-cause HR 0.73, 95% CI 0.69–0.78, P < 2.22e-16).

## Related tools

- **R** (Runs the per-cohort survival analyses; Cox models are fitted with the R survival package using age at blood sampling as the time scale)
- **STATA/SE 11.2** (Alternative environment in which the individual-cohort survival analyses were performed)
- **R-package meta** (Fixed-effect meta-analysis of cohort-specific Cox results when pooling across multiple cohorts)

## Examples

```
library(survival); m <- coxph(Surv(age, age_event, death_status) ~ GlycA_scaled, data = finrisk1997); summary(m)
```

## Evaluation signals

- GlycA all-cause mortality reproduces at HR 1.41, 95% CI 1.26–1.57, P 5.75e-10 in FINRISK 1997
- PUFA/FA all-cause mortality reproduces at HR 0.73, 95% CI 0.69–0.78, P < 2.22e-16 in FINRISK 1997
- Table contains all 14 biomarkers × 5 mortality outcomes (all-cause, cancer, cardiovascular, nonlocalized infections, other) with HRs, CIs, and P values
- The majority of the 14 biomarkers show associations with cause-specific outcomes in the same direction as all-cause mortality, including nonlocalized infections
- Time scale is age at blood sampling (left-truncated Surv object), not calendar follow-up time

## Limitations

- The Nightingale NMR platform captures only a fraction of metabolites in human serum, so associations are limited to the measured biomarker panel
- Cohort-specific scaling (per-study standard-deviation units) means the resulting biomarker score is not yet suitable for clinical patient classification
- Associations were established in a specific observational cohort; use as a predictor or surrogate endpoint needs further investigation

## Evidence

- [discussion] Cox time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] biomarker preprocessing: [evidence span withheld — anchor status: not_found]
- [intro] cohort size: "In the FINRISK 1997 cohort, consisting of 7603 individuals of whom 1213 died during follow-up"
- [discussion] analysis software: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
- [intro] rationale for cause-specific analysis: "predictors of intermediate- and long-term mortality should ideally represent generic immune-metabolic health adversity rather than only being indicators"
