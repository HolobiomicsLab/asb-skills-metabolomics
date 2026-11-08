---
name: cross-cohort-risk-score-validation
description: Use when a risk score's weights are fitted in a training cohort (e.g.
  Estonian Biobank, Supplementary Table 4) and must be tested for 5- and 10-year mortality
  prediction in a separate validation cohort (e.g. FINRISK 1997, 7603 individuals,
  1213 deaths).
when_to_use_negative:
- The goal is clinical patient classification — the score is based on scaled biomarker
  values created separately per cohort, so it is not yet suitable for classifying
  patients in the clinic
- The same cohort is used for both weight estimation and predictive testing — this
  inflates performance (the paper derives weights in the Estonian Biobank specifically
  to avoid overestimation)
- The outcome is a short-term endpoint rather than intermediate- and long-term (5–10
  years) all-cause mortality
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: R
  role: Cox proportional hazards model fitting with age at blood sampling as time
    scale and custom-made discrimination (C-statistic) functions
- name: STATA/SE 11.2
  role: Cohort-level survival analyses alongside R
- name: R-package meta
  role: Fixed-effect meta-analysis used to derive score weights and implement IDI
    comparisons
- name: Nightingale Health NMR metabolomics platform
  role: Source of the standardized metabolic biomarker measurements scored in the
    validation cohort
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:adbddfabe0722c3303929e70286d0d7012e214c0712b41f9abafc220e801df5a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  - R-package meta
  - STATA/SE
  - STATA/SE 11.2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-cohort-risk-score-validation

## Summary

Trains a weighted mortality risk score in one biobank cohort and validates its predictive accuracy out-of-cohort, mirroring the Deelen et al. design where Cox-derived weights from the Estonian Biobank were tested in FINRISK 1997 using Harrell C-statistics and integrated discrimination improvement (IDI).

## When to use

Use when a risk score's weights are fitted in a training cohort (e.g. Estonian Biobank, Supplementary Table 4) and must be tested for 5- and 10-year mortality prediction in a separate validation cohort (e.g. FINRISK 1997, 7603 individuals, 1213 deaths).

## When NOT to use

- The goal is clinical patient classification — the score is based on scaled biomarker values created separately per cohort, so it is not yet suitable for classifying patients in the clinic
- The same cohort is used for both weight estimation and predictive testing — this inflates performance (the paper derives weights in the Estonian Biobank specifically to avoid overestimation)
- The outcome is a short-term endpoint rather than intermediate- and long-term (5–10 years) all-cause mortality

## Inputs

- Estonian Biobank cohort individual-level data with conventional risk factors (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes, prevalent CVD, prevalent cancer) and mortality follow-up
- FINRISK 1997 validation cohort (7603 individuals, 1213 deaths) with log-transformed, SD-scaled per-cohort biomarker measurements and mortality follow-up
- Score weights derived from the training cohort (Supplementary Table 4) or ln(hazard ratio) values from Table 2 meta-analyses

## Outputs

- Harrell C-statistics for 5-year and 10-year mortality, for all ages and restricted to individuals above 60 years
- Integrated discrimination improvement (IDI) values comparing nested risk score models
- Results table (e.g. conventional_risk_factor_score_results.csv) assembling the four C-statistics per score

## How to apply

Fit a Cox proportional hazards model on the training cohort (Estonian Biobank) using age at blood sampling as the time scale, excluding age from the score itself because it serves as the time scale; extract the Cox coefficients as the score weights (ln(hazard ratio) from the meta-analytic results, Supplementary Table 4) to avoid overestimation. Apply the weighted linear predictor — scaled biomarkers multiplied by their ln(hazard ratio) weights and summed — to each individual in the FINRISK 1997 validation cohort. Compute Harrell C-statistics for 5-year and 10-year mortality, both for all ages and restricted to individuals above 60 years, using the custom-made discrimination functions in R described in the Methods; additionally compute integrated discrimination improvement (IDI) with the R-package meta when comparing nested scores. Benchmark the score against a conventional risk factor score (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, prevalent diabetes) trained identically — expected reference C-statistics are 0.772/0.790 (5-/10-year) for the conventional score versus 0.837/0.830 for the biomarker-plus-sex score.

## Related tools

- **R** (Cox proportional hazards model fitting with age at blood sampling as time scale and custom-made discrimination (C-statistic) functions)
- **STATA/SE 11.2** (Cohort-level survival analyses alongside R)
- **R-package meta** (Fixed-effect meta-analysis used to derive score weights and implement IDI comparisons)
- **Nightingale Health NMR metabolomics platform** (Source of the standardized metabolic biomarker measurements scored in the validation cohort)

## Examples

```
fit <- coxph(Surv(age, time, death) ~ score, data=finrisk1997); c5 <- harrell_C(fit, times=5*365); c10 <- harrell_C(fit, times=10*365)
```

## Evaluation signals

- The biomarker-plus-sex model must yield C-statistic = 0.837 (5-year) and 0.830 (10-year) in FINRISK 1997; the conventional risk factor score must yield 0.772 and 0.790 (all ages), and 0.626 and 0.650 restricted to individuals above 60 years
- Adding conventional risk factors to the 14-biomarker model must NOT further improve prediction accuracy (combined model must not exceed the biomarker-only C-statistic)
- Score weights must come from the training cohort only (Estonian Biobank, Supplementary Table 4), never from the FINRISK 1997 validation data
- Cox models must use age at blood sampling as the time scale, with age excluded from the score covariates
- Per-cohort biomarker preprocessing must be reproduced: x+1 for zero-containing biomarkers, then log-transformation and scaling to standard deviation units separately per study

## Limitations

- Risk score weights are trained only on the Estonian Biobank and tested only in FINRISK 1997 — broader external validation is lacking
- The metabolic biomarker score is not yet suitable for clinical patient classification because it is based on scaled biomarker values created separately for each cohort; a score in concentration units from individual-level data is still needed
- The targeted NMR platform captures only a fraction of the metabolites in human serum, so the attainable prediction ceiling with this biomarker set is limited
- Validation in relevant clinical settings is required before the metabolic profile could be used to guide patient care

## Evidence

- [discussion] Weights trained out-of-cohort: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [discussion] Validation measures: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [discussion] Cox time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [abstract] Headline C-statistics: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [intro] Validation cohort size: "In the FINRISK 1997 cohort, consisting of 7603 individuals of whom 1213 died during follow-up, we compare the predictive value of a score based on the identified mortality-associated biomarkers"
