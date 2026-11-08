---
name: conventional-risk-factor-score-construction
description: 'Use when building a conventional-risk-factor comparison score for mortality
  prediction: Cox regression on sex, BMI, SBP, cholesterol, HDL, triglycerides, creatinine,
  smoking, alcohol, diabetes with age as time scale.'
when_to_use_negative:
- The goal is to score metabolic biomarkers rather than conventional risk factors
  — use the metabolic biomarker score pipeline (ln(hazard ratio)-weighted scaled biomarkers)
  instead
- Age is intended as a score component — in this method age is reserved as the Cox
  model time scale, not a covariate weight
- The training and validation cohorts overlap or the weights are derived in the evaluation
  cohort itself, which the article explicitly avoids to prevent overestimation
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: R
  role: Survival analysis environment for fitting the Cox proportional hazards model
    and computing Harrell C-statistics with custom-made discrimination functions
- name: STATA/SE 11.2
  role: Alternative environment used alongside R for the cohort survival analyses
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:50735bf1011e6d3d83ab8cf9edefa4fde04dc597af4d75cd065781232d073ba6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STATA/SE 11.2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# conventional-risk-factor-score-construction

## Summary

Fits a Cox proportional hazards model on conventional clinical risk factors in a training cohort (Estonian Biobank) and extracts its coefficients as a weighted risk score for mortality prediction. Serves as the comparator risk score against the metabolic biomarker score.

## When to use

Use when building a conventional-risk-factor comparison score for mortality prediction: Cox regression on sex, BMI, SBP, cholesterol, HDL, triglycerides, creatinine, smoking, alcohol, diabetes with age as time scale.

## When NOT to use

- The goal is to score metabolic biomarkers rather than conventional risk factors — use the metabolic biomarker score pipeline (ln(hazard ratio)-weighted scaled biomarkers) instead
- Age is intended as a score component — in this method age is reserved as the Cox model time scale, not a covariate weight
- The training and validation cohorts overlap or the weights are derived in the evaluation cohort itself, which the article explicitly avoids to prevent overestimation

## Inputs

- Estonian Biobank cohort individual-level data with conventional risk factors (sex, BMI, SBP, total cholesterol, HDL, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes) and mortality follow-up
- FINRISK 1997 validation cohort individual-level data (7603 individuals, 1213 deaths) with the same risk-factor covariates
- Mortality event times/censoring indicators for both cohorts

## Outputs

- Vector of Cox model coefficients used as score weights (Supplementary Table 4 weighting scheme)
- Per-individual weighted linear-predictor risk scores in the FINRISK 1997 cohort
- Harrell C-statistics for 5-year and 10-year mortality (all ages and >60 years), e.g. conventional_risk_factor_score_results.csv

## How to apply

Load individual-level training cohort data (Estonian Biobank) containing sex, body mass index, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, and prevalent diabetes, plus mortality follow-up. Fit a Cox proportional hazards model using age at blood sampling as the time scale — exclude age itself from the score because it serves as the time scale, and derive weights in a separate training cohort to avoid overestimation. Extract the fitted Cox coefficients as the score weights (the Supplementary Table 4 scheme). Apply the weighted linear predictor to the validation cohort (FINRISK 1997: 7603 individuals, 1213 deaths) and evaluate with Harrell C-statistics for 5- and 10-year mortality, all ages and restricted to >60 years, using custom R discrimination functions. Expected reference values: C = 0.772 (5-year) and 0.790 (10-year) all ages; 0.626 and 0.650 for >60 — the benchmark against which the biomarker score (C = 0.837 and 0.830) is compared.

## Related tools

- **R** (Survival analysis environment for fitting the Cox proportional hazards model and computing Harrell C-statistics with custom-made discrimination functions)
- **STATA/SE 11.2** (Alternative environment used alongside R for the cohort survival analyses)

## Examples

```
cph <- coxph(Surv(age_blood, age_death, death) ~ sex + bmi + sbp + totchol + hdl + trig + creatinine + smoking + alcohol + diabetes, data = estonian_biobank); score_finrisk <- as.matrix(finrisk1997[, covariates]) %*% coef(cph)
```

## Evaluation signals

- Cox model time scale is age at blood sampling, and age does not appear as a covariate in the score
- Score weights reproduce the Estonian Biobank-derived weighting scheme of Supplementary Table 4
- Applied weighted linear predictor in FINRISK 1997 yields Harrell C-statistics of 0.772 (5-year) and 0.790 (10-year) all ages, and 0.626 / 0.650 for individuals >60 years
- Score contains exactly the ten conventional covariates: sex, BMI, SBP, total cholesterol, HDL, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes
- Result table (e.g. conventional_risk_factor_score_results.csv) assembles the four C-statistics for direct comparison with the biomarker score values (0.837 and 0.830)

## Limitations

- Weights are trained only on the Estonian Biobank cohort and predictive ability tested only in FINRISK 1997, limiting generalizability of the score
- The conventional risk score underperforms the metabolic biomarker score, especially at higher ages where risk factors like systolic blood pressure and total cholesterol show opposite associations

## Evidence

- [discussion] Weights derived from Estonian Biobank training cohort: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [discussion] Cox model specification with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [intro] Validation cohort size and mortality events: "In the FINRISK 1997 cohort, consisting of 7603 individuals of whom 1213 died during follow-up, we compare the predictive value of a score based on the identified mortality-associated biomarkers"
- [abstract] Comparison benchmark C-statistics against conventional risk factor model: [evidence span withheld — anchor status: not_found]
- [discussion] Evaluation measures and software environment: "We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
