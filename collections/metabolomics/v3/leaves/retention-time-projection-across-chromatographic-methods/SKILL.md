---
name: retention-time-projection-across-chromatographic-methods
description: Use when you have deep-learning-predicted RTs on one chromatographic
  method and need them transformed onto the time scale of other experimental reverse-phase
  methods (ECMs) to compute projection errors or rank candidates.
when_to_use_negative:
- The target chromatographic method is not reverse-phase LC (the SMRT dataset is an
  experimentally acquired reverse-phase chromatography retention time dataset, so
  projections to normal-phase or HILIC methods are out of scope).
- No partially identified calibration set exists — i.e. fewer than the ~50 shared
  molecules with RT in both the SMRT method and the target ECM are available to fit
  the projection.
- The goal is de novo RT prediction from molecular structure rather than projecting
  existing SMRT-predicted RTs onto another method's time scale.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: R
  role: Implementation language for the least-squares polynomial projection regression
- name: stats (R package)
  role: Provides the poly() function used to fit the four-polynomial projection of
    DLM-predicted RT onto each ECM
- name: PredRet database
  role: Source of experimental retention times for the independent reverse-phase ECMs
    (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:9f6dddadaf7a11703d6b80377172d4c96a7c7e59049d3e66262e6209693a3aec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PredRet database
  - R
  - stats (R package)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# retention-time-projection-across-chromatographic-methods

## Summary

Projects retention times predicted by the SMRT deep-learning model (trained on the METLIN SMRT reverse-phase dataset) onto independent experimental chromatographic methods (ECMs) using a robust least-squares four-degree polynomial fit, so predicted RTs become comparable with RTs measured under other methods.

## When to use

Use when you have deep-learning-predicted RTs on one chromatographic method and need them transformed onto the time scale of other experimental reverse-phase methods (ECMs) to compute projection errors or rank candidates.

## When NOT to use

- The target chromatographic method is not reverse-phase LC (the SMRT dataset is an experimentally acquired reverse-phase chromatography retention time dataset, so projections to normal-phase or HILIC methods are out of scope).
- No partially identified calibration set exists — i.e. fewer than the ~50 shared molecules with RT in both the SMRT method and the target ECM are available to fit the projection.
- The goal is de novo RT prediction from molecular structure rather than projecting existing SMRT-predicted RTs onto another method's time scale.

## Inputs

- DLM-predicted retention times for HMDB/KEGG molecules (SMRT chromatographic method time scale)
- Experimental retention times for target reverse-phase ECMs from the PredRet database (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Per-ECM set of ~50 shared molecules with experimental RT available in both the SMRT method and the target ECM

## Outputs

- A fitted four-degree polynomial projection function per ECM (one per chromatographic method)
- All predicted RTs transformed onto each ECM's experimental time scale
- Median absolute projection error in seconds per ECM (reported: 210 s FEM_long, 9.7 s LIFE_old, 5.7 s RIKEN, 89.4 s FEM_orbitrap_plasma)

## How to apply

Load the DLM-predicted retention times (e.g. for HMDB/KEGG molecules) and the experimental RTs of the target ECMs (e.g. PredRet reverse-phase methods FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). For each ECM, randomly select 50 molecules whose experimental RT is available in both the SMRT chromatographic method and the ECM, simulating a partially identified scenario. Fit the projection via robust least-squares regression of a four-polynomial function using the R stats::poly() function, minimizing the error between DLM-predicted RT and experimental RT of the 50 shared molecules. Apply each fitted projection to transform all predicted RTs onto the ECM time scale, then compute absolute differences and the median absolute error in seconds per ECM as the performance metric.

## Related tools

- **R** (Implementation language for the least-squares polynomial projection regression)
- **stats (R package)** (Provides the poly() function used to fit the four-polynomial projection of DLM-predicted RT onto each ECM)
- **PredRet database** (Source of experimental retention times for the independent reverse-phase ECMs (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma))

## Examples

```
fit <- lm(experimental_RT ~ poly(predicted_RT, 4), data = shared_50); projected_RT <- predict(fit, newdata = all_predicted)
```

## Evaluation signals

- Reproduced median absolute projection errors match the reported values: 210 s (FEM_long), 9.7 s (LIFE_old), 5.7 s (RIKEN) and 89.4 s (FEM_orbitrap_plasma).
- Each projection is fitted from exactly 50 randomly selected molecules per ECM that have experimental RT in both the SMRT chromatographic method and the target ECM.
- The projection is a four-degree polynomial (stats::poly) fitted by least squares against the DLM-predicted RT; a fit using a different polynomial order or loss indicates deviation from the method.
- For every molecule, the absolute difference between projected predicted RT and experimental RT is computable on the target ECM's time scale, and the per-ECM median absolute error is reported in seconds.

## Limitations

- Accuracy is highly method-dependent: projected errors range from 5.7 s (RIKEN) to 210 s (FEM_long), so projections to chromatographic methods dissimilar from the SMRT reverse-phase method can be poor.
- Each projection relies on only 50 shared molecules per ECM, simulating a partially identified scenario; the fit quality depends on this small random subset.
- The projection requires molecules with experimentally measured RT in both methods; it cannot be applied to ECMs lacking shared reference RTs.

## Evidence

- [abstract] SMRT reverse-phase dataset scope: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] Projection ability is an explicit analysis goal: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [methods] Polynomial least-squares projection fitting procedure: [evidence span withheld — anchor status: not_found]
- [results] Reported validation errors per ECM: [evidence span withheld — anchor status: not_found]
