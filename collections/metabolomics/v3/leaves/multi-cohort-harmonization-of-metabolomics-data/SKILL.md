---
name: multi-cohort-harmonization-of-metabolomics-data
description: Use when pooling metabolomics matrices from multiple cohorts (e.g., 226
  Nightingale NMR biomarkers across 12 studies, N=44,168) that contain below-detection-limit
  zeros and need per-cohort transformation before joint survival modeling or meta-analysis.
when_to_use_negative:
- A score in absolute concentration units is required (e.g., for clinical patient
  classification), since per-cohort scaled values are explicitly not suitable for
  this purpose in the article
- The input is already an analysis-ready, transformed biomarker matrix — re-applying
  the x+1 shift, log transform, and z-scaling would distort values
- The biomarkers come from a different metabolomics platform without zero-valued detection-limit
  entries, where the x+1 censoring correction is not applicable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment in which the per-cohort x+1 shift, log transformation, z-scaling,
    and writing of scaled_biomarker_matrix.csv are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:d18a377165e84c7297368d188dd615dbd3d646ab1c50f85e5865a4c7ad0f753b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# multi-cohort-harmonization-of-metabolomics-data

## Summary

Harmonizes metabolic biomarker measurements across multiple cohorts by handling below-detection-limit values (x+1 shift), log-transforming, and z-scaling each biomarker separately per study, producing an analysis-ready biomarker matrix suitable for joint Cox survival modeling and meta-analysis.

## When to use

Use when pooling metabolomics matrices from multiple cohorts (e.g., 226 Nightingale NMR biomarkers across 12 studies, N=44,168) that contain below-detection-limit zeros and need per-cohort transformation before joint survival modeling or meta-analysis.

## When NOT to use

- A score in absolute concentration units is required (e.g., for clinical patient classification), since per-cohort scaled values are explicitly not suitable for this purpose in the article
- The input is already an analysis-ready, transformed biomarker matrix — re-applying the x+1 shift, log transform, and z-scaling would distort values
- The biomarkers come from a different metabolomics platform without zero-valued detection-limit entries, where the x+1 censoring correction is not applicable

## Inputs

- Per-cohort matrix of 226 metabolic biomarkers quantified on the Nightingale Health NMR metabolomics platform (EDTA plasma and serum samples)
- Cohort membership labels mapping the 44,168 individuals to the 12 source studies

## Outputs

- Cohort-level analysis-ready biomarker matrices written to scaled_biomarker_matrix.csv (44,168 rows total; each scaled biomarker column with approximately zero mean and unit SD within each study)

## How to apply

Load the per-cohort matrix of the 226 metabolic biomarkers quantified on the Nightingale Health NMR platform from EDTA plasma and serum samples. For each study, add a value of one to all biomarker entries containing zeroes, which mark concentrations below the detection limit, so that log transformation is defined on all values. Log-transform every biomarker variable within each cohort, then scale each log-transformed biomarker to standard deviation units (mean 0, SD 1) separately per study, covering all 44,168 individuals across the 12 cohorts (FINRISK 1997, Alpha Omega Cohort, ERF, DILGOM, LLS, PROSPER, Rotterdam Study, ALSPAC, EGCUT, KORA F4, TwinsUK, Estonian Biobank). The per-cohort scaling is deliberate: it standardizes units across studies for meta-analyzed Cox models, but the resulting score is explicitly not yet suitable for clinical classification because it is based on cohort-specific scaled values rather than concentration units. Write the cohort-level analysis-ready matrices to scaled_biomarker_matrix.csv.

## Related tools

- **R** (Environment in which the per-cohort x+1 shift, log transformation, z-scaling, and writing of scaled_biomarker_matrix.csv are implemented)

## Examples

```
Rscript preprocess_biomarkers.R --input raw_nightingale_226.csv --cohorts 12 --output scaled_biomarker_matrix.csv
```

## Evaluation signals

- The output file exists in CSV format with 44,168 rows in total, matching all individuals across the 12 cohorts
- Each scaled biomarker column has approximately zero mean and unit standard deviation within each study (check per cohort, not pooled)
- Every biomarker entry that originally contained zeroes (below-detection-limit values) was shifted by +1 before log transformation
- All 226 biomarker columns are retained after preprocessing — no columns dropped or imputed out

## Limitations

- The metabolic biomarker score constructed from these per-cohort scaled values is not yet suitable for classification of patients in the clinic; a score based on concentration units from individual-level data is still needed
- Handling below-detection-limit values by adding one (x+1) before log transformation is a censoring approximation, not a formal censored-data method
- The targeted NMR platform captures only a fraction of the metabolites in human serum; more complete high-throughput metabolic biomarker platforms were not compared

## Evidence

- [discussion] per-cohort zero handling and scaling: [evidence span withheld — anchor status: not_found]
- [discussion] scope of biomarker quantification: "The metabolic biomarkers were quantified from EDTA plasma and serum samples using high‐throughput NMR metabolomics (Nightingale Health Ltd., Helsinki, Finland)"
- [abstract] study scale motivating multi-cohort harmonization: "we use a well-standardized metabolomics platform to identify metabolic predictors of long-term mortality in the circulation of 44,168 individuals"
- [discussion] clinical-use caveat of per-cohort scaling: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
