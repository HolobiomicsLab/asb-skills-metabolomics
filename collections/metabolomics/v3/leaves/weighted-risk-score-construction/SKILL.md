---
name: weighted-risk-score-construction
description: Use when you have pre-derived hazard-ratio weights and cohort-level metabolite/risk-factor
  measurements and need to build, apply, and comparatively evaluate weighted risk
  scores for mortality prediction (C-statistic, IDI) in an independent cohort.
when_to_use_negative:
- 'You need a clinically deployable score: the constructed metabolic biomarker score
  is based on scaled biomarker values created separately for each cohort and ''is
  not yet suitable for classification of patients in the clinic'''
- You are discovering which biomarkers associate with mortality (use the stepwise
  forward-backward meta-analysis selection procedure instead) rather than applying
  already-derived weights
- The biomarker weights come from the same cohort being scored, which would overestimate
  predictive ability
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for fitting Cox proportional hazards models and computing C-statistics;
    survival analyses in individual cohorts were performed using R
- name: R-package survival
  role: Cox proportional hazards fitting with age at blood sampling as the time scale
- name: custom-made functions in R
  role: Compute integrated discrimination improvement (IDI) comparing the biomarker
    and conventional scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:5772d3fcbe3682d8155da97930386428494446e9c991c74e7421049859e59084
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package survival
  - custom-made functions in R
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# weighted-risk-score-construction

## Summary

Construct weighted metabolic biomarker and conventional risk factor scores by multiplying log-transformed, SD-scaled biomarkers by ln(HR) weights derived from an external cohort (Estonian Biobank), then evaluate predictive ability in an independent cohort (FINRISK 1997) using Cox-based C-statistics and IDI.

## When to use

Use when you have pre-derived hazard-ratio weights and cohort-level metabolite/risk-factor measurements and need to build, apply, and comparatively evaluate weighted risk scores for mortality prediction (C-statistic, IDI) in an independent cohort.

## When NOT to use

- You need a clinically deployable score: the constructed metabolic biomarker score is based on scaled biomarker values created separately for each cohort and 'is not yet suitable for classification of patients in the clinic'
- You are discovering which biomarkers associate with mortality (use the stepwise forward-backward meta-analysis selection procedure instead) rather than applying already-derived weights
- The biomarker weights come from the same cohort being scored, which would overestimate predictive ability

## Inputs

- Cohort dataset with NMR-platform metabolic biomarker concentrations (EDTA plasma or serum), e.g. FINRISK 1997 (7603 individuals, 1213 deaths)
- Conventional risk factor measurements: sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer
- External ln(HR) weights for each biomarker (Estonian Biobank-derived, Supplementary Table 4)
- Follow-up time and all-cause mortality status per individual

## Outputs

- Per-individual weighted metabolic biomarker score (14 scaled biomarkers × ln(HR) weights, summed, plus sex)
- Per-individual conventional risk factor score (age excluded, serving as time scale)
- C-statistics for each score per follow-up window (5-year, 10-year) and age stratum (all, >60), with differences, SEs, and P values
- IDI values comparing the two scores (with SEs and P values), assembled into a results table mirroring Table 4

## How to apply

Load the target cohort (FINRISK 1997: 7603 individuals, 1213 deaths) with the 14 identified metabolic biomarkers, sex, and the conventional risk factors (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, and cancer). Preprocess biomarkers by adding 1 to any biomarker containing zeroes (x+1), log-transforming, and scaling to standard deviation units separately per study; apply ln(HR) weights from the Estonian Biobank-derived meta-analysis (Supplementary Table 4) to each scaled biomarker, sum them, and add sex to form the metabolic biomarker score; construct the conventional risk factor score excluding age at sampling since it serves as the Cox time scale. Fit Cox proportional hazards models with age at blood sampling as the time scale for 5-year and 10-year mortality windows, in all individuals and restricted to those above 60, and compute C-statistics for each score, then compute integrated discrimination improvement (IDI) with custom-made R functions. Weights are taken from an external cohort (Estonian Biobank) specifically to avoid overestimation of predictive ability in the test cohort.

## Related tools

- **R** (Environment for fitting Cox proportional hazards models and computing C-statistics; survival analyses in individual cohorts were performed using R)
- **R-package survival** (Cox proportional hazards fitting with age at blood sampling as the time scale)
- **custom-made functions in R** (Compute integrated discrimination improvement (IDI) comparing the biomarker and conventional scores)

## Examples

```
library(survival); coxph(Surv(age_sampling, age_sampling + 5, death) ~ biomarker_score, data = finrisk1997)
```

## Evaluation signals

- Reproduced 5-year C-statistics in FINRISK 1997 (all individuals): conventional score ≈ 0.772, metabolic biomarker score ≈ 0.837, difference ≈ 0.065
- Reproduced 10-year C-statistics (all individuals): conventional ≈ 0.790, biomarker ≈ 0.830, difference ≈ 0.040
- Age-stratified (>60) differences larger: ≈ 0.105 (5-year) and 0.065 (10-year)
- IDI consistently higher for the biomarker model: ≈ 5.9% (5-year) and 8.6% (10-year)
- Sanity check on preprocessing: biomarkers containing zeroes had 1 added before log-transformation, and scaling was done to SD units separately per study

## Limitations

- Weights are based only on the Estonian Biobank cohort and tested only in FINRISK 1997; broader validation needed
- The score 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'
- A score in concentration units from individual-level data would be needed for clinical research use
- The targeted NMR platform captures only a fraction of metabolites in human serum; more complete platforms were not compared

## Evidence

- [discussion] the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed: "the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed"
- [discussion] weights trained externally to avoid overestimation: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [discussion] two predictive-value measures: "We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [discussion] biomarker preprocessing: [evidence span withheld — anchor status: not_found]
- [methods] Cox model specification: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [abstract] headline C-statistic results: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] clinical-use caveat: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
