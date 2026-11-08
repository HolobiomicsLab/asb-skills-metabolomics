---
name: cross-dataset-molecule-matching
description: Use when predicted RTs from one chromatographic method must be made comparable
  to RTs measured under independent experimental reverse-phase methods, and per-method
  projection errors (e.g. median absolute error in seconds) must be quantified.
when_to_use_negative:
- The target chromatographic method already shares enough identified molecules for
  direct RT alignment without a regression projection
- The molecules of interest have no experimental RT overlap between the SMRT method
  and the target ECM, leaving no anchor set to fit the polynomial
- The goal is RT prediction itself rather than projecting existing predicted RTs onto
  another method's time scale
tools:
- name: R
  role: Implementation language used to fit the four-polynomial least-squares projection
    and compute median absolute errors per ECM
- name: stats (R package)
  role: Provides the poly() function used to build the four-polynomial basis fitted
    to minimize error between DLM-predicted RT and experimental RT
- name: PredRet database
  role: Source of experimental retention times for the four reverse-phase ECMs onto
    which predicted RTs are projected
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:95d748eddc24fa2bc7f6cb6729f7e9b9a8bd519d8c380bd03d22f96a379e9d10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PredRet database
  - R
  - stats (R package)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cross-dataset-molecule-matching

## Summary

Projects deep-learning-predicted retention times from the METLIN SMRT chromatographic method onto independent experimental reverse-phase ECMs (e.g. PredRet datasets) via robust least-squares polynomial regression, making predicted RTs directly comparable to RTs measured under other methods.

## When to use

Use when predicted RTs from one chromatographic method must be made comparable to RTs measured under independent experimental reverse-phase methods, and per-method projection errors (e.g. median absolute error in seconds) must be quantified.

## When NOT to use

- The target chromatographic method already shares enough identified molecules for direct RT alignment without a regression projection
- The molecules of interest have no experimental RT overlap between the SMRT method and the target ECM, leaving no anchor set to fit the polynomial
- The goal is RT prediction itself rather than projecting existing predicted RTs onto another method's time scale

## Inputs

- DLM-predicted retention times for HMDB KEGG molecules in the METLIN SMRT chromatographic method
- Experimental retention times for each target ECM (PredRet database: FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma)
- Mapping of molecules with experimental RT available in both the SMRT method and the target ECM (random subset of 50 per ECM)

## Outputs

- A fitted four-polynomial projection function per ECM (R stats::poly least-squares fit)
- All predicted RTs transformed onto each ECM's experimental time scale
- Per-molecule absolute differences between projected predicted RT and experimental RT
- Median absolute projection error in seconds per ECM (e.g. 210 s FEM_long, 9.7 s LIFE_old, 5.7 s RIKEN, 89.4 s FEM_orbitrap_plasma)

## How to apply

Load the DLM-predicted RTs for the HMDB KEGG molecules together with the experimental RTs of each target ECM (e.g. FEM_long, LIFE_old, RIKEN, FEM_orbitrap_plasma from the PredRet database). For each ECM, randomly select 50 molecules whose experimental RT is available in both the SMRT method and the ECM, simulating a partially identified scenario. For each ECM, fit a projection via robust least-squares regression of a four-polynomial function using R's stats::poly(), minimizing the error between DLM-predicted RT and experimental RT on those 50 shared molecules; the regression is what maps the SMRT time scale onto each ECM's time scale. Apply each fitted polynomial to transform all predicted RTs onto the corresponding ECM scale, then compute absolute differences between projected predicted RT and experimental RT for every molecule and derive the median absolute error in seconds per ECM. The random 50-molecule subset both anchors the fit and keeps the evaluation realistic for cases where only a few standards are identifiable in the new method.

## Related tools

- **R** (Implementation language used to fit the four-polynomial least-squares projection and compute median absolute errors per ECM)
- **stats (R package)** (Provides the poly() function used to build the four-polynomial basis fitted to minimize error between DLM-predicted RT and experimental RT)
- **PredRet database** (Source of experimental retention times for the four reverse-phase ECMs onto which predicted RTs are projected)

## Examples

```
fit <- lm(rt_exp ~ poly(rt_pred, degree = 4), data = shared50); projected <- predict(fit, newdata = list(rt_pred = all_pred_rt)); median(abs(projected - rt_exp_all))
```

## Evaluation signals

- Reproduced median absolute projection errors should match the reported values: 210 s (FEM_long), 9.7 s (LIFE_old), 5.7 s (RIKEN), and 89.4 s (FEM_orbitrap_plasma)
- Each ECM's projection is fitted on exactly 50 randomly selected molecules with experimental RT present in both the SMRT method and the ECM
- The fitted model is a four-degree polynomial basis from R's stats::poly(), fitted by least squares against experimental RT
- After projection, per-molecule absolute differences between projected predicted RT and experimental RT are non-negative and yield a finite median in seconds
- Projected RTs must fall on the target ECM's time scale (monotone mapping consistent with the ECM's RT range), not the SMRT scale

## Limitations

- Projection quality varies strongly across methods: median absolute error spans 5.7 s (RIKEN) to 210 s (FEM_long), so accuracy depends heavily on the similarity of the target ECM to the SMRT method
- The fit depends on a random subset of 50 molecules per ECM; a different random draw can change the fitted polynomial and resulting errors
- Only molecules with experimental RT in both the SMRT method and the ECM can anchor the projection, so coverage is limited by partially identified scenarios

## Evidence

- [abstract] SMRT dataset covers 80,038 molecules: "Here we introduce the METLIN small molecule retention time (SMRT) dataset, an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules."
- [intro] model evaluated for projection onto other chromatographic methods: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
- [intro] lack of experimental RT datasets limited large-scale RT prediction: "the lack of experimental RT datasets has hampered the design of ML methods for large-scale RT prediction"
