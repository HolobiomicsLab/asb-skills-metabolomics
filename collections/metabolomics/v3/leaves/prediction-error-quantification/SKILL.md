---
name: prediction-error-quantification
description: Use when RTs predicted under one chromatographic method (e.g., the METLIN
  SMRT reverse-phase method) must be made comparable with retention times measured
  under other independent experimental LC-MS methods, and when a median absolute error
  (in seconds) must be reported to validate the projection.
when_to_use_negative:
- RT prediction within the SMRT chromatographic method itself — no projection is needed;
  use the DLM predicted RT directly
- Candidate ranking/filtering tasks where predicted RT is used to rank accurate-mass-search
  candidates rather than to quantify cross-method transfer error
- ECMs for which fewer than ~50 molecules have experimental RT in both the SMRT method
  and the target method, where the polynomial fit cannot be reliably estimated
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment in which the least-squares polynomial projection regression and
    error computation are performed
- name: stats (R package)
  role: Provides poly() to construct the four-polynomial basis fitted by least squares
    to map DLM-predicted RT onto each ECM's RT scale
- name: PredRet database
  role: Source of experimental retention times for the four reverse-phase ECMs (FEM_long,
    LIFE_old, RIKEN, FEM_orbitrap_plasma) used as projection targets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:ad6e0af7f4e1194202e3bf7a3678b423421d46488f736a41a19ab9f9f7105ba9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PredRet database
  - R
  - stats (R package)
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# prediction-error-quantification

## Summary

Projects retention times predicted by the METLIN SMRT deep-learning model (DLM) onto independent experimental chromatographic methods (ECMs) from PredRet, then quantifies transfer error as the median absolute difference in seconds between projected predicted RT and experimental RT per ECM.

## When to use

Use when RTs predicted under one chromatographic method (e.g., the METLIN SMRT reverse-phase method) must be made comparable with retention times measured under other independent experimental LC-MS methods, and when a median absolute error (in seconds) must be reported to validate the projection.

## When NOT to use

- RT prediction within the SMRT chromatographic method itself — no projection is needed; use the DLM predicted RT directly
- Candidate ranking/filtering tasks where predicted RT is used to rank accurate-mass-search candidates rather than to quantify cross-method transfer error
- ECMs for which fewer than ~50 molecules have experimental RT in both the SMRT method and the target method, where the polynomial fit cannot be reliably estimated

## Inputs

- DLM-predicted retention times for HMDB/KEGG molecules (SMRT chromatographic method time scale)
- PredRet experimental retention times for reverse-phase ECMs: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma
- Random subset of 50 molecules per ECM with RT available in both the SMRT method and the ECM

## Outputs

- Fitted four-polynomial projection function per ECM (R stats::poly regression coefficients)
- All predicted RTs transformed onto each ECM time scale
- Median absolute projection error in seconds per ECM (expected: 210 s, 9.7 s, 5.7 s, 89.4 s)

## How to apply

Load the DLM-predicted RTs for the HMDB/KEGG molecules and the PredRet experimental RTs for the reverse-phase ECMs (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma). For each ECM, randomly select 50 molecules whose experimental RT exists in both the SMRT method and the target ECM, simulating a partially identified scenario. Fit a robust least-squares projection using the R stats::poly() function with four polynomials, regressing the ECM experimental RT on the DLM-predicted RT of those 50 shared molecules; the choice of 50 molecules balances fit stability against realistic partial-identification constraints. Apply the fitted polynomial to transform all predicted RTs onto the ECM time scale. Validate by computing the median absolute difference in seconds between projected predicted RT and experimental RT, which should reproduce the reported values of 210 s (FEM_long), 9.7 s (LIFE_old), 5.7 s (RIKEN) and 89.4 s (FEM_orbitrap_plasma).

## Related tools

- **R** (Environment in which the least-squares polynomial projection regression and error computation are performed)
- **stats (R package)** (Provides poly() to construct the four-polynomial basis fitted by least squares to map DLM-predicted RT onto each ECM's RT scale)
- **PredRet database** (Source of experimental retention times for the four reverse-phase ECMs (FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma) used as projection targets)

## Examples

```
fit <- lm(ecm_rt ~ poly(pred_rt, 4), data = shared50); projected <- predict(fit, all_pred); median(abs(projected - ecm_rt_all))
```

## Evaluation signals

- Reproduced median absolute errors per ECM match the reported values: 210 s (FEM_long), 9.7 s (LIFE_old), 5.7 s (RIKEN), 89.4 s (FEM_orbitrap_plasma)
- Each ECM projection is fitted on exactly 50 randomly selected molecules with RT present in both the SMRT method and the ECM
- Fitted model is a least-squares regression on a four-polynomial basis built with stats::poly(), minimizing error between DLM-predicted RT and ECM experimental RT
- Projected RT values are on the same numeric scale as the ECM experimental RT (positive, within the ECM RT range)
- Absolute differences between projected predicted RT and experimental RT are computed per molecule for every molecule before taking the median

## Limitations

- Transfer error varies drastically across ECMs (5.7 s for RIKEN vs 210 s for FEM_long), so a single projection quality cannot be assumed for new ECMs
- Projection relies on ~50 molecules identified in both chromatographic methods; accuracy degrades if shared-identified molecules are scarce or unrepresentative
- Only the four PredRet reverse-phase ECMs listed were validated; extension to other chromatographic methods or modes is untested

## Evidence

- [abstract] an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] We analyzed the model's predictive ability along with the ability to project predicted RT onto other CMs: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [results] Predicted RT was projected onto experimental chromatographic methods via a robust least-squares polynomial regression (R stats poly function, four polynomials): [evidence span withheld — anchor status: not_found]
