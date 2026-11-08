---
name: partial-correlation-analysis-of-biomarkers
description: Use when testing whether a metabolic biomarker panel adds predictive
  value beyond conventional risk factors, or when the question is whether risk-factor
  information is already captured by correlated metabolites (e.g. lipids measured
  by both clinical chemistry and NMR).
when_to_use_negative:
- When the goal is a clinically deployable classification score in concentration units
  — the article explicitly states the constructed metabolic biomarker score 'is not
  yet suitable for classification of patients in the clinic, since it is based on
  scaled biomarker values created separately for each cohort'
- When adding lipid-based conventional risk factors (total cholesterol, HDL, triglycerides)
  as if they were independent information — these are already captured by the Nightingale
  NMR platform and must be excluded from the combined score
- When the prediction target is a disease-specific endpoint rather than 5–10 year
  all-cause mortality, since biomarker weights and score design are mortality-specific
edam_operation: http://edamontology.org/operation_3659
tools:
- name: R
  role: Used for partial correlation computation, Cox proportional hazards model fitting
    (with age at blood sampling as time scale), C-statistic/IDI evaluation and custom-made
    comparison functions
- name: STATA/SE 11.2
  role: Used for cohort-level survival analyses underlying the risk-score comparison
- name: R-package meta
  role: Implements the fixed-effect meta-analysis framework from which the ln(hazard
    ratio) score weights are derived
- name: Nightingale Health NMR metabolomics platform
  role: Provides the 14 metabolic biomarkers and quantifies lipid fractions (total
    lipids in chylomicrons and extremely large VLDL) that overlap with conventional
    lipid risk factors
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:2e1882edc3dfb2c108598a6375ba7b98931c8236a95d37970076bab1c0d647e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  - R-package meta
  - STATA/SE
  - STATA/SE 11.2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# partial-correlation-analysis-of-biomarkers

## Summary

Quantifies the partial correlation structure between Nightingale NMR metabolic biomarkers and conventional risk factors, and tests whether adding conventional risk factors (sex, BMI, systolic blood pressure, cholesterol fractions, smoking, etc.) to a biomarker-only Cox risk score further improves 5- and 10-year all-cause mortality prediction.

## When to use

Use when testing whether a metabolic biomarker panel adds predictive value beyond conventional risk factors, or when the question is whether risk-factor information is already captured by correlated metabolites (e.g. lipids measured by both clinical chemistry and NMR).

## When NOT to use

- When the goal is a clinically deployable classification score in concentration units — the article explicitly states the constructed metabolic biomarker score 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'
- When adding lipid-based conventional risk factors (total cholesterol, HDL, triglycerides) as if they were independent information — these are already captured by the Nightingale NMR platform and must be excluded from the combined score
- When the prediction target is a disease-specific endpoint rather than 5–10 year all-cause mortality, since biomarker weights and score design are mortality-specific

## Inputs

- Per-cohort log-transformed, SD-scaled Nightingale NMR biomarker measurements
- Conventional risk factor measurements (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer)
- Risk-score weights estimated in the Estonian Biobank cohort (Supplementary Table 4)
- ln(hazard ratio) weights for the 14 biomarkers plus sex from Table 2 meta-analysis results

## Outputs

- Partial correlation matrix between the 14 biomarkers and conventional risk factors (reproducing Supplementary Fig. 18)
- C-statistics for 5- and 10-year mortality for the biomarker-plus-sex model and the combined (biomarker + conventional risk factor) model
- Integrated discrimination improvement (IDI) comparing the combined vs biomarker-only score
- Adjusted hazard ratio signs for total lipids in chylomicrons and extremely large VLDL (XXL-VLDL-L) and isoleucine from the fully adjusted model, documenting direction-of-effect changes

## How to apply

Load the per-cohort log-transformed, SD-scaled biomarker measurements (values with zeroes incremented by x+1 before log-transformation and scaling separately per study) alongside the conventional risk factor measurements (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer). Compute partial correlations between the 14 mortality-associated biomarkers and the conventional risk factors to reproduce the correlation structure reported in Supplementary Fig. 18. Fit Cox proportional hazards models with age at blood sampling as the time scale: a biomarker-only score (14 biomarkers plus sex, weighted by ln(hazard ratio) values from Table 2) and a combined score that additionally includes conventional risk factors, excluding total cholesterol, HDL cholesterol, triglycerides and creatinine because the Nightingale NMR platform already captures these lipid traits. Evaluate both scores in the FINRISK 1997 validation cohort using C-statistics for 5- and 10-year mortality and integrated discrimination improvement (IDI), implemented with the R-package meta and custom-made functions in R. The key decision point is that the combined model must NOT yield a higher C-statistic than the biomarker-only model.

## Related tools

- **R** (Used for partial correlation computation, Cox proportional hazards model fitting (with age at blood sampling as time scale), C-statistic/IDI evaluation and custom-made comparison functions)
- **STATA/SE 11.2** (Used for cohort-level survival analyses underlying the risk-score comparison)
- **R-package meta** (Implements the fixed-effect meta-analysis framework from which the ln(hazard ratio) score weights are derived)
- **Nightingale Health NMR metabolomics platform** (Provides the 14 metabolic biomarkers and quantifies lipid fractions (total lipids in chylomicrons and extremely large VLDL) that overlap with conventional lipid risk factors)

## Evaluation signals

- The combined (biomarker + conventional risk factor) model must not yield a higher C-statistic than the biomarker-plus-sex model, which reached C = 0.837 (5-year) and 0.830 (10-year)
- The conventional-risk-factor-only reference model should land near C = 0.772 (5-year) and 0.790 (10-year) for a valid comparison
- The computed partial correlations must reproduce the correlation structure reported in Supplementary Fig. 18 (14 biomarkers vs conventional risk factors)
- In the fully adjusted model, the effect directions of XXL-VLDL-L and isoleucine must flip sign relative to their unadjusted associations after adjusting for GlycA and the branched-chain amino acids leucine and valine
- Candidate biomarkers entering the model must have passed the Bonferroni-adjusted threshold of 2.21 × 10^-4 (0.05/226), with selection based on a 63-biomarker subset to prevent overfitting

## Limitations

- The metabolic biomarker score is based on per-cohort scaled biomarker values and is not yet suitable for clinical patient classification; a score on concentration units from individual-level data is still needed
- Risk score weights were trained only on the Estonian Biobank cohort and tested only in the FINRISK 1997 cohort, limiting generalizability of the prediction comparison
- The targeted NMR platform captures only a fraction of the metabolites in human serum, so the biomarker-vs-conventional comparison does not cover more complete metabolomics platforms
- Sign flips for XXL-VLDL-L and isoleucine upon adjustment for GlycA and branched-chain amino acids (leucine, valine) mean interpretation of adjusted effects requires care

## Evidence

- [results] Combined model does not improve prediction: [evidence span withheld — anchor status: not_found]
- [abstract] Biomarker score outperforms conventional model: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] Score not clinically classifiable: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
- [discussion] Score weights trained in Estonian Biobank: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
