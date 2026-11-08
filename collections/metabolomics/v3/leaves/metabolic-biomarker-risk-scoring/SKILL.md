---
name: metabolic-biomarker-risk-scoring
description: Use when NMR metabolomics biomarker measurements (e.g., Nightingale platform)
  plus survival data are available and you must build a weighted mortality risk score
  and test whether it outperforms conventional risk factors in a held-out validation
  cohort.
when_to_use_negative:
- Goal is clinical patient classification — the score is based on scaled biomarker
  values created separately per cohort and is not yet suitable for clinical use in
  concentration units
- The platform or measurement set is not the Nightingale NMR panel — the 14 selected
  biomarkers and the exclusion of lipids/creatinine from conventional factors depend
  on what the NMR platform captures
- The analysis target is a phenotype other than prospective all-cause mortality —
  whether similar multivariate direction-of-effect changes occur for other phenotypes
  was not investigated
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Nightingale Health NMR metabolomics platform
  role: High-throughput NMR quantification of metabolic biomarkers from EDTA plasma/serum
    used to build the risk scores
- name: R
  role: Cohort-level survival analyses (Cox models with age at blood sampling as time
    scale) and custom risk-score evaluation functions
- name: STATA/SE 11.2
  role: Used together with R for the cohort-level survival analyses
- name: R-package meta
  role: Fixed-effect meta-analysis across cohorts used to derive the ln(hazard ratio)
    weights for the risk score
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:3138558b0b5d80005824490adedc28c3cd6a246e2362b9d54b90682fdc5d2aaf
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
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# metabolic-biomarker-risk-scoring

## Summary

Constructs and validates a weighted metabolic biomarker risk score for all-cause mortality by multiplying Nightingale NMR-derived, log-transformed, SD-scaled biomarkers by ln(hazard ratio) weights and comparing predictive accuracy (C-statistic, IDI) against conventional risk factor models in an independent cohort.

## When to use

Use when NMR metabolomics biomarker measurements (e.g., Nightingale platform) plus survival data are available and you must build a weighted mortality risk score and test whether it outperforms conventional risk factors in a held-out validation cohort.

## When NOT to use

- Goal is clinical patient classification — the score is based on scaled biomarker values created separately per cohort and is not yet suitable for clinical use in concentration units
- The platform or measurement set is not the Nightingale NMR panel — the 14 selected biomarkers and the exclusion of lipids/creatinine from conventional factors depend on what the NMR platform captures
- The analysis target is a phenotype other than prospective all-cause mortality — whether similar multivariate direction-of-effect changes occur for other phenotypes was not investigated

## Inputs

- Per-cohort log-transformed, SD-scaled biomarker measurements from the Nightingale Health NMR metabolomics platform (EDTA plasma or serum)
- Conventional risk factor measurements (sex, BMI, systolic blood pressure, total cholesterol, HDL cholesterol, triglycerides, creatinine, smoking, alcohol, prevalent diabetes, CVD, cancer)
- Risk-score weights based on meta-analysis ln(hazard ratio) values (Supplementary Table 4, trained in the Estonian Biobank cohort)
- Cohort survival data with all-cause mortality follow-up and age at blood sampling as time scale

## Outputs

- Weighted metabolic biomarker risk score (biomarker-plus-sex model, C-statistic = 0.837 for 5-year and 0.830 for 10-year mortality in FINRISK 1997)
- C-statistics for 5- and 10-year mortality comparing biomarker-only vs. conventional risk factor vs. combined models
- Integrated discrimination improvement (IDI) values for model comparison

## How to apply

Preprocess per-cohort biomarker values by adding 1 to zero-containing biomarkers (x+1), log-transforming, and scaling to standard deviation units separately per study. Fit Cox proportional hazards models with age at blood sampling as the time scale to estimate biomarker-mortality associations, select independently associated biomarkers with a Bonferroni threshold of 2.21 x 10^-4 (0.05/226), and derive score weights as ln(hazard ratio) values trained in the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation. Compute the score as the sum of scaled biomarkers multiplied by their weights; build a biomarker-plus-sex score and a combined score (biomarkers plus conventional risk factors, excluding lipids/creatinine already captured by the Nightingale platform). Validate in the FINRISK 1997 cohort using C-statistics for 5- and 10-year mortality and integrated discrimination improvement (IDI) via R-package meta and custom R functions (cohort-level survival analyses in R and STATA/SE 11.2).

## Related tools

- **Nightingale Health NMR metabolomics platform** (High-throughput NMR quantification of metabolic biomarkers from EDTA plasma/serum used to build the risk scores)
- **R** (Cohort-level survival analyses (Cox models with age at blood sampling as time scale) and custom risk-score evaluation functions)
- **STATA/SE 11.2** (Used together with R for the cohort-level survival analyses)
- **R-package meta** (Fixed-effect meta-analysis across cohorts used to derive the ln(hazard ratio) weights for the risk score)

## Examples

```
score <- as.vector(scale(log2(biomarkers + 1)) %*% ln_hr_weights)  # weights from meta-analysis ln(HR), Table 2 / Supplementary Table 4
```

## Evaluation signals

- The combined model (biomarkers + conventional risk factors) must not yield a higher C-statistic than the biomarker-only model in FINRISK 1997
- The biomarker-plus-sex model should reach C-statistic = 0.837 (5-year) and 0.830 (10-year), exceeding the conventional risk factor model (0.772 and 0.790)
- Score weights must equal ln(hazard ratio) values from the meta-analysis (Table 2 / Supplementary Table 4), with biomarkers log-transformed and SD-scaled per study
- Direction-of-effect check: adjusted HR signs for total lipids in XXL-VLDL and isoleucine must flip relative to unadjusted associations after adjusting for GlycA and leucine/valine
- Biomarker significance must pass the Bonferroni threshold of 2.21 x 10^-4 (0.05/226)

## Limitations

- The metabolic biomarker score is not yet suitable for clinical patient classification since it is based on per-cohort scaled biomarker values; a score in concentration units from individual-level data is still needed
- The targeted NMR platform captures only a fraction of metabolites in human serum; more complete high-throughput platforms were not compared
- Risk score weights were trained only on the Estonian Biobank cohort and predictive ability tested only in the FINRISK 1997 cohort, limiting generalization

## Evidence

- [discussion] preprocessing: [evidence span withheld — anchor status: not_found]
- [discussion] score construction: "the log-transformed and scaled biomarkers were multiplied by their weights, based on the meta-analyses results (i.e., ln(hazard ratio) from Table 2), and subsequently summed"
- [discussion] validation metrics: "The predictive ability of the weighted risk scores was tested in the FINRISK 1997 cohort. We used two measures to assess the predictive value of the risk scores: (1) C-statistics and (2) IDI"
- [discussion] model specification: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [abstract] reported performance: "the prediction accuracy of 5- and 10-year mortality based on a model containing the identified biomarkers and sex (C-statistic = 0.837 and 0.830, respectively) is better than that of a model"
- [discussion] training cohort: "The weights for the risk scores were based on the Estonian Biobank cohort (Supplementary Table 4) to avoid overestimation"
