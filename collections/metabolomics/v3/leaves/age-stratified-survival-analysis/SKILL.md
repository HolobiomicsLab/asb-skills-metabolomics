---
name: age-stratified-survival-analysis
description: Use when testing whether biomarker–mortality associations hold across
  age groups, especially where conventional risk factors (e.g., systolic blood pressure,
  total cholesterol) show opposite associations in the elderly versus middle age.
when_to_use_negative:
- The analysis goal is discovery of which biomarkers associate with mortality (the
  stepwise forward-backward selection over 63 of 226 biomarkers with Bonferroni threshold
  2.21 × 10^-4), not consistency checking or prediction within age strata.
- The input cohort lacks age at blood sampling or adequate deaths within a stratum
  (e.g., >90 years), making stratified Cox models unstable.
- You need a clinically deployable score in concentration units — the article notes
  the per-cohort SD-scaled score 'is not yet suitable for classification of patients
  in the clinic'.
tools:
- name: R
  role: Environment for fitting Cox proportional hazards models (age at blood sampling
    as time scale) per age/sex stratum and computing C-statistics/IDI
- name: R-package meta
  role: Fixed-effect pooling of biomarker–mortality estimates underlying the score
    weights used in stratified analyses
- name: R-package survival
  role: Cox regression implementation for the survival models in FINRISK 1997
- name: STATA/SE 11.2
  role: Alternative environment used for the individual-cohort survival analyses
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:d99027f179fe38569416f28657a1b1172d6537a80669c3deb315eec9cd142885
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  - R-package survival
  - STATA/SE 11.2
  - custom-made functions in R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# Age-stratified survival analysis

## Summary

Fits Cox proportional hazards models with age at blood sampling as the time scale to estimate biomarker–mortality associations, then re-estimates them within age strata (<60, 60–70, 70–80, 80–90, >90 years) and sex to verify direction-of-effect consistency, as done for the 14 metabolic biomarkers in this study.

## When to use

Use when testing whether biomarker–mortality associations hold across age groups, especially where conventional risk factors (e.g., systolic blood pressure, total cholesterol) show opposite associations in the elderly versus middle age.

## When NOT to use

- The analysis goal is discovery of which biomarkers associate with mortality (the stepwise forward-backward selection over 63 of 226 biomarkers with Bonferroni threshold 2.21 × 10^-4), not consistency checking or prediction within age strata.
- The input cohort lacks age at blood sampling or adequate deaths within a stratum (e.g., >90 years), making stratified Cox models unstable.
- You need a clinically deployable score in concentration units — the article notes the per-cohort SD-scaled score 'is not yet suitable for classification of patients in the clinic'.

## Inputs

- Cohort-level individual data with biomarker values (log-transformed, SD-scaled per cohort), sex, and mortality follow-up (e.g., FINRISK 1997: 7603 individuals, 1213 deaths)
- Per-biomarker fixed-effect meta-analysis HR estimates and 95% CIs (e.g., from Table 2 / BBMRI-omics atlas summary statistics)
- Age at blood sampling (used as the Cox time scale) and predefined strata boundaries (<60, 60–70, 70–80, 80–90, >90 years)

## Outputs

- Per-stratum hazard ratios and 95% CIs for each of the 14 biomarkers, with direction-of-effect consistency flags across the five age strata and both sexes
- Overall and age-stratified C-statistics and C-statistic differences (with SEs and P values) for 5- and 10-year mortality windows
- IDI values comparing metabolic biomarker score vs conventional risk factor score (with SEs and P values)
- Results table (e.g., metabolic_biomarker_score_results.csv) mirroring Table 4 / Supplementary Data 4

## How to apply

Fit a Cox proportional hazards model with age at blood sampling as the time scale on the full cohort (e.g., 44,168 individuals across 12 cohorts, using the Nightingale NMR biomarkers preprocessed as x+1 for zeroes, log-transformed, and scaled to SD units per study). Then divide samples into age groups of <60, 60–70, 70–80, 80–90, and >90 years and repeat the association models per stratum, plus separate models for men and women. Compare hazard-ratio directions across strata against Supplementary Data 4 to flag any biomarker whose direction reverses. In R, pool stratum estimates with the fixed-effect meta package where needed. For prediction contexts, additionally compute C-statistics for 5- and 10-year mortality windows in all individuals and restricted to those above 60, where the biomarker score advantage grows (ΔC = 0.105 for 5-year, 0.065 for 10-year in the >60 group).

## Related tools

- **R** (Environment for fitting Cox proportional hazards models (age at blood sampling as time scale) per age/sex stratum and computing C-statistics/IDI)
- **R-package meta** (Fixed-effect pooling of biomarker–mortality estimates underlying the score weights used in stratified analyses)
- **R-package survival** (Cox regression implementation for the survival models in FINRISK 1997)
- **STATA/SE 11.2** (Alternative environment used for the individual-cohort survival analyses)

## Examples

```
Rscript -e 'library(survival); fit <- coxph(Surv(age_sampling, age_event, death) ~ biomarker_score + sex, data=finrisk97); coxph(..., subset=age_sampling>=60)'
```

## Evaluation signals

- All 14 biomarkers show consistent association directions across all five age strata (<60, 60–70, 70–80, 80–90, >90) and both sexes, matching Supplementary Data 4.
- In FINRISK 1997, reproduced C-statistics approach the reported values: metabolic score 0.837 (5-year, all) and 0.830 (10-year, all) vs conventional score 0.772 and 0.790.
- In the >60 age group, score differences are larger than overall (ΔC = 0.105 for 5-year, 0.065 for 10-year).
- The metabolite score's per-unit mortality association is HR = 2.73 (95% CI 2.60–2.86), and IDI is consistently higher for the biomarker model (5.9% and 8.6%).
- Output CSV contains 14 biomarker rows with ln(HR) weights matching Table 2 before stratified modeling.

## Limitations

- The NMR platform captures only a fraction of metabolites in human serum; more complete high-throughput platforms were not compared for predictive accuracy.
- Score weights were trained only on the Estonian Biobank and tested only in FINRISK 1997, limiting generalizability of stratified predictions.
- The metabolic biomarker score is based on SD-scaled biomarker values created separately per cohort, so it is not yet suitable for clinical patient classification.
- Whether the identified biomarkers reflect generic immune-metabolic health adversity and by what mechanisms remains undetermined.

## Evidence

- [discussion] Cox model with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Age strata definition: "samples were divided into age groups of <60, 60–70, 70–80, 80–90, and >90 years"
- [abstract] Consistency across strata: "Overall, these associations are similar in men and women and across different age strata."
- [abstract] Prediction advantage at higher ages: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] Biomarker preprocessing convention: [evidence span withheld — anchor status: not_found]
