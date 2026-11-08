---
name: mortality-covariate-handling
description: Use when constructing or comparing weighted mortality risk scores from
  longitudinal cohort data where age must serve as the Cox time scale rather than
  a covariate, and weights must be trained on an independent training cohort.
when_to_use_negative:
- When the goal is identifying which biomarkers independently associate with mortality
  — that uses the stepwise forward-backward meta-analysis procedure with the 2.21
  × 10^-4 Bonferroni threshold, not this covariate-specification skill
- When the analysis treats age as a standard covariate (e.g. time-on-study models)
  — here age must be the time scale and is excluded from the score
- When the aim is clinical patient classification — the score is based on per-cohort
  scaled biomarker values, not concentration units, and is not yet suitable for the
  clinic
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: R
  role: Fits the Cox proportional hazards model with age at blood sampling as time
    scale, extracts score weights, and computes Harrell C-statistics with custom-made
    discrimination functions
- name: STATA/SE 11.2
  role: Alternative environment for the individual-cohort survival analyses
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:9edad8551008e39f4f1e42bbce46e8f361b14c0697bc4093f04a79d1b0a55ab0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STATA/SE 11.2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mortality-covariate-handling

## Summary

Handles covariate specification for mortality risk scores: fits a Cox proportional hazards model with age at blood sampling as the time scale (so age is never a score covariate), derives score weights from the Estonian Biobank to avoid overestimation, and evaluates via Harrell C-statistics in FINRISK 1997.

## When to use

Use when constructing or comparing weighted mortality risk scores from longitudinal cohort data where age must serve as the Cox time scale rather than a covariate, and weights must be trained on an independent training cohort.

## When NOT to use

- When the goal is identifying which biomarkers independently associate with mortality — that uses the stepwise forward-backward meta-analysis procedure with the 2.21 × 10^-4 Bonferroni threshold, not this covariate-specification skill
- When the analysis treats age as a standard covariate (e.g. time-on-study models) — here age must be the time scale and is excluded from the score
- When the aim is clinical patient classification — the score is based on per-cohort scaled biomarker values, not concentration units, and is not yet suitable for the clinic

## Inputs

- Estonian Biobank cohort individual-level data with conventional risk factors (sex, BMI, systolic blood pressure, total cholesterol, HDL, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes, prevalent CVD, prevalent cancer) and mortality follow-up
- FINRISK 1997 cohort covariate and biomarker values (validation set)
- Log-transformed, per-study scaled biomarker values (x+1 for zero-containing biomarkers)

## Outputs

- Cox coefficient weights for each score covariate (Supplementary Table 4 weighting scheme)
- Weighted linear-predictor risk score per individual
- Harrell C-statistics for 5-year and 10-year mortality, all ages and age >60 (e.g. conventional_risk_factor_score_results.csv)

## How to apply

Fit a Cox proportional hazards model with age at blood sampling as the time scale, deliberately excluding age from the score because it is absorbed by the time scale. For the conventional risk factor comparison score, include sex, body mass index, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, and prevalent diabetes; extract the fitted Cox coefficients as weights (Supplementary Table 4 scheme). Derive the weights in the Estonian Biobank cohort — not in the validation cohort — to avoid overestimation. Apply the weighted linear predictor to FINRISK 1997 (7603 individuals, 1213 deaths) and compute Harrell C-statistics for 5-year and 10-year mortality, both for all ages and restricted to individuals above 60 years, using custom-made discrimination functions in R.

## Related tools

- **R** (Fits the Cox proportional hazards model with age at blood sampling as time scale, extracts score weights, and computes Harrell C-statistics with custom-made discrimination functions)
- **STATA/SE 11.2** (Alternative environment for the individual-cohort survival analyses)

## Examples

```
coxph(Surv(age_blood, age_blood + time, dead) ~ sex + bmi + sbp + chol + hdl + trig + creatinine + smoke + alcohol + diabetes, data = estbb) # age only via time scale
```

## Evaluation signals

- Age does not appear as a covariate in the fitted model — it is used exclusively as the Cox time scale
- The conventional risk factor score contains exactly the 10 listed covariates (sex, BMI, SBP, total cholesterol, HDL, triglycerides, creatinine, smoking, alcohol, prevalent diabetes)
- Weights reproduce the Estonian Biobank-derived Supplementary Table 4 scheme rather than coefficients refit on FINRISK 1997
- Reference C-statistics recovered: 0.772 and 0.790 (all ages) and 0.626 and 0.650 (age >60) for the conventional risk factor model
- Validation is run on FINRISK 1997 (n=7603, 1213 deaths) with both 5-year and 10-year horizons reported

## Limitations

- Score weights were trained only on the Estonian Biobank and predictive ability tested only in FINRISK 1997, limiting generalizability
- The metabolic biomarker score is based on scaled biomarker values created separately for each cohort, so it is not yet suitable for classification of patients in the clinic; a concentration-unit score remains to be created
- No code repository is mentioned; only summary statistics were made publicly available in the BBMRI-omics atlas, so exact C-statistic computation code must be reimplemented

## Evidence

- [discussion] Cox time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] training cohort choice: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [discussion] validation design: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [discussion] software: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
- [intro] conventional comparison: "we compare the predictive value of a score based on the identified mortality-associated biomarkers with a score based on conventional risk factors for mortality."
