---
name: polynomial-regression-fitting
description: Use when retention times predicted under one chromatographic method (e.g.
  the METLIN SMRT reverse-phase method) must be made comparable to RTs measured under
  independent experimental methods (ECMs), via a per-method polynomial projection.
when_to_use_negative:
- When the predicted and experimental RTs already share the same chromatographic method
  and time scale, so no projection is needed
- When fewer molecules than the required shared subset (50 molecules with RT in both
  methods) are available for an ECM, undermining the least-squares fit of a four-polynomial
  function
- When the target chromatographic method is not reverse-phase LC, outside the PredRet
  ECMs and SMRT reverse-phase setting covered here
tools:
- name: R stats package (poly function)
  role: Builds the four-polynomial basis used in the robust least-squares regression
    that maps DLM-predicted RT onto each ECM's experimental RT scale
- name: PredRet database
  role: Source of the experimental retention times for the four reverse-phase ECMs
    used as projection targets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:65493d213f84ca3144c22aae5cf1054856ff421cc881db3ba77c31325b05f494
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PredRet database
  - R
  - R stats package (poly function)
  - stats (R package)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# polynomial-regression-fitting

## Summary

Fits a least-squares polynomial regression (R stats::poly, four polynomials) on a small shared-molecule subset to project deep-learning-predicted retention times onto independent experimental chromatographic methods (ECMs), enabling cross-method RT comparability and median absolute error reporting.

## When to use

Use when retention times predicted under one chromatographic method (e.g. the METLIN SMRT reverse-phase method) must be made comparable to RTs measured under independent experimental methods (ECMs), via a per-method polynomial projection.

## When NOT to use

- When the predicted and experimental RTs already share the same chromatographic method and time scale, so no projection is needed
- When fewer molecules than the required shared subset (50 molecules with RT in both methods) are available for an ECM, undermining the least-squares fit of a four-polynomial function
- When the target chromatographic method is not reverse-phase LC, outside the PredRet ECMs and SMRT reverse-phase setting covered here

## Inputs

- SMRT deep-learning-model-predicted retention times for the molecule set (e.g. HMDB KEGG molecules)
- PredRet experimental retention times for the target reverse-phase ECMs (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Per-ECM random subset of 50 molecules with experimental RT available in both the SMRT chromatographic method and the ECM

## Outputs

- Fitted four-polynomial projection function per ECM (one per chromatographic method)
- All predicted RTs transformed onto each ECM time scale
- Median absolute projection error in seconds per ECM (expected: 210 s FEM_long, 9.7 s LIFE_old, 5.7 s RIKEN, 89.4 s FEM_orbitrap_plasma)

## How to apply

Load the DLM-predicted RTs for the molecules (e.g. HMDB KEGG molecules) and the PredRet experimental RTs for each target reverse-phase ECM (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). For each ECM, randomly select 50 molecules whose experimental RT is available in both the SMRT method and that ECM, simulating a partially identified scenario. Fit the projection by least-squares regression of a four-polynomial function built with R's stats::poly(), minimizing error between DLM-predicted RT and experimental RT for the 50 shared molecules. Apply each fitted projection to all predicted RTs to map them onto the ECM time scale, then compute absolute differences against experimental RT for every molecule and report the median absolute error in seconds per ECM. The small (n=50) shared subset is deliberate: the projection must work when only a few standards are identified in a new method.

## Related tools

- **R stats package (poly function)** (Builds the four-polynomial basis used in the robust least-squares regression that maps DLM-predicted RT onto each ECM's experimental RT scale)
- **PredRet database** (Source of the experimental retention times for the four reverse-phase ECMs used as projection targets)

## Examples

```
Rscript -e 'fit <- lm(ecm_rt ~ poly(pred_rt, degree = 4), data = shared50); proj <- predict(fit, newdata = all_pred)'
```

## Evaluation signals

- Reproduced median absolute projection errors match the reported values: 210 s (FEM_long), 9.7 s (LIFE_old), 5.7 s (RIKEN), 89.4 s (FEM_orbitrap_plasma)
- Each ECM yields exactly one fitted four-polynomial model built with stats::poly() from a random subset of 50 shared molecules
- Projected predicted RTs and experimental RTs are on the same time scale, so absolute differences per molecule are computable in seconds
- The 50 shared molecules per ECM each have experimental RT present in both the SMRT method and the target ECM
- Projection is fitted only on the 50-molecule subset, but evaluated by median absolute error over every molecule in the ECM

## Limitations

- Accuracy depends on the ECM: projection errors span nearly two orders of magnitude (5.7 s for RIKEN vs 210 s for FEM_long), so projected RTs are not uniformly reliable across methods
- Requires 50 molecules with experimental RT identified in both methods; projection quality degrades when the shared, identified subset is small or unrepresentative
- The projection is a per-ECM polynomial approximation, not a physical model of chromatographic retention

## Evidence

- [other] projection workflow: [evidence span withheld — anchor status: not_found]
- [intro] cross-method scope: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [abstract] dataset scale: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
