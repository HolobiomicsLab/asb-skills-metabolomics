---
name: multivariate-adjustment-direction-of-effect-analysis
description: Use when biomarker effect estimates may change sign after multivariate
  adjustment for correlated immune-metabolic covariates, and you must reproduce or
  verify direction-of-effect changes (e.g., sign flips for XXL-VLDL-L or isoleucine)
  alongside partial-correlation structure.
when_to_use_negative:
- When the analysis goal is the biomarker-only vs. combined-model C-statistic comparison
  (C-statistic = 0.837/0.830 biomarker+sex model) rather than direction-of-effect
  stability under adjustment
- When biomarkers of interest show no partial correlation with candidate adjustment
  covariates — no sign-flip mechanism exists to detect
- When the outcome is not a time-to-event endpoint amenable to Cox models with age
  as the time scale
tools:
- name: R
  role: Fits Cox proportional hazards models (with age at blood sampling as time scale)
    for unadjusted vs. adjusted biomarker associations
- name: STATA/SE 11.2
  role: Used for cohort-level survival analyses underlying the association estimates
    compared across adjustment sets
- name: R-package meta
  role: Fixed-effect meta-analysis across cohorts used in the stepwise forward-backward
    selection that produces the adjustment structure
- name: Nightingale Health NMR metabolomics platform
  role: Quantifies the EDTA plasma/serum metabolic biomarkers (e.g., XXL-VLDL-L, GlycA,
    amino acids) whose adjusted effect directions are compared
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:b07c32fd0e493ac38d3f12583327291e21570910c5260247c7cd4b9aaac9732a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  - R-package meta
  - STATA/SE
  - STATA/SE 11.2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# multivariate-adjustment-direction-of-effect-analysis

## Summary

Fits multivariable-adjusted Cox proportional hazards models on Nightingale NMR metabolic biomarkers and compares adjusted effect signs (ln(HR)) against unadjusted associations to expose direction-of-effect flips driven by correlated covariates such as GlycA and the branched-chain amino acids leucine and valine.

## When to use

Use when biomarker effect estimates may change sign after multivariate adjustment for correlated immune-metabolic covariates, and you must reproduce or verify direction-of-effect changes (e.g., sign flips for XXL-VLDL-L or isoleucine) alongside partial-correlation structure.

## When NOT to use

- When the analysis goal is the biomarker-only vs. combined-model C-statistic comparison (C-statistic = 0.837/0.830 biomarker+sex model) rather than direction-of-effect stability under adjustment
- When biomarkers of interest show no partial correlation with candidate adjustment covariates — no sign-flip mechanism exists to detect
- When the outcome is not a time-to-event endpoint amenable to Cox models with age as the time scale

## Inputs

- Per-cohort log-transformed, SD-scaled biomarker measurements from the Nightingale Health NMR metabolomics platform
- Conventional risk factor measurements (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer)
- Risk-score weights / ln(hazard ratio) values from the Estonian Biobank cohort (Supplementary Table 4)
- Follow-up time and all-cause mortality status with age at blood sampling as the time scale

## Outputs

- Table of unadjusted vs. fully adjusted effect signs (ln(HR)) per biomarker, flagging sign flips (e.g., XXL-VLDL-L and isoleucine)
- Partial correlation matrix between the 14 biomarkers and conventional risk factors (Supplementary Fig. 18 structure)
- Adjusted Cox model coefficient estimates with hazard ratios per adjustment set

## How to apply

Start from per-cohort log-transformed, SD-scaled biomarker measurements (adding 1 to biomarkers containing zeroes, i.e. x+1, before log transformation). Fit Cox proportional hazards models with age at blood sampling as the time scale: first an unadjusted (or sex-only) model per biomarker, then a fully adjusted model adding the relevant correlated covariates — here GlycA and the branched-chain amino acids leucine and valine. Compute partial correlations between the biomarkers of interest and the adjustment variables (reproducing the Supplementary Fig. 18 correlation structure) to justify which covariates drive shared variance. Compare the sign of each adjusted ln(hazard ratio) against its unadjusted counterpart and flag sign flips (e.g., total lipids in chylomicrons and extremely large VLDL, XXL-VLDL-L, and isoleucine). Rationale: partially correlated biomarkers can absorb or reverse each other's marginal associations, so sign stability under adjustment is the diagnostic for whether an effect is independent.

## Related tools

- **R** (Fits Cox proportional hazards models (with age at blood sampling as time scale) for unadjusted vs. adjusted biomarker associations)
- **STATA/SE 11.2** (Used for cohort-level survival analyses underlying the association estimates compared across adjustment sets)
- **R-package meta** (Fixed-effect meta-analysis across cohorts used in the stepwise forward-backward selection that produces the adjustment structure)
- **Nightingale Health NMR metabolomics platform** (Quantifies the EDTA plasma/serum metabolic biomarkers (e.g., XXL-VLDL-L, GlycA, amino acids) whose adjusted effect directions are compared)

## Examples

```
coxph(Surv(age_at_sampling, mortality) ~ XXL_VLDL_L + isoleucine + sex, data=scaled); coxph(Surv(age_at_sampling, mortality) ~ XXL_VLDL_L + isoleucine + sex + GlycA + leucine + valine, data=scaled)
```

## Evaluation signals

- The adjusted effects of total lipids in chylomicrons and extremely large VLDL (XXL-VLDL-L) and isoleucine must show sign flips relative to their unadjusted associations when GlycA and leucine/valine are included
- The computed partial correlations must reproduce the correlation structure reported in Supplementary Fig. 18
- Biomarker preprocessing must match: x+1 for zero-containing biomarkers, then log transformation and SD scaling separately per study
- Cox models must use age at blood sampling as the time scale, not calendar time
- Adjusted coefficient signs and ln(HR) values should be traceable to the meta-analysis weights in Supplementary Table 4 / Table 2

## Limitations

- Whether similar multivariate direction-of-effect changes occur for other phenotypes was not investigated in this study
- The targeted NMR platform captures only a fraction of metabolites in human serum, so the adjustment structure is limited to the measured biomarker panel
- Risk-score weights were trained only on the Estonian Biobank cohort and predictive ability tested only in FINRISK 1997, limiting generalizability of the adjustment findings

## Evidence

- [methods] Adjusted sign flips are the validation target: [evidence span withheld — anchor status: not_found]
- [discussion] Stepwise meta-analysis selection defines adjustment structure: "we performed a stepwise (forward-backward) procedure based on successive rounds of meta-analyses"
- [discussion] Cox model specification with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Partial correlation structure is the motivating evidence: "It would be interesting to see if a similar effect is also observed for other phenotypes using multivariate adjusted models"
- [supplementary] Correlation structure between biomarkers and conventional factors: [evidence span withheld — anchor status: not_found]
