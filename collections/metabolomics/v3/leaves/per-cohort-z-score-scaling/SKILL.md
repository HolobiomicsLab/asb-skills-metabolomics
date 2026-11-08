---
name: per-cohort-z-score-scaling
description: Use when a matrix of Nightingale NMR metabolic biomarker concentrations
  contains below-detection-limit zeroes and must be log-transformed and z-scaled within
  each study before cohort-level survival or meta-analysis.
when_to_use_negative:
- Building a clinically deployable risk score in concentration units — the paper explicitly
  notes the score based on per-cohort scaled biomarker values 'is not yet suitable
  for classification of patients in the clinic'
- Biomarker data are already log-transformed and z-scaled within cohort, or the platform
  output contains no below-detection-limit zeroes
- The analysis targets absolute physiological concentration comparisons across cohorts
  rather than within-cohort relative risk modeling
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Used to add 1 to biomarkers containing zeroes, log-transform, and z-scale
    each biomarker separately per study to produce the analysis-ready matrix
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:cdf6c882e91df37526838f82ec049ea3b59445b245038bb9105d0070ee22572d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# per-cohort-z-score-scaling

## Summary

Prepares the analysis-ready biomarker matrix from the 226 Nightingale NMR metabolic biomarkers by adding 1 to values below the detection limit (zeroes), log-transforming, and scaling each biomarker to standard deviation units separately within each of the 12 cohorts (44,168 individuals), as required before Cox proportional hazards modeling.

## When to use

Use when a matrix of Nightingale NMR metabolic biomarker concentrations contains below-detection-limit zeroes and must be log-transformed and z-scaled within each study before cohort-level survival or meta-analysis.

## When NOT to use

- Building a clinically deployable risk score in concentration units — the paper explicitly notes the score based on per-cohort scaled biomarker values 'is not yet suitable for classification of patients in the clinic'
- Biomarker data are already log-transformed and z-scaled within cohort, or the platform output contains no below-detection-limit zeroes
- The analysis targets absolute physiological concentration comparisons across cohorts rather than within-cohort relative risk modeling

## Inputs

- Per-cohort matrix of 226 metabolic biomarkers quantified on the Nightingale Health NMR metabolomics platform from EDTA plasma and serum samples (12 cohorts, 44,168 individuals)

## Outputs

- scaled_biomarker_matrix.csv: cohort-level analysis-ready biomarker matrix (44,168 rows total), each biomarker column log-transformed and scaled to mean 0, SD 1 within its study

## How to apply

Load the per-cohort matrix of the 226 metabolic biomarkers quantified on the Nightingale Health NMR platform from EDTA plasma and serum samples. For each study separately, add a value of one to all biomarker entries containing zeroes (x + 1), since zeroes mark concentrations below the detection limit and would be undefined under a log transform. Log-transform every biomarker variable within the cohort, then scale each log-transformed biomarker to standard deviation units (mean 0, SD 1) per study using R. This per-study scaling is deliberate: the Cox models with age at blood sampling as the time scale and the meta-analyses require comparably distributed inputs, and the resulting ln(hazard ratio) weights are applied to these scaled values when constructing the metabolic biomarker score. Write the cohort-level result to scaled_biomarker_matrix.csv.

## Related tools

- **R** (Used to add 1 to biomarkers containing zeroes, log-transform, and z-scale each biomarker separately per study to produce the analysis-ready matrix)

## Examples

```
scaled <- scale(log(biom + 1))  # per cohort, R; zeros offset by +1, then log-transform and z-scale each biomarker within study
```

## Evaluation signals

- scaled_biomarker_matrix.csv exists in CSV format with 44,168 rows in total
- Every scaled biomarker column has approximately zero mean and unit standard deviation within each study
- No log-transform was applied to raw zeroes: every biomarker containing zeroes was offset by +1 before log-transformation (x + 1)
- Scaling was performed separately per study (12 cohorts), not pooled across the full 44,168-individual matrix

## Limitations

- The metabolic biomarker score constructed on these scaled values is not yet suitable for clinical patient classification because scaling is created separately for each cohort; a score based on concentration units from individual-level data is still needed
- Zero-valued (below-detection-limit) entries are censored by the +1 offset rather than imputed, so true concentration information at the detection limit is lost
- Because z-scaling is per-cohort, absolute biomarker levels are not directly comparable across cohorts after transformation

## Evidence

- [discussion] Biomarkers containing zeroes had 1 added (x+1), then log-transformed and scaled to standard deviation units per study: [evidence span withheld — anchor status: not_found]
- [discussion] Per-cohort scaling limits clinical use of the score: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
- [discussion] Downstream modeling context: Cox model on scaled biomarkers: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
