---
name: saturated-peak-exclusion
description: Use when fitting standard-curve regressions (constant/linear/quadratic)
  or QC-RLSC batch-effect corrections in MRMQuant, and some calibration points or
  BQC peaks may be saturated with unreliable abundances.
when_to_use_negative:
- Input is already a computed concentration/feature table rather than raw MRM peak
  abundances needing regression
- No calibration points or QC samples are present and no regression is being fitted
  (e.g. constant-abundance mode for compounds run without standards)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MRMQuant
  role: Performs automatic saturated-peak exclusion within standard-curve regression
    and QC-RLSC batch-effect correction
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime in which MRMQuant's least-squares regression (with saturated points
    excluded) and findpeaks-based quantitation execute
- name: LOESS
  role: LOESS curve fitted to QC-sample abundances during QC-RLSC correction, after
    saturated peaks are excluded
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:32f0c14b0c2c61ae16409d9f034af12e0ad1e3b895563da90a720540dc4496be
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
  - LOESS
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# saturated-peak-exclusion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatically removes saturated calibration/QC peaks from regression in MRMQuant before fitting standard curves or QC-RLSC batch-effect correction, because saturated abundances are unreliable and would bias least-squares fits.

## When to use

Use when fitting standard-curve regressions (constant/linear/quadratic) or QC-RLSC batch-effect corrections in MRMQuant, and some calibration points or BQC peaks may be saturated with unreliable abundances.

## When NOT to use

- Input is already a computed concentration/feature table rather than raw MRM peak abundances needing regression
- No calibration points or QC samples are present and no regression is being fitted (e.g. constant-abundance mode for compounds run without standards)

## Inputs

- MRM sample files (converted via msconvert/ProteoWizard) with calibration-point peak abundances (e.g. 14 standards spanning 0.025–500 mol/mL)
- MRM method file with targeted compound information
- BQC (batch-pooled QC) abundances across injection batches for QC-RLSC correction

## Outputs

- Fitted standard-curve regression coefficients (slope, intercept, higher-order terms) per transition, with curve type and weighting
- Corrected BQC abundances after saturated-peak-aware QC-RLSC correction

## How to apply

Load the calibration-point peak abundances and targeted-compound assignments into MRMQuant (MATLAB); saturated peaks are excluded from the regression automatically, with no abundance threshold the user must set by hand. Fit each ion chromatogram with the user-selected constant, linear, or quadratic model, optionally weighted by 1/x or 1/x², using least squares; saturated points never contribute to the fit. When an ion chromatogram has only two usable points after exclusion, fall back to a two-point straight line, and use constant-abundance mode for compounds run without standards. In QC-RLSC batch-effect correction, saturated BQC peaks are likewise excluded before the LOESS fit. After automatic exclusion, the user may still skip additional points by unchecking their boxes in the regression plot if they do not fit the curve.

## Related tools

- **MRMQuant** (Performs automatic saturated-peak exclusion within standard-curve regression and QC-RLSC batch-effect correction) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime in which MRMQuant's least-squares regression (with saturated points excluded) and findpeaks-based quantitation execute)
- **LOESS** (LOESS curve fitted to QC-sample abundances during QC-RLSC correction, after saturated peaks are excluded)

## Evaluation signals

- Saturated calibration points do not appear as contributing points in the regression plot (only manually unchecked or excluded points are skipped)
- Post-correction BQC regression for SM d18:1/C18:0 flattens from slope 0.0004 / R² = 0.3 to slope ~0.0 / R² = 1.44 × 10–4
- Ion chromatograms left with only two usable points use a two-point straight line rather than an over-fit constant/linear/quadratic model
- Returned output records curve type and weighting (1, 1/x, 1/x²) per transition alongside the fitted coefficients

## Limitations

- Ion chromatograms reduced to only two usable points after exclusion fall back to a two-point straight line, limiting curve-shape accuracy
- Automatic exclusion is not a substitute for inspection: the user may still need to uncheck points in the regression plot that do not fit the curve
- Compounds run without standards cannot use SC-based quantitation and revert to constant-abundance mode

## Evidence
