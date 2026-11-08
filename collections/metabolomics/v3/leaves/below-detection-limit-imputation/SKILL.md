---
name: below-detection-limit-imputation
description: Use when a biomarker matrix (e.g., 226 Nightingale NMR metabolites) contains
  zeros representing concentrations below the detection limit and the values must
  be made analysis-ready for per-cohort regression or survival modeling.
when_to_use_negative:
- The biomarker values are already on concentration units without zero-coded below-detection-limit
  entries, so no x + 1 shift is needed before log-transformation
- The goal is a clinically deployable metabolic biomarker score on raw concentration
  units — the article notes the per-cohort scaled score is not suitable for patient
  classification in the clinic
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Used to apply the x + 1 shift for below-detection-limit values, log-transform
    all 226 biomarkers, scale each to SD units per study, and write scaled_biomarker_matrix.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:cd27771de58842e3d9ff3c09a089adfce1e0e261ec7bbe7df87762f940ae46af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# below-detection-limit-imputation

## Summary

Prepares the 226 Nightingale NMR metabolic biomarkers for Cox survival modeling by shifting below-detection-limit values (x + 1), log-transforming, and scaling to SD units separately per cohort, yielding an analysis-ready biomarker matrix.

## When to use

Use when a biomarker matrix (e.g., 226 Nightingale NMR metabolites) contains zeros representing concentrations below the detection limit and the values must be made analysis-ready for per-cohort regression or survival modeling.

## When NOT to use

- The biomarker values are already on concentration units without zero-coded below-detection-limit entries, so no x + 1 shift is needed before log-transformation
- The goal is a clinically deployable metabolic biomarker score on raw concentration units — the article notes the per-cohort scaled score is not suitable for patient classification in the clinic

## Inputs

- Per-cohort matrix of 226 metabolic biomarkers quantified on the Nightingale Health NMR metabolomics platform from EDTA plasma and serum samples
- Cohort membership information for the 44,168 individuals across the 12 cohorts (to apply transformations separately per study)

## Outputs

- scaled_biomarker_matrix.csv — cohort-level analysis-ready biomarker matrices with 44,168 rows in total, each biomarker log-transformed and scaled to mean 0, SD 1 within each study

## How to apply

Load the per-cohort matrix of the 226 metabolic biomarkers quantified on the Nightingale Health NMR platform from EDTA plasma and serum samples. For each study, add a value of one to all biomarker entries containing zeroes (i.e., x + 1), since zeroes indicate concentrations below the detection limit. Log-transform every biomarker variable within each cohort, then scale each log-transformed biomarker to standard deviation units (mean 0, SD 1) separately per study — the per-study scaling is deliberate so the matrix is comparable within cohorts while avoiding cross-cohort leakage. The transformation covers all 44,168 individuals across the 12 cohorts and produces the analysis-ready input for the Cox proportional hazards models with age at blood sampling as the time scale.

## Related tools

- **R** (Used to apply the x + 1 shift for below-detection-limit values, log-transform all 226 biomarkers, scale each to SD units per study, and write scaled_biomarker_matrix.csv)

## Examples

```
biom_shift <- log(biom + (colSums(biom == 0) > 0)); biom_scaled <- scale(biom_shift); write.csv(biom_scaled, "scaled_biomarker_matrix.csv")
```

## Evaluation signals

- scaled_biomarker_matrix.csv exists and is a valid CSV file
- The output contains 44,168 rows in total across the 12 cohorts
- Each scaled biomarker column has approximately zero mean and unit standard deviation within each study
- No zero values remain in the log-transformed columns: every biomarker that contained zeroes had 1 added before log-transformation
- Scaling statistics differ per cohort (applied separately per study, not pooled across all 44,168 individuals)

## Limitations

- The metabolic biomarker score constructed from these per-cohort scaled values is not suitable for clinical patient classification, since scaling is created separately for each cohort; a score based on concentration units from individual-level data is still needed
- The targeted NMR platform captures only a fraction of the metabolites in human serum, so the transformed matrix is limited to the 226 quantified biomarkers

## Evidence

- [discussion] a value of one was added to all biomarkers containing zeroes (i.e., x + 1) ... all biomarkers were log-transformed and scaled to standard deviation units, separately per study: [evidence span withheld — anchor status: not_found]
- [discussion] the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
- [discussion] the number of biomarkers captured by our targeted NMR platform is only a fraction of the metabolites in the human serum: "the number of biomarkers captured by our targeted NMR platform is only a fraction of the metabolites in the human serum"
