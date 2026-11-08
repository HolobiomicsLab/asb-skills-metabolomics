---
name: chromatographic-elution-order-comparison
description: Use when predicted retention times from one chromatographic method (e.g.,
  SMRT reverse-phase) must be made comparable to RTs measured under other experimental
  methods (e.g., PredRet ECMs), and median absolute projection errors are needed.
when_to_use_negative:
- The experimental RTs and predicted RTs are already on the same chromatographic method/time
  scale and no cross-method projection is needed
- Fewer than the ~50 shared molecules with RT in both methods are available for a
  target ECM, leaving too few points to fit a four-polynomial regression reliably
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R stats poly()
  role: Fits the four-polynomial least-squares regression that projects DLM-predicted
    RT onto each ECM's experimental RT scale
- name: PredRet database
  role: Source of experimental retention times for the reverse-phase external chromatographic
    methods (ECMs)
- name: METLIN SMRT dataset
  role: Source of experimentally acquired reverse-phase RTs (up to 80,038 small molecules)
    used to train the DLM whose predictions are projected
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:c00fc71cee199866734e19816bbcf1f50e89373eaf2498700eb18a492f8571c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - METLIN SMRT dataset
  - PredRet database
  - R
  - R stats poly()
  - stats (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# chromatographic-elution-order-comparison

## Summary

Projects deep-learning-model (SMRT DLM) predicted retention times onto independent experimental chromatographic methods by fitting a robust least-squares four-polynomial regression per external chromatographic method (ECM), enabling comparison of predicted and measured RT across chromatographic systems.

## When to use

Use when predicted retention times from one chromatographic method (e.g., SMRT reverse-phase) must be made comparable to RTs measured under other experimental methods (e.g., PredRet ECMs), and median absolute projection errors are needed.

## When NOT to use

- The experimental RTs and predicted RTs are already on the same chromatographic method/time scale and no cross-method projection is needed
- Fewer than the ~50 shared molecules with RT in both methods are available for a target ECM, leaving too few points to fit a four-polynomial regression reliably

## Inputs

- DLM-predicted retention times for target molecules (e.g., SMRT model predictions for HMDB KEGG molecules)
- Experimental retention times for external chromatographic methods (PredRet ECMs: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Random subset of 50 molecules per ECM with RT available in both the SMRT method and the ECM

## Outputs

- Fitted four-polynomial projection function per ECM (R stats poly() regression)
- Projected predicted RTs for all molecules on each ECM time scale
- Median absolute projection error in seconds per ECM

## How to apply

Load the DLM-predicted retention times (e.g., for HMDB KEGG molecules from the METLIN SMRT dataset, covering up to 80,038 small molecules) and the experimental retention times of the target ECMs (e.g., FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma from PredRet). For each ECM, randomly select 50 molecules whose experimental RT is available in both the SMRT chromatographic method and the ECM, simulating a partially identified scenario. Fit the projection via least-squares regression of a four-polynomial function using the R stats poly() function, minimizing the error between DLM-predicted RT and experimental RT for those 50 shared molecules. Apply each fitted projection to transform all predicted RTs onto the corresponding ECM time scale, then compute absolute differences between projected predicted RT and experimental RT for every molecule and derive the median absolute error in seconds per ECM.

## Related tools

- **R stats poly()** (Fits the four-polynomial least-squares regression that projects DLM-predicted RT onto each ECM's experimental RT scale)
- **PredRet database** (Source of experimental retention times for the reverse-phase external chromatographic methods (ECMs))
- **METLIN SMRT dataset** (Source of experimentally acquired reverse-phase RTs (up to 80,038 small molecules) used to train the DLM whose predictions are projected)

## Examples

```
fit <- lm(ecm_rt ~ poly(smrt_pred_rt, degree = 4), data = shared_50); projected <- predict(fit, newdata = all_pred)
```

## Evaluation signals

- Reproduced median absolute errors should match the reported values: 210 s (FEM_long), 9.7 s (LIFE_old), 5.7 s (RIKEN), 89.4 s (FEM_orbitrap_plasma)
- Each ECM projection is fitted on exactly 50 molecules randomly selected as having experimental RT in both the SMRT method and the ECM
- Regression uses four polynomials via R stats poly() minimizing error between DLM-predicted RT and experimental RT
- Per-molecule absolute differences between projected predicted RT and experimental RT are computed for every molecule, not just the training subset

## Limitations

- Errors vary dramatically across ECMs (5.7 s to 210 s), so projection quality depends strongly on how similar the target method is to the SMRT chromatographic method
- The projection is trained on a random 50-molecule subset per ECM, simulating a partially identified scenario; results depend on that subset draw

## Evidence

- [abstract] abstract: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] intro: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [methods] methods: [evidence span withheld — anchor status: not_found]
