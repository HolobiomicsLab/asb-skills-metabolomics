---
name: rt-log-transformation-and-robust-iqr-scaling
description: Use when fitting or applying the meta-learned GP projection between chromatographic
  methods (e.g., PredRet predicted-vs-experimental RT pairs), where RT inputs must
  first be log-transformed and robust-scaled before GP meta-training.
when_to_use_negative:
- Training the RT prediction regressors themselves (DNN, XGBoost, LightGBM, CatBoost)
  on SMRT, where a quantile transformation is applied to RTs before fitting instead
  of log + IQR scaling
- When RTs are already on the target chromatographic method's experimental scale and
  no cross-method projection is needed
- When the molecule set includes non-retained compounds that have not yet been filtered
  out, since they distort the learned scaling statistics
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: cmmrt
  role: Implements the projection pipeline in which log-transformation and robust
    median/IQR scaling are applied to PredRet RT pairs before GP meta-training
  repo: https://github.com/constantino-garcia/cmmrt
- name: Python
  role: Language in which the cmmrt package and its transformation/GP projection code
    are implemented and run via the Makefile targets
  repo: https://github.com/constantino-garcia/cmmrt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-022-00613-8
    title: Probabilistic metabolite annotation using retention time prediction and
      meta-learned projections
  source_content_hash: sha256:c451cff8309662eae35db883a2d3c1cfb6e30e7e3cff2e8f46e3ca37ead10d4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/constantino-garcia/cmmrt
  tools:
  - Python
  - cmmrt
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# RT log-transformation and robust IQR scaling

## Summary

A preprocessing step of the Bayesian meta-learning RT projection module: retention times are compressed with log(1+x) and then robustly scaled using the median and 0.741*IQR so that the Gaussian Process can model residuals on a stable, roughly Gaussian scale.

## When to use

Use when fitting or applying the meta-learned GP projection between chromatographic methods (e.g., PredRet predicted-vs-experimental RT pairs), where RT inputs must first be log-transformed and robust-scaled before GP meta-training.

## When NOT to use

- Training the RT prediction regressors themselves (DNN, XGBoost, LightGBM, CatBoost) on SMRT, where a quantile transformation is applied to RTs before fitting instead of log + IQR scaling
- When RTs are already on the target chromatographic method's experimental scale and no cross-method projection is needed
- When the molecule set includes non-retained compounds that have not yet been filtered out, since they distort the learned scaling statistics

## Inputs

- PredRet predicted-vs-experimental RT pairs per chromatographic method (FEM long: 342, FEM orbitrap plasma: 133, LIFE old: 148, RIKEN: 271, remainder: 2418 molecules), with non-retained molecules excluded
- Experimental RTs of ~10 stratified-sampled identified molecules for the target chromatographic method

## Outputs

- Log-transformed, robustly scaled RT inputs in [0,1] and outcomes in [-1,1] ready for GP meta-training
- Learned median and 0.741*IQR scaling statistics (fit on predicted RTs) used to invert transformations after GP prediction

## How to apply

First exclude non-retained molecules from the PredRet meta-task datasets, then apply the log transform log(1+x) to all retention times (both predicted and experimental). Learn the scaling statistics from the predicted RTs only: robustly scale the transformed inputs to [0,1] and the outcomes to [-1,1] using the median and 0.741*IQR — the 0.741 constant is chosen so that sigma corresponds to ~0.741*IQR under normality, giving the GP a well-calibrated residual scale. Fit these scalings once per projection task and apply the same mapping to the ~10 stratified-sampled identified molecules used as target-task training points. After computing the GP posterior predictive mean and 95% credible intervals, invert the log and scaling transforms to return projections on the original RT scale.

## Related tools

- **cmmrt** (Implements the projection pipeline in which log-transformation and robust median/IQR scaling are applied to PredRet RT pairs before GP meta-training) — https://github.com/constantino-garcia/cmmrt
- **Python** (Language in which the cmmrt package and its transformation/GP projection code are implemented and run via the Makefile targets) — https://github.com/constantino-garcia/cmmrt

## Examples

```
make train_projections
```

## Evaluation signals

- Scaling statistics are learned from the predicted RTs (not the experimental RTs), and inputs land in [0,1] while outcomes land in [-1,1]
- The 0.741*IQR constant is used (not the raw IQR), so the scaled residual sigma is consistent with a normality assumption
- Non-retained molecules are excluded before computing the median and IQR statistics
- After GP prediction, inverting log(1+x) and the robust scaling recovers RTs on the original experimental scale of the target chromatographic method
- The meta-learned GP fitted on transformed data must achieve Delta-L_avg > 0 versus a non-meta-learned reference GP with constant mean and squared exponential kernel

## Limitations

- The 0.741*IQR constant assumes approximate normality of the transformed RT residuals; heavy-tailed residuals break this calibration
- Scaling parameters are learned exclusively from predicted RTs, so a poorly calibrated RT predictor would shift the scaling
- This transformation scheme is specific to the GP projection module; the RT prediction models in the same paper use a quantile transformation instead

## Evidence

- [methods] Log-transformation of RTs: "RTs are transformed to log space using"
- [methods] Robust scaling with median and IQR: "data is further scaled using robust statistics."
- [methods] Quantile transformation of RTs: "quantile transformation was applied to RTs before fitting."
- [readme] projections_to_different_cm.ipynb notebook description: "map experimental retention times to the retention times predicted with a DNN (or viceversa). This is done by training a meta-learned GP prior on a small subset of known molecules."
- [readme] Makefile projection rule: "Meta-trains a Gaussian Process (GP) for computing projections between CMs using the PredRet database for a few epochs."
