---
name: gam-injection-order-drift-modeling
description: Use when an untargeted metabolomics data matrix (samples × peaks) has
  an injection order vector but no batch label, and you must decide which frequency-domain
  components carry run-order (batch) drift before removal.
when_to_use_negative:
- When a batch label is available and the original WaveICA (F-test on ICA components,
  P < 0.05) can be applied instead
- When the input is not a peak intensity matrix ordered by injection order (e.g. raw
  mzXML files or already-corrected data)
- When run-order/injection drift is absent and only between-batch offsets need removal
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3343
tools:
- name: mgcv
  role: Fits the thin plate regression spline GAM of each independent component on
    injection order, selecting smoothing parameters by generalized cross-validation
- name: ica
  role: Provides the independent components that are tested against injection order
    by the GAM
- name: waveslim
  role: Computes the MODWT (Haar) decomposition whose wavelet components feed the
    ICA + GAM screening
- name: WaveICA_2.0
  role: R package implementing the full MODWT → ICA → GAM → inverse MODWT pipeline
    including the injection-order GAM step
  repo: https://github.com/dengkuistat/WaveICA_2.0
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s11306-021-01839-7
    title: 'WaveICA 2.0: a novel batch effect removal method for untargeted metabolomics
      data without using batch information'
  source_content_hash: sha256:ddc9db59e54d0280e8738e17b777f0ea22fd449bd2dcaba2ec687b08a3df4230
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/dengkuistat/WaveICA_2.0
  tools:
  - R
  - WaveICA_2.0
  - ica
  - mgcv
  - waveslim
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3343
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# gam-injection-order-drift-modeling

## Summary

Fits a generalized additive model (GAM) with a thin plate regression spline of each ICA-derived independent component (y) on injection order (x), as used in WaveICA 2.0 to flag batch/run-order drift in untargeted metabolomics peak intensity matrices without batch labels.

## When to use

Use when an untargeted metabolomics data matrix (samples × peaks) has an injection order vector but no batch label, and you must decide which frequency-domain components carry run-order (batch) drift before removal.

## When NOT to use

- When a batch label is available and the original WaveICA (F-test on ICA components, P < 0.05) can be applied instead
- When the input is not a peak intensity matrix ordered by injection order (e.g. raw mzXML files or already-corrected data)
- When run-order/injection drift is absent and only between-batch offsets need removal

## Inputs

- Peak intensity data matrix (samples × metabolite peaks) from XCMS preprocessing of mzXML files
- Injection order vector (sample acquisition order)
- Independent components from ICA applied to each MODWT wavelet component

## Outputs

- Adjusted R-squared value per independent component from the injection-order GAM
- Binary flag/selection of components with adjusted R² > 0.1 as batch-effect-related
- Set of independent components retained for inverse MODWT reconstruction

## How to apply

First decompose the peak intensity matrix with MODWT (Haar wavelet, Mallat pyramid algorithm, waveslim) and apply ICA to each wavelet component to extract candidate batch-effect-related independent components. For each independent component, fit a GAM in R's mgcv package modeling the component (y) on injection order (x) using a thin plate regression spline, with smoothness parameters selected by generalized cross-validation. Flag the component as batch-effect-related when its adjusted R-squared exceeds the cutoff of 0.1, then exclude flagged components before inverse MODWT reconstruction. The rationale is that components truly driven by instrument/batch drift show strong nonlinear association with injection order, whereas biological signal does not.

## Related tools

- **mgcv** (Fits the thin plate regression spline GAM of each independent component on injection order, selecting smoothing parameters by generalized cross-validation)
- **ica** (Provides the independent components that are tested against injection order by the GAM)
- **waveslim** (Computes the MODWT (Haar) decomposition whose wavelet components feed the ICA + GAM screening)
- **WaveICA_2.0** (R package implementing the full MODWT → ICA → GAM → inverse MODWT pipeline including the injection-order GAM step) — https://github.com/dengkuistat/WaveICA_2.0

## Examples

```
library(mgcv); adj_r2 <- summary(gam(y ~ s(x), method = "GCV.Cp", data = ic_df))$adj.r.squared  # ic_df: one ICA component (y) vs injection order (x)
```

## Evaluation signals

- Each GAM should report an adjusted R-squared; components with adjusted R² > 0.1 must be removed as batch-effect-related
- After removal and inverse MODWT reconstruction, Pearson correlation coefficients between any two QCSs should approach 1
- Corrected intensities should no longer show intensity drift over injection order (e.g. peak M122T352 drift disappears)
- On multi-batch data, PCA score plots should show QCSs and subject samples gathered together, comparable to the original WaveICA method
- Cumulative RSD curves should show reduced batch-effect variance relative to the uncorrected data

## Limitations

- The adjusted R² cutoff of 0.1 is dataset-specific to this study and was not tuned per-dataset in the paper
- In the three-batch comparison, WaveICA (with batch labels) achieved a slightly shorter average distance in QCSs than WaveICA 2.0, so label-free drift modeling can be marginally less aggressive at QCS gathering
- The paper's discussion section provides no explicit limitations or software version details for WaveICA 2.0

## Evidence

- [methods] the generalized additive model (GAM) is used to examine the nonlinear associations between the independent component: "the generalized additive model (GAM) is used to examine the nonlinear associations between the independent component"
- [methods] components with adjusted R2 > 0.1 are batch-related: [evidence span withheld — anchor status: not_found]
- [results] injection order is the only input: "the WaveICA 2.0 method does not use batch information and only use injection order information for batch effects removal"
- [readme] README method description: "WaveICA 2.0 method is an improved version of WaveICA method to remove inter-batch and intra-batch effects for untargeted metabolomics data without using batch information."
