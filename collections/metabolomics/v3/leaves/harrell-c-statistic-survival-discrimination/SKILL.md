---
name: harrell-c-statistic-survival-discrimination
description: Use when comparing the predictive ability of two or more risk scores
  for a censored time-to-event endpoint (e.g. 5- and 10-year all-cause mortality)
  and rank discrimination (C-statistic) is the required performance measure.
when_to_use_negative:
- When calibration or absolute risk improvement is needed rather than rank discrimination
  — the article pairs C-statistics with IDI for that purpose
- When the outcome is a binary/cross-sectional label without censoring or follow-up
  time (no survival structure for Harrell's C)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Implements the custom-made discrimination functions that compute Harrell C-statistics
    at 5- and 10-year horizons; also used for the underlying Cox proportional hazards
    analyses
- name: STATA/SE 11.2
  role: Used for the cohort survival analyses whose fitted model coefficients provide
    the score weights evaluated by the C-statistic
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:ea8bfdf06c18b9d98c52e2aed9aa4002136937bef2a32a14853256708c025e1c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - STATA/SE 11.2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# harrell-c-statistic-survival-discrimination

## Summary

Computes Harrell's C-statistic to quantify rank discrimination of weighted risk scores against censored mortality follow-up at 5- and 10-year horizons, as used to show the 14-biomarker metabolic score (C = 0.837/0.830) outperforms a conventional risk factor score (C = 0.772/0.790) in FINRISK 1997.

## When to use

Use when comparing the predictive ability of two or more risk scores for a censored time-to-event endpoint (e.g. 5- and 10-year all-cause mortality) and rank discrimination (C-statistic) is the required performance measure.

## When NOT to use

- When calibration or absolute risk improvement is needed rather than rank discrimination — the article pairs C-statistics with IDI for that purpose
- When the outcome is a binary/cross-sectional label without censoring or follow-up time (no survival structure for Harrell's C)

## Inputs

- Validation cohort individual-level survival data with follow-up time, event indicator, and age (FINRISK 1997: 7603 individuals, 1213 deaths)
- Weighted risk score per individual (linear predictor from Cox-derived weights, e.g. Estonian Biobank Supplementary Table 4)
- Evaluation horizons (5-year and 10-year mortality) and optional age restriction (>60 years)

## Outputs

- Harrell C-statistics per score per horizon for all ages and for individuals above 60 years (reference values: 0.772 and 0.790 all ages; 0.626 and 0.650 for >60 for the conventional score)
- Results table, e.g. conventional_risk_factor_score_results.csv

## How to apply

First derive score weights externally to the validation cohort: fit a Cox proportional hazards model with age at blood sampling as the time scale on the Estonian Biobank cohort (excluding age from the score itself) and extract the coefficients (Supplementary Table 4 weighting scheme) to avoid overestimation. Apply each weighted linear predictor (e.g. the conventional score of sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking status, alcohol consumption, and prevalent diabetes; or the metabolic biomarker score with ln(hazard ratio) weights) to the validation cohort, FINRISK 1997 (7603 individuals, 1213 deaths). Compute Harrell C-statistics for 5-year and 10-year mortality horizons using the custom-made discrimination functions in R described in the Methods, both for all ages and restricted to individuals above 60 years. Interpret a higher C as better rank discrimination, and compare against the reported reference values (conventional score 0.772/0.790 all ages and 0.626/0.650 for >60; biomarker score + sex 0.837/0.830).

## Related tools

- **R** (Implements the custom-made discrimination functions that compute Harrell C-statistics at 5- and 10-year horizons; also used for the underlying Cox proportional hazards analyses)
- **STATA/SE 11.2** (Used for the cohort survival analyses whose fitted model coefficients provide the score weights evaluated by the C-statistic)

## Examples

```
Rscript c_statistics.R --input finrisk1997.csv --score biomarker_score,conventional_score --horizons 5,10 --min-age 0 --out conventional_risk_factor_score_results.csv
```

## Evaluation signals

- Biomarker score + sex must yield C = 0.837 (5-year) and C = 0.830 (10-year) in FINRISK 1997 all ages
- Conventional risk factor score must yield C = 0.772/0.790 all ages and 0.626/0.650 restricted to >60 years
- C-statistic for the metabolic biomarker score should exceed that of the conventional risk factor score at both horizons
- Both horizons (5-year and 10-year) and both age strata (all ages and >60) must be reported, not just one
- Output CSV (e.g. conventional_risk_factor_score_results.csv) contains the four C-statistics assembled as a results table

## Limitations

- Score weights were trained only on the Estonian Biobank cohort and predictive ability tested only in FINRISK 1997, so generalization to other populations is untested
- The metabolic biomarker score is based on scaled biomarker values created separately per cohort, making it not yet suitable for clinical patient classification
- No code repository is mentioned for the custom-made R discrimination functions, so exact implementation must be reproduced from the Methods description

## Evidence

- [discussion] C-statistics used as predictive measure in FINRISK 1997: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [abstract] Reference C-statistic values for biomarker model: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] Software for survival analyses: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
