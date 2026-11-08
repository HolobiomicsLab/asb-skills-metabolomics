---
name: chromatographic-quantitation-curve-modeling
description: Use when absolute quantitation of targeted compounds from MRM data is
  needed and calibration standard samples (e.g. 14 standards spanning 0.025–500 mol/mL)
  are available to build concentration–abundance curves.
when_to_use_negative:
- Compounds were run without standards — use constant-abundance mode rather than fitting
  an SC
- Only two usable calibration points remain after excluding saturated/unchecked points
  — a two-point straight line fallback, not a user-selected quadratic model, is the
  appropriate fit
- The goal is relative quantitation without calibration (e.g. batch-effect correction
  on BQC samples), which does not require SC construction
edam_operation: http://edamontology.org/operation_3630
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MRMQuant
  role: Performs the SC regression of computed abundances against concentrations with
    selectable constant/linear/quadratic functions and 1, 1/x, or 1/x² weighting,
    including automatic exclusion of saturated points
  repo: https://github.com/kslynn128171/MRMQuant
- name: MATLAB
  role: Runtime environment in which MRMQuant's least-squares SC fit and regression
    plots are implemented (built-in functions such as findpeaks support upstream peak
    detection)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.4c02462
    title: 'MRMQuant: Automated MRM Data Quantitation for Large-Scale Targeted Metabolomics
      Analysis'
  source_content_hash: sha256:0255c1bfb8a1e5964b483d534ac70b41760f0d7f4a2f95cc4cd121cfce10fd8e
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
  edam_operation: http://edamontology.org/operation_3630
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# chromatographic-quantitation-curve-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits standard curves (SCs) for absolute quantitation of targeted MRM ion chromatograms by regressing computed peak abundances against known concentrations, with flexible regression models and weighting. Implemented in MRMQuant (MATLAB) with automatic exclusion of unreliable calibration points and fallback modes for sparse data.

## When to use

Use when absolute quantitation of targeted compounds from MRM data is needed and calibration standard samples (e.g. 14 standards spanning 0.025–500 mol/mL) are available to build concentration–abundance curves.

## When NOT to use

- Compounds were run without standards — use constant-abundance mode rather than fitting an SC
- Only two usable calibration points remain after excluding saturated/unchecked points — a two-point straight line fallback, not a user-selected quadratic model, is the appropriate fit
- The goal is relative quantitation without calibration (e.g. batch-effect correction on BQC samples), which does not require SC construction

## Inputs

- Calibration-point peak abundances for targeted ion chromatograms from MRMQuant peak detection of standard sample files (mzML or mzXML converted via msconvert in ProteoWizard)
- Method file describing targeted compounds and transitions
- Known concentrations of standard samples (e.g. 14 standards spanning 0.025–500 mol/mL)
- User-specified SC regression model (constant, linear, or quadratic) and weighting option (1, 1/x, or 1/x²)

## Outputs

- Fitted regression coefficients per transition (slope, intercept, and higher-order terms as applicable)
- Per-ion-chromatogram curve type and weighting used
- Regression plot with checkable points for visual inspection and point exclusion

## How to apply

Load calibration-point peak abundances and targeted-compound assignments prepared from the MRM sample files and method file, then fit each ion chromatogram's SC by least-squares regression of abundance against concentration. Choose a constant, linear, or quadratic regression function per ion chromatogram, optionally weighted by 1, 1/x, or 1/x². Automatically exclude saturated calibration points from the regression because their abundances are unreliable, and let the user skip additional points by unchecking boxes in the regression plot when they do not fit the curve. Apply the special-case rules: if only two usable points exist, fall back to a two-point straight line; for compounds run without standards, use constant-abundance mode instead of an SC.

## Related tools

- **MRMQuant** (Performs the SC regression of computed abundances against concentrations with selectable constant/linear/quadratic functions and 1, 1/x, or 1/x² weighting, including automatic exclusion of saturated points) — https://github.com/kslynn128171/MRMQuant
- **MATLAB** (Runtime environment in which MRMQuant's least-squares SC fit and regression plots are implemented (built-in functions such as findpeaks support upstream peak detection))

## Evaluation signals

- Each ion chromatogram's returned SC records the selected curve type (constant/linear/quadratic) and weighting (1, 1/x, or 1/x²) actually used
- Saturated calibration points are absent from the fitted regression because their abundances are unreliable
- Ion chromatograms with only two usable points are fit as two-point straight lines rather than higher-order models
- Compounds without standards are quantified in constant-abundance mode instead of via an SC
- Points the user unchecked in the regression plot are excluded from the least-squares fit

## Limitations

- Saturated calibration points yield unreliable abundances and must be automatically excluded from regression
- Some points may not fit the selected curve and require manual skipping by unchecking boxes in the regression plot
- When only two usable points remain, the fit degrades to a two-point straight line, limiting curve-shape flexibility

## Evidence
