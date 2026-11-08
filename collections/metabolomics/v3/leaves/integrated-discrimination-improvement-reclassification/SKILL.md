---
name: integrated-discrimination-improvement-reclassification
description: Use when two nested or competing risk scores must be compared for predictive
  value of 5- or 10-year mortality (or similar survival endpoints), and you need C-statistic
  differences with SEs/P values plus IDI with SEs/P values.
when_to_use_negative:
- The metabolic biomarker score is based on per-cohort scaled biomarker values and
  is not yet suitable for clinical patient classification, so do not use these C-statistic/IDI
  comparisons to justify deploying the score as a clinical classifier.
- Do not apply this comparison when the new score was trained on the same cohort it
  is tested in — here weights come from the Estonian Biobank and are tested in FINRISK
  1997 specifically to avoid overestimation of predictive ability.
- Not appropriate for case-control or classification settings without a survival time
  scale; the method requires Cox models with age at blood sampling as the time scale.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for survival analyses, C-statistic computation, and IDI calculation
    with custom-made functions
- name: R-package survival
  role: Fits Cox proportional hazards models with age at blood sampling as the time
    scale for 5- and 10-year mortality windows
- name: custom-made functions in R
  role: Compute integrated discrimination improvement (IDI) and C-statistic differences
    with SEs and P values
- name: STATA/SE 11.2
  role: Used alongside R for the survival analyses in the individual cohorts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:fd9815ae675e6f9c8cce4b9e02d76ce74df670e1220947049f09f09ff01dcf22
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
n_claims: 8
---

# integrated-discrimination-improvement-reclassification

## Summary

Quantifies whether a new risk score (e.g., a metabolic biomarker score) improves prediction of a time-to-event outcome over a conventional score, using C-statistic differences and integrated discrimination improvement (IDI) computed with custom R functions in a Cox model framework.

## When to use

Use when two nested or competing risk scores must be compared for predictive value of 5- or 10-year mortality (or similar survival endpoints), and you need C-statistic differences with SEs/P values plus IDI with SEs/P values.

## When NOT to use

- The metabolic biomarker score is based on per-cohort scaled biomarker values and is not yet suitable for clinical patient classification, so do not use these C-statistic/IDI comparisons to justify deploying the score as a clinical classifier.
- Do not apply this comparison when the new score was trained on the same cohort it is tested in — here weights come from the Estonian Biobank and are tested in FINRISK 1997 specifically to avoid overestimation of predictive ability.
- Not appropriate for case-control or classification settings without a survival time scale; the method requires Cox models with age at blood sampling as the time scale.

## Inputs

- Cohort survival dataset with time-to-event and status (e.g., FINRISK 1997, 7603 individuals, 1213 deaths during follow-up)
- Metabolic biomarker score per individual: 14 NMR-measured biomarkers (log-transformed, SD-scaled) multiplied by ln(HR) weights from the Estonian Biobank plus sex
- Conventional risk factor score per individual (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer)
- Follow-up windows of interest (5-year and 10-year mortality) and age restriction (above 60 years)

## Outputs

- C-statistics for each risk score, per follow-up window (5-year, 10-year) and stratum (all individuals, >60 years)
- C-statistic differences with SEs and P values (biomarker score minus conventional score)
- Integrated discrimination improvement (IDI) values with SEs and P values
- Results table mirroring Table 4 of the article

## How to apply

Construct the two weighted risk scores first: multiply log-transformed, SD-scaled biomarkers by their ln(hazard ratio) weights (from Estonian Biobank meta-analysis, Supplementary Table 4) and sum, plus sex for the metabolic score; build the conventional score from sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, and cancer, excluding age at sampling since it serves as the Cox time scale. Fit Cox proportional hazards models with age at blood sampling as the time scale for 5-year and 10-year mortality windows, in all individuals and restricted to those above 60, and compute Harrell's C-statistics for each score. Quantify the C-statistic difference between scores for each follow-up/age combination, then compute IDI comparing the two scores using custom-made functions in R. Assemble C-statistics, differences (with SEs and P values), and IDI values (with SEs and P values) into a results table mirroring Table 4. Validate against the reported anchors: biomarker score C = 0.837 (5-year, all) vs conventional 0.772 (difference 0.065), and C = 0.830 (10-year) vs 0.790 (difference 0.040), with IDI of 5.9% and 8.6%.

## Related tools

- **R** (Environment for survival analyses, C-statistic computation, and IDI calculation with custom-made functions)
- **R-package survival** (Fits Cox proportional hazards models with age at blood sampling as the time scale for 5- and 10-year mortality windows)
- **custom-made functions in R** (Compute integrated discrimination improvement (IDI) and C-statistic differences with SEs and P values)
- **STATA/SE 11.2** (Used alongside R for the survival analyses in the individual cohorts)

## Examples

```
coxph(Surv(age_sampling, age_sampling + 5, death) ~ metab_score, data = finrisk1997); idi_custom(scores metab vs conventional, window = 5y)
```

## Evaluation signals

- Conventional risk factor score C-statistic for 5-year mortality in all individuals reproduces to ≈0.772, and the metabolic biomarker score to ≈0.837 (difference 0.065).
- 10-year mortality C-statistics reproduce to ≈0.830 (biomarker) vs ≈0.790 (conventional), a difference of 0.040.
- In the >60 age group, differences are larger: 0.105 for 5-year and 0.065 for 10-year mortality.
- IDI is consistently higher for the biomarker model, with reported values of 5.9% (5-year) and 8.6% (10-year), each accompanied by SE and P value.
- Age at sampling must not appear as a covariate in either score, since it serves as the time scale of the Cox model.

## Limitations

- The number of biomarkers captured by the targeted NMR platform is only a fraction of the metabolites in human serum; more complete platforms were not compared, so predictive accuracy may underestimate what broader metabolomics could achieve.
- The metabolic biomarker score is based on scaled biomarker values created separately for each cohort, so it is not yet suitable for classification of patients in the clinic; a score in concentration units from individual-level data is still needed.
- Score weights were trained only on the Estonian Biobank and tested only in FINRISK 1997; further validation in relevant clinical settings is required before use to guide patient care.
- Use of the identified metabolic profile as a predictor of mortality or surrogate endpoint in clinical studies needs further investigation.

## Evidence

- [discussion] C-statistics and IDI as the two predictive-value measures: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [abstract] Prediction accuracy comparison (abstract): "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] Biomarker score construction from ln(HR) weights: "the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed"
- [discussion] Cox model with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Weights trained on Estonian Biobank to avoid overestimation: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [intro] FINRISK 1997 cohort size and deaths: "In the FINRISK 1997 cohort, consisting of 7603 individuals of whom 1213 died during follow-up, we compare the predictive value of a score based on the identified mortality-associated biomarkers"
- [discussion] Tools used for survival analyses: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
