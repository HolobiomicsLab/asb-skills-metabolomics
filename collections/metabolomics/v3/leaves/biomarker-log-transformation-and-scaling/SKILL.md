---
name: biomarker-log-transformation-and-scaling
description: Use when preparing Nightingale-platform metabolic biomarker concentration
  tables (with skewness and cohort-specific distributions) for Cox proportional hazards
  models or meta-analysis, especially when zeroes and cross-cohort scale differences
  are present.
when_to_use_negative:
- Biomarker values are already on the log scale or already z-standardized per cohort
  (re-applying x+1/log would double-transform).
- You need a clinically deployable risk score in original concentration units — the
  article notes the per-cohort scaled score 'is not yet suitable for classification
  of patients in the clinic'.
- Data come from a different metabolomics platform with different scaling behavior
  and no zero-inflated biomarkers; verify transformation assumptions before applying.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: R
  role: Environment in which the per-cohort x+1, log-transformation and SD-scaling
    of the 226 biomarkers is implemented before Cox modeling
- name: Nightingale Health NMR metabolomics platform
  role: Source instrument generating the raw biomarker concentrations that require
    this preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-11311-9
    title: A metabolic profile of all-cause mortality risk identified in an observational
      study of 44,168 individuals
  source_content_hash: sha256:e03ebbd76803f02af7f5ab23ed184caf98232616ec9b4fcb09935438f52969ea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nightingale Health NMR metabolomics platform
  - R
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# biomarker-log-transformation-and-scaling

## Summary

Per-cohort preprocessing of circulating metabolic biomarkers (Nightingale NMR platform, 226 biomarkers) prior to survival analysis: add 1 to biomarkers containing zeroes, log-transform, and scale to standard deviation units separately per study so effect sizes are comparable in SD-unit hazard ratios.

## When to use

Use when preparing Nightingale-platform metabolic biomarker concentration tables (with skewness and cohort-specific distributions) for Cox proportional hazards models or meta-analysis, especially when zeroes and cross-cohort scale differences are present.

## When NOT to use

- Biomarker values are already on the log scale or already z-standardized per cohort (re-applying x+1/log would double-transform).
- You need a clinically deployable risk score in original concentration units — the article notes the per-cohort scaled score 'is not yet suitable for classification of patients in the clinic'.
- Data come from a different metabolomics platform with different scaling behavior and no zero-inflated biomarkers; verify transformation assumptions before applying.

## Inputs

- Cohort-level metabolic biomarker concentration matrices (226 biomarkers per individual; EDTA plasma or serum, quantified on the Nightingale Health NMR platform)
- Indicator of which biomarkers contain zero values (or the raw values themselves to detect them)

## Outputs

- Log-transformed, per-cohort scaled (z-score) biomarker matrices ready for Cox proportional hazards regression
- Per-cohort scaling parameters (means/SDs) implicitly needed to reproduce SD-unit HRs and biomarker-score weights

## How to apply

For each cohort separately, first add a value of one to all biomarkers containing zeroes (x+1) so log-transformation is defined, then log-transform all biomarker values, and finally scale to standard deviation units (z-scores) per biomarker per study. The per-cohort scaling ensures that hazard ratios are expressed per 1-SD increase and makes cohort-level estimates combinable in a fixed-effect meta-analysis (R-package meta) of 226 biomarkers across 12 cohorts. Apply the transformation identically in every cohort before fitting Cox models with age at blood sampling as the time scale; note that because scaling is cohort-specific, downstream scores built from these scaled values are not directly usable in the clinic on concentration units. Remember the rationale: biomarker distributions are right-skewed, and log+1 transformation plus standardization stabilizes variance and yields interpretable per-SD effects.

## Related tools

- **R** (Environment in which the per-cohort x+1, log-transformation and SD-scaling of the 226 biomarkers is implemented before Cox modeling)
- **Nightingale Health NMR metabolomics platform** (Source instrument generating the raw biomarker concentrations that require this preprocessing)

## Examples

```
scaled <- apply(raw_biomarkers, 2, function(x) { x <- log(x + 1); (x - mean(x)) / sd(x) })  # per cohort, before Cox PH with age as time scale
```

## Evaluation signals

- All biomarkers that contained zeroes must show a minimum value of 1 after the x+1 step (no non-positive values entering the log).
- After scaling, each biomarker within a cohort has mean 0 and SD 1 (z-score check).
- The transformation must be applied independently per cohort — verify no pooling across the 12 cohorts before scaling.
- Downstream per-SD hazard ratios should match published magnitudes (e.g., GlycA HR 1.32, 95% CI 1.27-1.38; XXL-VLDL-L HR 0.80, 95% CI 0.75-0.85) after meta-analysis.

## Limitations

- Because scaled biomarker values are created separately for each cohort, the resulting metabolic biomarker score is not suitable for clinical patient classification on concentration units.
- The x+1 offset is an arbitrary constant; results can depend on the offset for biomarkers with many zeroes.
- The targeted NMR platform captures only a fraction of metabolites in human serum, limiting generalization of the transformed feature set.

## Evidence

- [discussion] a value of one was added to all biomarkers containing zeroes (i.e., x + 1) ... all biomarkers were log-transformed and scaled to standard deviation units, separately per study: [evidence span withheld — anchor status: not_found]
- [discussion] Cox model with age as time scale: "a Cox proportional hazards model with age at blood sampling as the time scale, was used to determine the associations of the biomarkers with all-cause mortality"
- [discussion] score not clinically usable: "the metabolic biomarker score constructed is not yet suitable for classification of patients in the clinic, since it is based on scaled biomarker values created separately for each cohort"
- [discussion] summary statistics in BBMRI-omics atlas: "The summary statistics of our primary survival meta-analysis have been made publically available in the BBMRI -omics atlas: http://bbmri.researchlumc.nl/atlas"
- [discussion] NMR platform coverage limitation: "the number of biomarkers captured by our targeted NMR platform is only a fraction of the metabolites in the human serum24."
