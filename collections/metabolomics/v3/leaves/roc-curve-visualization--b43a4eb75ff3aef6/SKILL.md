---
name: roc-curve-visualization
description: Use when two weighted risk scores must be compared for predictive accuracy
  of a time-to-event outcome with right censoring — e.g. biomarker vs conventional
  scores for 5-/10-year all-cause mortality in a cohort like FINRISK 1997.
when_to_use_negative:
- The risk scores were trained in the same cohort used for evaluation — weights must
  come from an independent cohort (here the Estonian Biobank) to avoid overestimation
  of predictive ability
- The outcome is a binary, uncensored endpoint — a standard ROC/AUC on case-control
  labels is then appropriate instead of a censoring-aware C-statistic from a Cox model
- The goal is clinical patient classification — the article explicitly states the
  biomarker score 'is not yet suitable for classification of patients in the clinic,
  since it is based on scaled biomarker values created separately for each cohort'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Environment for fitting the Cox proportional hazards survival analyses and
    computing C-statistics and IDI in the FINRISK 1997 cohort
- name: R-package survival
  role: Fits the Cox proportional hazards models with age at blood sampling as the
    time scale used to derive the C-statistics
- name: custom-made functions in R
  role: Compute integrated discrimination improvement (IDI) with SEs and P values
    comparing the two risk scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:ca897cb4bf56e44df8136499dba0cf0c9d6b98aef6c804ffc3ca91544466b4b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package survival
  - custom-made functions in R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# roc-curve-visualization

## Summary

Compares the discrimination of competing mortality risk scores in a censored cohort by computing Harrell's C-statistics and integrated discrimination improvement (IDI) from Cox proportional hazards models fitted with age at blood sampling as the time scale, producing a Table 4-style contrast (ΔC with SE/P, IDI with SE/P) for 5- and 10-year follow-up windows.

## When to use

Use when two weighted risk scores must be compared for predictive accuracy of a time-to-event outcome with right censoring — e.g. biomarker vs conventional scores for 5-/10-year all-cause mortality in a cohort like FINRISK 1997.

## When NOT to use

- The risk scores were trained in the same cohort used for evaluation — weights must come from an independent cohort (here the Estonian Biobank) to avoid overestimation of predictive ability
- The outcome is a binary, uncensored endpoint — a standard ROC/AUC on case-control labels is then appropriate instead of a censoring-aware C-statistic from a Cox model
- The goal is clinical patient classification — the article explicitly states the biomarker score 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'

## Inputs

- Individual-level FINRISK 1997 cohort data (7603 individuals, 1213 deaths) with follow-up time and death indicator
- 14 metabolic biomarkers measured on the Nightingale Health NMR platform, log-transformed and scaled to SD units per cohort
- Conventional risk factors: sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, and cancer
- Estonian Biobank-derived ln(hazard ratio) weights for the 14 biomarkers (Supplementary Table 4)

## Outputs

- C-statistics for the metabolic biomarker score and the conventional risk factor score, per follow-up window (5-year, 10-year) and per age stratum (all individuals, >60 years)
- C-statistic differences (biomarker score minus conventional score) with SEs and P values
- IDI values with SEs and P values from custom-made R functions
- A results table mirroring Table 4 of the article

## How to apply

Log-transform and scale each biomarker to standard deviation units per study (adding 1 to biomarkers containing zeroes, i.e. x+1), then multiply the 14 scaled biomarkers by their ln(hazard ratio) weights from the Estonian Biobank meta-analysis (Supplementary Table 4) and sum, adding sex; build the conventional risk factor score from sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, and cancer, deliberately excluding age at sampling because it serves as the time scale. Fit Cox proportional hazards models with age at blood sampling as the time scale in R using the survival package, separately for 5-year and 10-year mortality windows, in all 7603 FINRISK 1997 individuals (1213 deaths) and restricted to those above 60 years. Compute the C-statistic for each score per follow-up/age combination, report the C-statistic difference with its SE and P value, and quantify integrated discrimination improvement (IDI) with custom-made R functions. Assemble C-statistics, ΔC values, and IDI values (with SEs and P values) into a results table mirroring Table 4.

## Related tools

- **R** (Environment for fitting the Cox proportional hazards survival analyses and computing C-statistics and IDI in the FINRISK 1997 cohort)
- **R-package survival** (Fits the Cox proportional hazards models with age at blood sampling as the time scale used to derive the C-statistics)
- **custom-made functions in R** (Compute integrated discrimination improvement (IDI) with SEs and P values comparing the two risk scores)

## Evaluation signals

- Reproduced conventional score C-statistics match the reported 0.772 (5-year, all) and 0.790 (10-year, all)
- Reproduced metabolic biomarker score C-statistics match the reported 0.837 (5-year, all) and 0.830 (10-year, all)
- C-statistic differences match 0.065 (5-year) and 0.040 (10-year) in all individuals, and are larger in the >60 group (0.105 and 0.065)
- IDI is consistently positive and matches the reported 5.9% (5-year) and 8.6% (10-year), with IDI higher for the biomarker model across combinations
- The conventional risk factor score excludes age at sampling, since age serves as the Cox model time scale

## Limitations

- The metabolic biomarker score is not suitable for clinical patient classification because it is based on per-cohort scaled biomarker values; a score based on concentration units from individual-level data is still needed
- Risk score weights were trained only on the Estonian Biobank and predictive ability tested only in FINRISK 1997
- The targeted NMR platform captures only a fraction of the metabolites in human serum, and the predictive accuracy of more complete platforms was not compared
- Further validation in relevant clinical settings is required before use to guide patient care

## Evidence

- [abstract] C-statistic discrimination comparison: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830"
- [discussion] Two discrimination measures: "We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [discussion] Cox model specification: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Independent training of weights: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [discussion] Clinical classification caveat: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values"
