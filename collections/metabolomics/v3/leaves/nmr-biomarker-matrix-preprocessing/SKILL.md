---
name: nmr-biomarker-matrix-preprocessing
description: Use when raw NMR metabolomics biomarker concentrations (e.g., 226 Nightingale
  Health platform measures in EDTA plasma/serum) contain zeroes marking values below
  the detection limit and must be made analysis-ready for per-cohort Cox survival
  models.
when_to_use_negative:
- The biomarker matrix contains no zeroes/below-detection-limit entries and values
  are already on the log scale needed for modeling — the x+1 offset would distort
  genuine low concentrations.
- The goal is a clinically deployable mortality risk score in concentration units;
  the paper explicitly states the score built on per-cohort scaled biomarker values
  'is not yet suitable for classification of patients in the clinic' and that concentration-unit
  scores still need to be created.
- The data come from a different metabolomics platform or untargeted assay for which
  per-study z-scaling of 226 Nightingale biomarkers is not the appropriate normalization.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: R
  role: Environment in which the x+1 / log-transform / per-study scaling is performed
    and in which the scaled matrices feed Cox proportional hazards survival analyses
- name: Nightingale Health NMR metabolomics platform
  role: Source of the 226 metabolic biomarkers quantified from EDTA plasma and serum
    samples that constitute the raw matrix being preprocessed
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:d38a27aa71c49ae5ca0daa5e6c9c079222a17f2794ff8de7d8ee4ea73848ccb5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# nmr-biomarker-matrix-preprocessing

## Summary

Per-cohort preprocessing of Nightingale Health NMR metabolic biomarker measurements: adds 1 to biomarker values below the detection limit (zeroes), log-transforms all biomarkers, and scales them to standard deviation units within each study, yielding an analysis-ready biomarker matrix for Cox proportional hazards survival modeling.

## When to use

Use when raw NMR metabolomics biomarker concentrations (e.g., 226 Nightingale Health platform measures in EDTA plasma/serum) contain zeroes marking values below the detection limit and must be made analysis-ready for per-cohort Cox survival models.

## When NOT to use

- The biomarker matrix contains no zeroes/below-detection-limit entries and values are already on the log scale needed for modeling — the x+1 offset would distort genuine low concentrations.
- The goal is a clinically deployable mortality risk score in concentration units; the paper explicitly states the score built on per-cohort scaled biomarker values 'is not yet suitable for classification of patients in the clinic' and that concentration-unit scores still need to be created.
- The data come from a different metabolomics platform or untargeted assay for which per-study z-scaling of 226 Nightingale biomarkers is not the appropriate normalization.

## Inputs

- Per-cohort matrix of 226 metabolic biomarkers quantified on the Nightingale Health NMR metabolomics platform from EDTA plasma and serum samples
- Cohort metadata indicating study membership (e.g., FINRISK 1997, Alpha Omega Cohort, ERF, DILGOM, KORA F4, LLS, PROSPER, ALSPAC, EGCUT, TwinsUK, Rotterdam Study, Estonian Biobank) so transformations are applied separately per study

## Outputs

- Cohort-level analysis-ready biomarker matrix (scaled_biomarker_matrix.csv) with 44,168 rows total, each biomarker log-transformed and scaled to mean 0, SD 1 within each study

## How to apply

Load the per-cohort matrix of the 226 metabolic biomarkers quantified on the Nightingale Health NMR platform from EDTA plasma and serum samples. For each study, add a value of one to every biomarker entry containing zeroes, since zeroes indicate concentrations below the detection limit; the x+1 step must precede the log-transform so that below-limit entries remain defined. Log-transform every biomarker variable within the cohort, then scale each log-transformed biomarker to standard deviation units (mean 0, SD 1), applying the transformation separately per study across all 44,168 individuals in the 12 cohorts. The rationale for per-study scaling is consistency with the paper's downstream risk-score construction, in which these scaled biomarker values are multiplied by ln(hazard ratio) weights from the meta-analyses; note this also means the score is not yet directly usable as a clinical classifier in concentration units. Write the cohort-level analysis-ready matrices (e.g., to scaled_biomarker_matrix.csv) before fitting Cox proportional hazards models with age at blood sampling as the time scale in R (survival package) or STATA/SE 11.2.

## Related tools

- **R** (Environment in which the x+1 / log-transform / per-study scaling is performed and in which the scaled matrices feed Cox proportional hazards survival analyses)
- **Nightingale Health NMR metabolomics platform** (Source of the 226 metabolic biomarkers quantified from EDTA plasma and serum samples that constitute the raw matrix being preprocessed)

## Examples

```
Rscript preprocess_biomarkers.R --input biomarker_matrix.csv --output scaled_biomarker_matrix.csv --offset 1 --scale per_study
```

## Evaluation signals

- Output file scaled_biomarker_matrix.csv exists in CSV format and contains 44,168 rows in total across the 12 cohorts.
- Within each study, every scaled biomarker column has approximately zero mean and unit standard deviation.
- Entries that were zero (below detection limit) in the raw matrix are positive after transformation (x+1 ensures log is defined).
- Downstream Cox hazard ratios reproduce published values (e.g., GlycA all-cause HR 1.41, 95% CI 1.26-1.57; PUFA/FA all-cause HR 0.73, 95% CI 0.69-0.78), which depends on identical preprocessing.
- The transformation is verified to have been applied separately per study, not pooled across cohorts.

## Limitations

- The metabolic biomarker score constructed from these scaled biomarkers 'is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort'; a score in concentration units from individual-level data is still needed.
- The targeted NMR platform captures 'only a fraction of the metabolites in the human serum'; more complete high-throughput platforms were not compared.
- Risk-score weights were trained only on the Estonian Biobank cohort and tested only in FINRISK 1997, so scaled-matrix-based scores may not generalize to other cohorts.

## Evidence

- [discussion] a value of one was added to all biomarkers containing zeroes (i.e., x + 1) ... all biomarkers were log-transformed and scaled to standard deviation units, separately per study: "a value of one was added to all biomarkers containing zeroes (i.e., x + 1)"
- [discussion] The metabolic biomarkers were quantified from EDTA plasma and serum samples using high-throughput NMR metabolomics: "The metabolic biomarkers were quantified from EDTA plasma and serum samples using high-throughput NMR metabolomics"
- [discussion] a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality: "a Cox proportional hazards model with age at blood sampling as the time scale, was used"
- [discussion] the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort: "not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
- [discussion] the number of biomarkers captured by our targeted NMR platform is only a fraction of the metabolites in the human serum: "the number of biomarkers captured by our targeted NMR platform is only a fraction of the metabolites in the human serum"
