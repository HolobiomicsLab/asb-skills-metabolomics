---
name: c-statistic-model-comparison-survival
description: Use when two nested mortality-prediction models must be compared in an
  independent validation cohort to decide whether adding predictors (e.g., conventional
  risk factors to 14 metabolic biomarkers) improves 5- or 10-year risk prediction.
when_to_use_negative:
- The goal is selecting which biomarkers independently associate with mortality —
  use the stepwise forward-backward meta-analysis procedure with the Bonferroni threshold
  of 2.21 × 10^-4 (0.05/226) instead
- Model weights would be estimated and evaluated on the same cohort, overestimating
  predictive ability — weights must come from a separate training cohort (here, Estonian
  Biobank)
- The outcome is not time-to-event mortality or no validation cohort independent of
  the weight-estimation cohort is available
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Used with custom-made functions to fit Cox proportional hazards models and
    compute C-statistics and IDI for the risk-score comparison
- name: STATA/SE 11.2
  role: Used for the cohort-level survival analyses feeding the model comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:b05e9df2665e8e600fb3ea888a8c89decba203525f21f4f385f56fc7cfef813f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STATA/SE
  - STATA/SE 11.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# c-statistic-model-comparison-survival

## Summary

Compares the predictive accuracy of two nested survival models (e.g., a metabolic biomarker score vs. a combined score with conventional risk factors) using Harrell's C-statistics for 5- and 10-year mortality plus integrated discrimination improvement (IDI), evaluated in a held-out validation cohort.

## When to use

Use when two nested mortality-prediction models must be compared in an independent validation cohort to decide whether adding predictors (e.g., conventional risk factors to 14 metabolic biomarkers) improves 5- or 10-year risk prediction.

## When NOT to use

- The goal is selecting which biomarkers independently associate with mortality — use the stepwise forward-backward meta-analysis procedure with the Bonferroni threshold of 2.21 × 10^-4 (0.05/226) instead
- Model weights would be estimated and evaluated on the same cohort, overestimating predictive ability — weights must come from a separate training cohort (here, Estonian Biobank)
- The outcome is not time-to-event mortality or no validation cohort independent of the weight-estimation cohort is available

## Inputs

- Per-cohort log-transformed, SD-scaled Nightingale NMR biomarker measurements (14 mortality-associated biomarkers)
- Conventional risk factor measurements (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer)
- Risk-score weights as ln(hazard ratio) values estimated on the Estonian Biobank cohort (Supplementary Table 4)
- Survival data (age at blood sampling as time scale, mortality events) from the FINRISK 1997 validation cohort

## Outputs

- C-statistics for 5- and 10-year mortality per model (e.g., biomarker-plus-sex: 0.837 and 0.830; conventional risk factors: 0.772 and 0.790)
- Integrated discrimination improvement (IDI) comparing the nested models
- Decision on whether the combined model improves over the biomarker-only model

## How to apply

First construct weighted risk scores by multiplying per-cohort log-transformed, SD-scaled biomarker values by their ln(hazard ratio) weights (from Table 2 / Supplementary Table 4, estimated on the Estonian Biobank cohort to avoid overestimation) and summing them. Fit Cox proportional hazards models with age at blood sampling as the time scale — one for the biomarker-only score (14 biomarkers plus sex) and one for the combined model that adds conventional risk factors (sex, BMI, systolic blood pressure, smoking, alcohol, prevalent diabetes, CVD, cancer), excluding total cholesterol, HDL, triglycerides, and creatinine because the Nightingale NMR platform already captures them. Evaluate both scores in the independent FINRISK 1997 cohort using (1) C-statistics for 5- and 10-year mortality and (2) IDI, implemented with R and custom-made functions (STATA/SE 11.2 for cohort-level survival analyses). The comparison is decisive when the combined model yields no higher C-statistic than the biomarker-only model, demonstrating the conventional factors add no predictive value.

## Related tools

- **R** (Used with custom-made functions to fit Cox proportional hazards models and compute C-statistics and IDI for the risk-score comparison)
- **STATA/SE 11.2** (Used for the cohort-level survival analyses feeding the model comparison)

## Examples

```
fit <- coxph(Surv(age_sample, death) ~ biomarker_score, data = FINRISK1997); c_stat <- survival::concordance(fit)$concordance
```

## Evaluation signals

- The combined model (biomarkers + conventional risk factors) must NOT yield a higher C-statistic than the biomarker-only model — the paper reports no improvement when conventional risk factors are added
- The biomarker-plus-sex score must reach C-statistic = 0.837 (5-year) and 0.830 (10-year) in FINRISK 1997, exceeding the conventional risk factor model (0.772 and 0.790)
- IDI must be reported alongside the C-statistic as the second predictive-value measure
- Score weights must equal ln(hazard ratio) values from Table 2 / Supplementary Table 4 applied to log-transformed, SD-scaled biomarkers
- Score estimation (Estonian Biobank) and evaluation (FINRISK 1997) must occur in disjoint cohorts

## Limitations

- Risk-score weights were trained only on the Estonian Biobank cohort and tested only in FINRISK 1997, so generalization to other populations is untested
- The metabolic biomarker score is based on per-cohort scaled biomarker values and is not yet suitable for clinical patient classification; a concentration-unit score from individual-level data is still needed
- The targeted NMR platform captures only a fraction of metabolites in human serum, so predictive accuracy from more complete metabolic platforms is untested

## Evidence

- [discussion] C-statistic/IDI evaluation in validation cohort: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [abstract] Benchmark C-statistics for biomarker-plus-sex model: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively)"
- [discussion] Independent weight training to avoid overestimation: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
- [methods] Cox model specification with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
