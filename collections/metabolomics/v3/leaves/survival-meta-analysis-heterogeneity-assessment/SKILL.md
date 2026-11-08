---
name: survival-meta-analysis-heterogeneity-assessment
description: Use when you have per-cohort Cox model summary statistics (lnHR and SE)
  for many candidate biomarkers and must pool them across cohorts while quantifying
  between-cohort heterogeneity (I2, P(het)) and selecting an independent biomarker
  set.
when_to_use_negative:
- You only have a single cohort — no cross-cohort pooling or I2/P(het) assessment
  is possible
- You need an individual-level pooled analysis or a random-effects model; this skill
  is specific to successive rounds of fixed-effect meta-analysis on summary statistics
- Your goal is a clinically usable risk score in concentration units — the article
  states the scaled-biomarker score 'is not yet suitable for classification of patients
  in the clinic'
edam_operation: http://edamontology.org/operation_3799
tools:
- name: R
  role: Environment for fitting per-cohort Cox proportional hazards models with age
    at blood sampling as time scale and for running the stepwise selection
- name: R-package meta
  role: Implements the fixed-effect model used to combine cohort-level survival estimates
    and derive I2/P(het)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:769aa149318b544320fa813519fac42fb33c38e58fc6de3627061efa10327c6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R-package meta
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# survival-meta-analysis-heterogeneity-assessment

## Summary

Combines cohort-level Cox proportional hazards estimates for circulating biomarkers into a fixed-effect survival meta-analysis (R-package meta) and reports HR, 95% CI, P, I2 and P(het) per biomarker, then feeds the pooled P values into a Bonferroni-controlled stepwise (forward-backward) selection of independently mortality-associated biomarkers.

## When to use

Use when you have per-cohort Cox model summary statistics (lnHR and SE) for many candidate biomarkers and must pool them across cohorts while quantifying between-cohort heterogeneity (I2, P(het)) and selecting an independent biomarker set.

## When NOT to use

- You only have a single cohort — no cross-cohort pooling or I2/P(het) assessment is possible
- You need an individual-level pooled analysis or a random-effects model; this skill is specific to successive rounds of fixed-effect meta-analysis on summary statistics
- Your goal is a clinically usable risk score in concentration units — the article states the scaled-biomarker score 'is not yet suitable for classification of patients in the clinic'

## Inputs

- Cohort-level Cox proportional hazards summary statistics for 226 NMR metabolic biomarkers (BBMRI-omics atlas export)
- Per-cohort biomarker values preprocessed as x+1, log-transformed, and scaled to standard deviation units separately per study
- Cohort metadata: 12 cohorts, 44,168 individuals, sex and study-specific demographic/relatedness covariates

## Outputs

- table2_biomarker_meta_analysis.tsv with per-biomarker HR, 95% CI, P value, I2 heterogeneity statistic, and P(het) for the 14 retained biomarkers
- Fixed-effect meta-analysis results per round of the stepwise forward-backward selection procedure
- Final set of 14 independently associated biomarkers with direction of effect

## How to apply

Load cohort-level survival summary statistics (e.g., the 226-biomarker BBMRI-omics atlas file) into R and fit Cox proportional hazards models per cohort with age at blood sampling as the time scale, adjusted for sex and study-specific covariates. Preprocess biomarkers as x+1 for biomarkers containing zeroes, log-transform, and scale to standard deviation units separately per cohort. Combine cohort-level lnHR estimates with a fixed-effect meta-analysis implemented in the R-package meta, tabulating HR, 95% CI, P value, I2 heterogeneity statistic and P(het) for each biomarker. Run the stepwise (forward-backward) procedure on the 63-biomarker subset chosen to prevent overfitting: each round add the unselected biomarker with the lowest P value, drop biomarkers whose P rises above the Bonferroni-adjusted threshold of 2.21e-4 (0.05/226), and stop when all unselected biomarkers exceed the threshold. This yielded 14 independently associated biomarkers, with citrate dropping out due to limited additional contribution; record direction of effect (lower risk: XXL-VLDL lipids, small-HDL lipids, VLDL diameter, PUFA/FA, His, Leu, Val, Alb; higher risk: Glc, Lac, Ile, Phe, AcAce, GlycA).

## Related tools

- **R** (Environment for fitting per-cohort Cox proportional hazards models with age at blood sampling as time scale and for running the stepwise selection)
- **R-package meta** (Implements the fixed-effect model used to combine cohort-level survival estimates and derive I2/P(het))

## Examples

```
meta_res <- meta::metagen(TE=lnHR, seTE=se, data=cohort_stats, sm="HR", fixed=TRUE); summary(meta_res)$I2
```

## Evaluation signals

- Exactly 14 biomarkers are retained after the stepwise procedure, with citrate dropping out due to limited additional contribution
- Tabulated HRs and 95% CIs match published Table 2 values (e.g., GlycA HR 1.32, 95% CI 1.27-1.38; XXL-VLDL-L HR 0.80, 95% CI 0.75-0.85)
- Significance threshold applied is the Bonferroni-adjusted 2.21e-4 (0.05/226) at every stepwise round
- Selection is restricted to the 63-biomarker subset to prevent overfitting
- Direction of effect is correct: lower mortality for XXL-VLDL lipids, small-HDL lipids, VLDL diameter, PUFA/FA, His, Leu, Val, Alb; higher mortality for Glc, Lac, Ile, Phe, AcAce, GlycA

## Limitations

- I2 heterogeneity is reported but no formal sensitivity analysis across cohorts is described; the fixed-effect model assumes homogeneous effects
- The score built from these weights is based on per-cohort scaled biomarker values, so it 'is not yet suitable for classification of patients in the clinic'
- The targeted NMR platform captures 'only a fraction of the metabolites in the human serum', so the biomarker set is not exhaustive

## Evidence

- [discussion] fixed-effect meta-analysis implementation: "the meta-analyses were performed using a fixed-effect model implemented in the R-package meta"
- [discussion] Cox model specification: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] Bonferroni threshold: "Biomarkers were considered significant when the P value was below the Bonferroni-adjusted threshold of 2.21 × 10−4 (0.05/226)"
- [discussion] 63-biomarker subset rationale: "the selection of independently associated biomarkers was based on a subset of 63 biomarkers to prevent overfitting"
- [discussion] preprocessing of biomarkers: "all biomarkers were log-transformed and scaled to standard deviation units, separately per study"
