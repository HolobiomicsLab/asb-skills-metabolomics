---
name: c-statistic-discrimination-analysis
description: Use when two weighted mortality/risk scores must be compared for discrimination
  in a prospective cohort — e.g., C-statistic differences for 5- and 10-year follow-up
  windows, optionally stratified by age.
when_to_use_negative:
- The goal is biomarker discovery rather than head-to-head comparison of two pre-specified
  risk scores — use the stepwise forward-backward selection procedure instead.
- 'The score is intended for clinical patient classification: the article warns the
  biomarker score ''is not yet suitable for classification of patients in the clinic,
  since it is based on scaled biomarker values created separately for each cohort''.'
- The outcome is not right-censored survival data amenable to a Cox model with age
  as the time scale.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for the survival analyses and computation of C-statistics and
    IDI
- name: R-package survival
  role: Fits Cox proportional hazards models with age at blood sampling as the time
    scale for 5- and 10-year mortality windows
- name: custom-made functions in R
  role: Compute integrated discrimination improvement (IDI) comparing the two risk
    scores
- name: STATA/SE 11.2
  role: Used alongside R for survival analyses in the individual cohorts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:41421d1a4ccf67f7c9c73b63a7aef5da54af04ca6c4c7c651406fc1e0784c76d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package survival
  - STATA/SE 11.2
  - custom-made functions in R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# c-statistic-discrimination-analysis

## Summary

Compares predictive discrimination of competing risk scores (e.g., a metabolic biomarker score vs. a conventional risk factor score) by fitting Cox proportional hazards models with age as the time scale and reporting Harrell's C-statistic differences and integrated discrimination improvement (IDI).

## When to use

Use when two weighted mortality/risk scores must be compared for discrimination in a prospective cohort — e.g., C-statistic differences for 5- and 10-year follow-up windows, optionally stratified by age.

## When NOT to use

- The goal is biomarker discovery rather than head-to-head comparison of two pre-specified risk scores — use the stepwise forward-backward selection procedure instead.
- The score is intended for clinical patient classification: the article warns the biomarker score 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'.
- The outcome is not right-censored survival data amenable to a Cox model with age as the time scale.

## Inputs

- Prospective cohort data with survival time, event status (e.g., 5- and 10-year all-cause mortality), and predictor variables
- Metabolic biomarker measurements (14 biomarkers quantified by the Nightingale Health NMR platform), log-transformed and scaled to SD units
- Score weights (ln(hazard ratio) values, e.g., derived from the Estonian Biobank cohort, Supplementary Table 4)
- Conventional risk factors: sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer

## Outputs

- C-statistics for each risk score per follow-up window (5- and 10-year) and age stratum (all, >60)
- C-statistic differences with standard errors and P values between biomarker and conventional scores
- IDI values with SEs and P values comparing the two scores
- Results table mirroring Table 4 of the article

## How to apply

Load the cohort (FINRISK 1997: 7603 individuals, 1213 deaths) with both sets of predictors. Construct weighted scores by log-transforming biomarkers, scaling to standard deviation units per study, multiplying by ln(hazard ratio) weights from the meta-analysis, and summing, adding sex for the metabolic score; build the conventional score from sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, and cancer, excluding age since age at blood sampling serves as the Cox time scale. Fit Cox proportional hazards models for 5-year and 10-year mortality windows, in all individuals and restricted to those above 60, and compute C-statistics for each score with the R survival package. Quantify the C-statistic difference between scores and compute IDI using custom-made R functions. Assemble C-statistics, differences (with SEs and P values), and IDI values into a table mirroring the paper's Table 4.

## Related tools

- **R** (Environment for the survival analyses and computation of C-statistics and IDI)
- **R-package survival** (Fits Cox proportional hazards models with age at blood sampling as the time scale for 5- and 10-year mortality windows)
- **custom-made functions in R** (Compute integrated discrimination improvement (IDI) comparing the two risk scores)
- **STATA/SE 11.2** (Used alongside R for survival analyses in the individual cohorts)

## Examples

```
library(survival); fit <- coxph(Surv(age_sampling, age_sampling + tlimit, death) ~ biomarker_score, data = finrisk1997); summary(concordance(fit))
```

## Evaluation signals

- Reproduced conventional score C-statistics ≈ 0.772 (5-year, all) and 0.790 (10-year, all).
- Reproduced metabolic biomarker score C-statistics ≈ 0.837 (5-year, all) and 0.830 (10-year, all).
- C-statistic differences ≈ 0.065 (5-year) and 0.040 (10-year) in all individuals, and larger in the >60 group (≈ 0.105 and 0.065).
- IDI consistently favors the biomarker score (≈ 5.9% for 5-year and 8.6% for 10-year mortality).
- Age at sampling is excluded from both scores and used as the Cox model time scale, not as a covariate.

## Limitations

- Score weights were trained only on the Estonian Biobank cohort and predictive ability tested only in FINRISK 1997; generalization beyond these settings is untested.
- The metabolic biomarker score is based on per-cohort scaled biomarker values and is not yet suitable for clinical patient classification; a score in concentration units is still needed.
- The targeted NMR platform captures only a fraction of metabolites in human serum; more complete platforms were not compared.

## Evidence

- [abstract] the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] Cox proportional hazards model with age at blood sampling as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Four weighted risk scores trained on Estonian Biobank and tested in FINRISK 1997 using C-statistics and IDI: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [discussion] Metabolic biomarker score constructed by multiplying scaled biomarkers by ln(hazard ratio) weights and summing: "the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed"
- [discussion] The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
