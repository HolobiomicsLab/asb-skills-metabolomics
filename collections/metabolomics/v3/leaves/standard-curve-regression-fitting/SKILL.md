---
name: standard-curve-regression-fitting
description: Use when converting detected MRM peak abundances from standard samples
  (e.g. 14 standards spanning 0.025–500 mol/mL) into calibration curves for absolute
  concentration calculation.
when_to_use_negative:
- The goal is batch-effect correction or BQC-based normalization rather than absolute
  quantitation via standard curves
- Peak detection, deconvolution, and abundance computation have not yet been performed
  on the MRM sample files
- Only relative quantitation using internal standards is desired and no standard samples
  were acquired
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MRMQuant
  role: Performs the standard-curve least-squares regression (constant/linear/quadratic,
    weighted by 1, 1/x, 1/x²), auto-excludes saturated points, and supports manual
    point skipping via the regression plot
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment providing the least-squares fitting (and the built-in
    findpeaks used upstream) underlying MRMQuant's SC generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:60c8b4fbed78fff967bfc5d6f950c3d82a0eb05cb66aee33fe2cf3624487bae3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/kslynn128171/MRMQuant
  repo_url: https://github.com/kslynn128171/MRMQuant
  tools:
  - MATLAB
  - MRMQuant
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# standard-curve-regression-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits standard curves (SCs) for absolute quantitation in MRMQuant by regressing computed peak abundances against calibration concentrations using constant, linear, or quadratic least-squares functions with optional 1, 1/x, or 1/x² weights, in MATLAB.

## When to use

Use when converting detected MRM peak abundances from standard samples (e.g. 14 standards spanning 0.025–500 mol/mL) into calibration curves for absolute concentration calculation.

## When NOT to use

- The goal is batch-effect correction or BQC-based normalization rather than absolute quantitation via standard curves
- Peak detection, deconvolution, and abundance computation have not yet been performed on the MRM sample files
- Only relative quantitation using internal standards is desired and no standard samples were acquired

## Inputs

- Peak abundances of calibration points from standard samples (e.g. 14 standards spanning 0.025–500 mol/mL)
- Targeted-compound/transition assignments from the MRMQuant method file
- User-selected regression function (constant, linear, quadratic) and weighting (1, 1/x, 1/x²)

## Outputs

- Fitted regression coefficients (slope, intercept, higher-order terms) per ion chromatogram/transition
- Curve type and weighting used for each standard curve
- Regression plot for visual inspection of fit and manual point exclusion

## How to apply

Load calibration-point abundances and targeted-compound assignments for the standard samples, then fit a user-selected regression function (constant, linear, or quadratic) per ion chromatogram, optionally weighted by 1/x, 1/x², or left unweighted, using MRMQuant's least-squares fit in MATLAB. Automatically exclude saturated calibration points from the regression because their abundances are unreliable, and skip any points the user unchecks in the regression plot when they do not fit the curve. When only two usable points remain for an ion chromatogram, fall back to a two-point straight line; for compounds run without standards, use constant-abundance mode. Return the fitted regression coefficients (slope, intercept, and higher-order terms as applicable) per transition, together with the curve type and weighting used, for downstream concentration calculation.

## Related tools

- **MRMQuant** (Performs the standard-curve least-squares regression (constant/linear/quadratic, weighted by 1, 1/x, 1/x²), auto-excludes saturated points, and supports manual point skipping via the regression plot) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment providing the least-squares fitting (and the built-in findpeaks used upstream) underlying MRMQuant's SC generation)

## Evaluation signals

- Each transition returns fitted coefficients (slope, intercept, higher-order terms) together with the curve type and weighting used
- Saturated calibration points are absent from the fit; user-unchecked points are excluded from the regression
- Ion chromatograms with only two usable points fall back to a two-point straight line rather than a failed fit
- Compounds run without standards fall back to constant-abundance mode instead of an SC regression
- Visual inspection of the regression plot shows calibration points (e.g. 0.025–500 mol/mL range) conforming to the fitted curve

## Limitations

- Saturated calibration points yield unreliable abundances and must be excluded from the regression
- Points that do not seem to fit the curve must be manually skipped via unchecking in the regression plot
- The two-point fallback (straight line) and constant-abundance mode are coarse approximations used only when full SCs are unavailable

## Evidence
