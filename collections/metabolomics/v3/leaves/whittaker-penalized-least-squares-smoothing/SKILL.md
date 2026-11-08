---
name: whittaker-penalized-least-squares-smoothing
description: Use when a group-delay-corrected FID dataframe from 1H-NMR metabolomics
  data still contains a dominant residual solvent decay that must be separated from
  the sample signal before apodization and Fourier transform.
when_to_use_negative:
- The data are already frequency-domain spectra — baseline issues there are handled
  by px.baseline_correction(spectra_df, method='airpls'), not by this FID-domain solvent
  removal.
- The FIDs have not yet been group-delay corrected, so the solvent decay cannot be
  modeled as a smooth component of grpd_df.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Protomix
  role: provides solvent_residuals_removal, which applies the Whittaker penalized-least-squares
    smoother to FID data
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: holds the FID and solvent DataFrames and writes them to CSV outputs
- name: numpy
  role: numerical backend for the discrete penalized least squares computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:33abc8f9719add261505deb87b056f7bd082173ac76cfebee674532c8a7aa2c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# whittaker-penalized-least-squares-smoothing

## Summary

Estimates and removes residual solvent (water) signals from 1H-NMR FID data by fitting a smooth component via discrete penalized least squares with second-order differences (Whittaker smoother), as implemented in Protomix's solvent_residuals_removal function.

## When to use

Use when a group-delay-corrected FID dataframe from 1H-NMR metabolomics data still contains a dominant residual solvent decay that must be separated from the sample signal before apodization and Fourier transform.

## When NOT to use

- The data are already frequency-domain spectra — baseline issues there are handled by px.baseline_correction(spectra_df, method='airpls'), not by this FID-domain solvent removal.
- The FIDs have not yet been group-delay corrected, so the solvent decay cannot be modeled as a smooth component of grpd_df.

## Inputs

- pandas DataFrame of group-delay-corrected FIDs (grpd_df) with samples as rows and time points as columns, e.g. from px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)

## Outputs

- pandas DataFrame of solvent-corrected FIDs (same shape as the input grpd_df)
- pandas DataFrame of estimated solvent components (returned when returnSolvent=True)

## How to apply

Load the group-delay-corrected FID dataframe (grpd_df, one row per sample, one column per time point) produced by px.group_delay_removal on fids_df extracted from Bruker data. Call px.solvent_residuals_removal(fid_df=grpd_df, returnSolvent=True) so that both the corrected FIDs and the estimated solvent components are returned; the underlying method estimates the solvent component by discrete penalized least squares with a second-order difference penalty (Whitaker's smoother, Whittaker 1922; Eilers 2003), yielding a smooth baseline-like curve that tracks the dominant solvent decay. Store the corrected FID table and the estimated solvent table (e.g. solvent_corrected_fids.csv and estimated_solvents.csv) with pandas. Rationale: the residual water signal dominates the FID early decay and would otherwise contaminate downstream apodization, zero-filling, and Fourier transform steps; the second-order penalty enforces smoothness so the metabolite signal, which varies faster, is preserved. The correction is the third step in Protomix's FID pipeline, between group_delay_removal and apodization (LB=5, exponential).

## Related tools

- **Protomix** (provides solvent_residuals_removal, which applies the Whittaker penalized-least-squares smoother to FID data) — https://github.com/mzniber/protomix
- **pandas** (holds the FID and solvent DataFrames and writes them to CSV outputs)
- **numpy** (numerical backend for the discrete penalized least squares computation)

## Examples

```
fids, solvents = px.solvent_residuals_removal(fid_df=grpd_df, returnSolvent=True)
```

## Evaluation signals

- Output corrected-FID and solvent DataFrames have the same number of rows (samples) and columns (time points) as the input grpd_df.
- The estimated solvent component is smooth and captures the dominant residual solvent decay (qualitative smoothness check; the article provides no quantitative threshold).
- The corrected FID preserves the remaining (non-solvent) signal for downstream apodization and zero_filling steps.
- Both output tables (e.g. solvent_corrected_fids.csv and estimated_solvents.csv) are written and non-empty.
- The function call returns two objects (fids, solvents) when returnSolvent=True.

## Limitations

- No quantitative acceptance threshold is given in the text; validation is a shape-preservation and smoothness check only.
- The method estimates the solvent component as a smooth baseline-like decay, so fast-decaying genuine metabolite signal at the FID start could in principle be affected.

## Evidence

- [methods] solvent_residuals_removal applies Whittaker smoother: [evidence span withheld — anchor status: not_found]
- [methods] example invocation from methods: "fids, solvents = px.solvent_residuals_removal(fid_df=grpd_df, returnSolvent=True)"
- [readme] README lists Solvent Residuals Removal as a pipeline feature: [evidence span withheld — anchor status: external_doc]
