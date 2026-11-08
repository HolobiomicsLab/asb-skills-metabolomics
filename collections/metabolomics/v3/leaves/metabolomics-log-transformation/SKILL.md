---
name: metabolomics-log-transformation
description: Use when a plasma/serum NMR metabolomics matrix contains zero values
  (below detection limit) that must be shifted, log-transformed, and z-scaled per
  cohort before survival or association modeling.
when_to_use_negative:
- The goal is a clinically deployable score in absolute concentration units — the
  article notes the per-cohort scaled score 'is not yet suitable for classification
  of patients in the clinic'
- The biomarker matrix contains no zero/below-detection-limit values and is already
  on a comparable scale across cohorts
- The downstream analysis requires raw concentration units rather than per-study standardized
  values
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Implement the per-cohort zero-shift, log-transformation and SD scaling of
    the 226 NMR biomarkers and write scaled_biomarker_matrix.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:26bc31ca031522c30400bb44d6aec2f297ef87a31965ce93026c20a3d0b35a0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NormalizeMets
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabolomics-log-transformation

## Summary

Cohort-level preprocessing of Nightingale Health NMR metabolic biomarker matrices: shift below-detection-limit zeros by +1, log-transform, and scale to standard deviation units separately per study, yielding the analysis-ready biomarker matrix for Cox proportional hazards modeling.

## When to use

Use when a plasma/serum NMR metabolomics matrix contains zero values (below detection limit) that must be shifted, log-transformed, and z-scaled per cohort before survival or association modeling.

## When NOT to use

- The goal is a clinically deployable score in absolute concentration units — the article notes the per-cohort scaled score 'is not yet suitable for classification of patients in the clinic'
- The biomarker matrix contains no zero/below-detection-limit values and is already on a comparable scale across cohorts
- The downstream analysis requires raw concentration units rather than per-study standardized values

## Inputs

- Per-cohort raw matrix of 226 Nightingale Health NMR metabolic biomarkers from EDTA plasma and serum samples (44,168 individuals across 12 cohorts), with zeroes denoting below-detection-limit concentrations

## Outputs

- scaled_biomarker_matrix.csv: cohort-level analysis-ready biomarker matrix, log-transformed and scaled to standard deviation units (mean 0, SD 1) per study

## How to apply

Load the per-cohort matrix of the 226 metabolic biomarkers quantified on the Nightingale Health NMR platform from EDTA plasma and serum samples. For each study, add one to every biomarker entry containing zeroes (i.e., x + 1) so that below-detection-limit values become valid for the logarithm. Log-transform every biomarker variable within the cohort, then scale each log-transformed biomarker to standard deviation units (mean 0, SD 1) separately per study. Rationale: zero values flag concentrations below the detection limit and cannot be logged directly, and per-study scaling harmonizes cohorts (12 cohorts, 44,168 individuals) before the Cox proportional hazards model with age at blood sampling as time scale. Write the result to scaled_biomarker_matrix.csv, one matrix per cohort.

## Related tools

- **R** (Implement the per-cohort zero-shift, log-transformation and SD scaling of the 226 NMR biomarkers and write scaled_biomarker_matrix.csv)

## Examples

```
scaled <- scale(log(mat + (mat == 0)))  # shift zeroes by +1, log-transform, then scale to SD units per cohort
```

## Evaluation signals

- Output file exists in CSV format with 44,168 rows in total across the 12 cohorts
- Each scaled biomarker column has approximately zero mean and unit standard deviation within each study
- No zero values remain in the transformed matrix — every biomarker containing zeroes received the x + 1 shift before log-transformation
- All 226 biomarker columns are present and log-transformed, matching the Nightingale NMR panel
- Scaling parameters were computed separately per study, not pooled across cohorts

## Limitations

- Because biomarkers are scaled separately per cohort, downstream scores built on them are not suitable for clinical patient classification; a concentration-unit score would need to be generated from individual-level data
- The +1 shift is a specific handling for below-detection-limit zeroes and should not be blindly applied to biomarkers without zeroes

## Evidence

- [discussion] a value of one was added to all biomarkers containing zeroes (i.e., x + 1) ... all biomarkers were log-transformed and scaled to standard deviation units, separately per study: [evidence span withheld — anchor status: not_found]
- [discussion] NMR platform and sample types: "The metabolic biomarkers were quantified from EDTA plasma and serum samples using high‐throughput NMR metabolomics (Nightingale Health Ltd., Helsinki, Finland)"
- [discussion] downstream use in Cox model: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] per-cohort scaling caveat: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
