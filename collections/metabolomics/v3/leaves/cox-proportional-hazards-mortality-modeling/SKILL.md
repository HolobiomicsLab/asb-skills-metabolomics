---
name: cox-proportional-hazards-mortality-modeling
description: Use when relating log-transformed, SD-scaled biomarker measurements (e.g.,
  Nightingale NMR metabolomics) to mortality endpoints in cohort data, and per-cohort
  Cox results must be meta-analyzed into pooled HRs and 95% CIs.
when_to_use_negative:
- When the goal is short-term risk prediction rather than intermediate/long-term (5-10
  year) mortality association, or when a concentration-unit clinical classification
  score is required (the article notes the score is 'not yet suitable for classification
  of patients in the clinic' because biomarkers are scaled per cohort)
- When biomarker platform coverage is the limiting factor — the targeted Nightingale
  NMR platform captures only a fraction of serum metabolites, so broader metabolite
  discovery is out of scope
- When modeling non-time-to-event outcomes that lack a censored survival structure,
  where Cox PH with age-as-time-scale does not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0102
tools:
- name: R
  role: Cohort-level Cox proportional hazards survival analyses with age at blood
    sampling as the time scale
- name: STATA/SE 11.2
  role: Used for the cohort-level survival analyses alongside R
- name: R-package meta
  role: Fixed-effect pooling of per-cohort Cox meta-analysis results and successive-round
    meta-analyses in the stepwise selection procedure
- name: Nightingale Health NMR metabolomics platform
  role: Source of the standardized metabolic biomarker quantifications (EDTA plasma
    and serum) that serve as model inputs
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:46b18b4a23386c4de7255a939dcf83a150359fd61dcd44883f95d8ad3019e746
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  - R-package meta
  - STATA/SE
  - STATA/SE 11.2
  - custom-made functions in R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cox-proportional-hazards-mortality-modeling

## Summary

Fits Cox proportional hazards models with age at blood sampling as the time scale to associate metabolic biomarkers with all-cause and cause-specific mortality, then pools per-cohort results with a fixed-effect meta-analysis in the R-package meta.

## When to use

Use when relating log-transformed, SD-scaled biomarker measurements (e.g., Nightingale NMR metabolomics) to mortality endpoints in cohort data, and per-cohort Cox results must be meta-analyzed into pooled HRs and 95% CIs.

## When NOT to use

- When the goal is short-term risk prediction rather than intermediate/long-term (5-10 year) mortality association, or when a concentration-unit clinical classification score is required (the article notes the score is 'not yet suitable for classification of patients in the clinic' because biomarkers are scaled per cohort)
- When biomarker platform coverage is the limiting factor — the targeted Nightingale NMR platform captures only a fraction of serum metabolites, so broader metabolite discovery is out of scope
- When modeling non-time-to-event outcomes that lack a censored survival structure, where Cox PH with age-as-time-scale does not apply

## Inputs

- Per-cohort biomarker measurement tables (Nightingale NMR metabolomics values, log-transformed and SD-scaled per study)
- Mortality endpoints per cohort (all-cause and cause-of-death: cancer, cardiovascular, nonlocalized infections, other)
- Covariates per cohort (sex, age at blood sampling, study-specific covariates)
- Deposited primary survival meta-analysis summary statistics from the BBMRI-omics atlas (http://bbmri.researchlumc.nl/atlas)

## Outputs

- Per-cohort Cox model hazard ratios, 95% CIs, and P values per biomarker-outcome pair
- Fixed-effect meta-analysis pooled hazard ratios and 95% CIs (e.g., GlycA all-cause HR 1.41, 95% CI 1.26-1.57)
- Age-stratified and sex-stratified association-direction flags per biomarker
- Result tables (e.g., metabolic_biomarker_score_results.csv, stepwise_selected_biomarkers.csv) with 14 biomarker rows

## How to apply

Preprocess each biomarker by adding 1 to zero values (x+1), log-transforming, and scaling to standard deviation units separately per study. In each cohort, fit a Cox proportional hazards model in R (survival package) or STATA/SE 11.2 using age at blood sampling as the time scale, adjusted for sex and study-specific covariates. Pool cohort-level log hazard ratios with a fixed-effect model in the R-package meta. Apply the Bonferroni-adjusted significance threshold of 2.21 x 10^-4 (0.05/226) when screening biomarkers, and extend the same modeling frame to cause-specific endpoints (cancer, cardiovascular, nonlocalized infections) and to age strata (<60, 60-70, 70-80, 80-90, >90 years) and sex to check association-direction consistency.

## Related tools

- **R** (Cohort-level Cox proportional hazards survival analyses with age at blood sampling as the time scale)
- **STATA/SE 11.2** (Used for the cohort-level survival analyses alongside R)
- **R-package meta** (Fixed-effect pooling of per-cohort Cox meta-analysis results and successive-round meta-analyses in the stepwise selection procedure)
- **Nightingale Health NMR metabolomics platform** (Source of the standardized metabolic biomarker quantifications (EDTA plasma and serum) that serve as model inputs)

## Examples

```
Rscript -e 'library(survival); fit <- coxph(Surv(age_at_blood_sampling, death) ~ GlycA + sex, data = cohort); summary(fit)'
```

## Evaluation signals

- Reproduced GlycA all-cause HR = 1.41 (95% CI 1.26-1.57, P 5.75e-10) and PUFA/FA HR = 0.73 (95% CI 0.69-0.78, P < 2.22e-16) match Table 3 values
- Pooled score HR for the 14-biomarker metabolic score equals 2.73 (95% CI 2.60-2.86), matching Table 2-derived ln(HR) weights
- All 14 biomarkers show consistent association directions across the five age strata (<60, 60-70, 70-80, 80-90, >90) and both sexes, as in Supplementary Data 4
- Biomarker significance screening uses the Bonferroni threshold 0.05/226 = 2.21 x 10^-4
- Result CSVs contain 14 biomarker rows whose ln(HR) weights match Table 2 and cross-check against the BBMRI-omics atlas summary statistics

## Limitations

- Associations are observational and biomarker weights are derived from per-cohort scaled values, so the score is not yet suitable for clinical patient classification
- The targeted NMR platform captures only a fraction of metabolites in human serum; more complete platforms were not compared
- Whether similar multivariate direction-of-effect changes occur for other phenotypes was not investigated; GlycA and branched-chain amino acid adjustment flips the sign of XXL-VLDL-L and isoleucine effects

## Evidence

- [methods] Cox model with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [methods] Fixed-effect meta-analysis pooling: "the meta-analyses were performed using a fixed-effect model implemented in the R-package meta"
- [methods] Biomarker preprocessing convention: "a value of one was added to all biomarkers containing zeroes (i.e., x + 1)"
- [methods] Bonferroni significance threshold: "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
- [results] Pooled score hazard ratio: [evidence span withheld — anchor status: not_found]
