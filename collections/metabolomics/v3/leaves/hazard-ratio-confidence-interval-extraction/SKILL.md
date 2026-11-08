---
name: hazard-ratio-confidence-interval-extraction
description: 'Use when reproducing or extending the biomarker-outcome association
  table (e.g. Table 3 of the Deelen et al. metabolic mortality profile): each of 14
  NMR biomarkers (e.g. GlycA, PUFA/FA) must yield an HR, 95% CI, and P per cause-of-death
  endpoint.'
when_to_use_negative:
- The goal is cross-cohort meta-analysis with the fixed-effect R-package meta or stepwise
  forward-backward biomarker selection, not per-cohort association extraction
- Input lacks time-to-event structure (age at sampling, death/censoring ages), so
  Cox models with age as time scale cannot be fitted
- Biomarker values are in concentration units for clinical score construction — the
  article notes the score built from per-cohort scaled values is not suitable for
  clinical patient classification
tools:
- name: R
  role: Fits Cox proportional hazards models (survival package) and assembles the
    HR/CI/P table
- name: R-package meta
  role: Fixed-effect meta-analysis context from which the 14 independently associated
    biomarkers and their ln(HR) weights were derived
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:76d0e9531f0766b5642190caa6b08934f01d9fe7de7eae774c1bb8f23ef6f1eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# hazard-ratio-confidence-interval-extraction

## Summary

Extracts per-biomarker hazard ratios, 95% confidence intervals, and P values from Cox proportional hazards models fitted on cohort survival data, producing a Table 3-style biomarker-by-outcome matrix for the 14 mortality-associated NMR metabolic biomarkers.

## When to use

Use when reproducing or extending the biomarker-outcome association table (e.g. Table 3 of the Deelen et al. metabolic mortality profile): each of 14 NMR biomarkers (e.g. GlycA, PUFA/FA) must yield an HR, 95% CI, and P per cause-of-death endpoint.

## When NOT to use

- The goal is cross-cohort meta-analysis with the fixed-effect R-package meta or stepwise forward-backward biomarker selection, not per-cohort association extraction
- Input lacks time-to-event structure (age at sampling, death/censoring ages), so Cox models with age as time scale cannot be fitted
- Biomarker values are in concentration units for clinical score construction — the article notes the score built from per-cohort scaled values is not suitable for clinical patient classification

## Inputs

- FINRISK 1997 cohort individual-level data (N=7603, 1213 deaths) with 14 Nightingale NMR metabolic biomarkers and cause-of-death endpoints
- Time-to-event variables: age at blood sampling (time scale) and age at death/censoring
- Cause-of-death classification (all-cause, cancer, cardiovascular, nonlocalized infections, other)

## Outputs

- Table of hazard ratios, 95% confidence intervals, and P values for each of the 14 biomarkers × 5 mortality outcomes (Table 3 layout)
- Per-model Cox fit objects (coxph summaries) from the R survival package

## How to apply

Load the FINRISK 1997 cohort (7603 individuals, 1213 deaths) with the 14 biomarkers (XXL-VLDL-L, S-HDL-L, VLDL-D, PUFA/FA, Glc, Lac, His, Ile, Leu, Val, Phe, AcAce, Alb, GlycA) and cause-of-death endpoints in R. Preprocess per the article's methods: add 1 to biomarkers containing zeroes (x+1), log-transform, and scale to standard deviation units separately per study. For each biomarker-outcome pair (all-cause, cancer, cardiovascular, nonlocalized infections, other), fit a Cox proportional hazards model with age at blood sampling as the time scale using the survival package, and collect the hazard ratio, 95% CI, and P value. Assemble the estimates into a biomarker-rows × cause-of-death-columns table matching the layout of Table 3. Validate reproduced estimates against reported anchors such as GlycA all-cause HR 1.41 (95% CI 1.26–1.57, P 5.75e-10) and PUFA/FA all-cause HR 0.73 (95% CI 0.69–0.78).

## Related tools

- **R** (Fits Cox proportional hazards models (survival package) and assembles the HR/CI/P table)
- **R-package meta** (Fixed-effect meta-analysis context from which the 14 independently associated biomarkers and their ln(HR) weights were derived)

## Examples

```
library(survival); fit <- coxph(Surv(age_blood, age_event, death) ~ scale(log(GlycA + 1)), data = finrisk97); summary(fit)$conf.int
```

## Evaluation signals

- Reproduced GlycA all-cause estimate matches HR 1.41, 95% CI 1.26–1.57, P 5.75e-10
- Reproduced PUFA/FA all-cause estimate matches HR 0.73, 95% CI 0.69–0.78, P < 2.22e-16
- Every reported 95% CI contains its point HR estimate and the table has 14 biomarker rows × 5 outcome columns
- Biomarker-outcome pairs flagged significant have P below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)
- Direction of effect for most biomarkers is consistent across cause-specific outcomes (cardiovascular, cancer, nonlocalized infections) and with all-cause mortality

## Limitations

- The targeted NMR platform captures only a fraction of metabolites in human serum, so the 14-biomarker set is platform-limited
- Biomarkers are log-transformed and scaled per cohort (x+1, log, SD units), so raw HRs are per-SD and not directly comparable across cohorts or usable clinically
- Associations are observational; the metabolic profile as a predictor or surrogate endpoint needs further investigation before clinical use

## Evidence

- [discussion] Cox model time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] biomarker preprocessing: [evidence span withheld — anchor status: not_found]
- [discussion] analysis software: "The survival analyses in the individual cohorts were performed using R and STATA/SE 11.2"
- [discussion] significance threshold: "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
