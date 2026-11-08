---
name: weighted-least-squares-calibration
description: 'Use when converting MRM peak abundances to absolute concentrations for
  targeted metabolites: calibration standards of known concentration (e.g. 14 standards
  spanning 0.025–500 mol/mL) are available and regression coefficients per transition
  are needed.'
when_to_use_negative:
- Compounds were run without standards — use constant-abundance mode instead of fitting
  a standard curve
- The goal is relative comparison of features without absolute quantitation, so no
  calibration standards of known concentration exist
- Only two usable calibration points exist for an ion chromatogram — apply the two-point
  straight-line fallback rather than a full weighted fit
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3914
tools:
- name: MRMQuant
  role: Performs the least-squares standard-curve regression (constant/linear/quadratic,
    weighted by 1, 1/x, 1/x²) on calibration abundances within the absolute-quantitation
    workflow
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime in which MRMQuant's least-squares fit and regression plotting (with
    uncheckable points) are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:aace1a80c0b419b3059200df87b1142ec28839039f84349c420cff08be25a356
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
  - http://edamontology.org/topic_3914
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# weighted-least-squares-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs standard curves for absolute quantitation in targeted metabolomics by least-squares regression of computed peak abundances against known calibrant concentrations, selecting among constant, linear, or quadratic models with optional 1/x or 1/x² weighting, as implemented in MRMQuant (MATLAB).

## When to use

Use when converting MRM peak abundances to absolute concentrations for targeted metabolites: calibration standards of known concentration (e.g. 14 standards spanning 0.025–500 mol/mL) are available and regression coefficients per transition are needed.

## When NOT to use

- Compounds were run without standards — use constant-abundance mode instead of fitting a standard curve
- The goal is relative comparison of features without absolute quantitation, so no calibration standards of known concentration exist
- Only two usable calibration points exist for an ion chromatogram — apply the two-point straight-line fallback rather than a full weighted fit

## Inputs

- Calibration-point peak abundances per ion chromatogram from standard samples (e.g. 14 standards spanning 0.025–500 mol/mL)
- Targeted-compound/transition assignments from an MRM method file
- User selection of regression function (constant, linear, quadratic) and weighting (1, 1/x, 1/x²)

## Outputs

- Fitted regression coefficients per transition (slope, intercept, and higher-order terms as applicable)
- Curve type and weighting used per ion chromatogram
- Standard-curve regression plot with per-point checkboxes for user exclusion

## How to apply

For each ion chromatogram, load the calibration-point peak abundances and targeted-compound assignments from the sample and method files, then fit abundances against given concentrations using a user-specified constant, linear, or quadratic function with a least-squares fit in MRMQuant (MATLAB), choosing unweighted, 1/x, or 1/x² regression weights. Automatically exclude saturated calibration points from the regression since their abundances are unreliable, and let the user skip additional points by unchecking boxes in the regression plot when they do not fit the curve. Apply the special-case fallbacks: use a two-point straight line when only two usable points exist, and use constant-abundance mode for compounds run without standards. Return the fitted regression coefficients (slope, intercept, and higher-order terms as applicable) per transition, together with the curve type and weighting used, for downstream concentration calculation.

## Related tools

- **MRMQuant** (Performs the least-squares standard-curve regression (constant/linear/quadratic, weighted by 1, 1/x, 1/x²) on calibration abundances within the absolute-quantitation workflow) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime in which MRMQuant's least-squares fit and regression plotting (with uncheckable points) are implemented)

## Evaluation signals

- Each ion chromatogram has a recorded curve type (constant, linear, or quadratic) and weighting (none, 1/x, 1/x²) consistent with the user's specification
- Saturated calibration points are absent from the fitted regression points, and any manually unchecked points are excluded from the fit
- Returned coefficients match the selected model: slope and intercept for linear; higher-order terms additionally present for quadratic
- Ion chromatograms with only two usable points fall back to a two-point straight line, and no-standard compounds fall back to constant-abundance mode
- Downstream concentrations computed via the fitted curves are plausible across the calibration range (e.g. 0.025–500 mol/mL)

## Limitations

- Saturated peaks must be excluded from regression because their abundances are unreliable
- Points that do not seem to fit the curve must be manually skipped via the regression plot checkboxes; poor fits are not corrected automatically
- For ion chromatograms with only two usable points, only a two-point straight line is possible; compounds run without standards cannot be absolutely quantitated via SC

## Evidence
