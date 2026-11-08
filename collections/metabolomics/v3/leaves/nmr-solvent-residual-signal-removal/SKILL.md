---
name: nmr-solvent-residual-signal-removal
description: Use when you hold a pandas DataFrame of FIDs (grpd_df) that has already
  been through group-delay correction, and dominant residual solvent decays must be
  removed before apodization and Fourier transform in a 1H-NMR metabolomics pipeline.
when_to_use_negative:
- Input is already frequency-domain spectra (spectra_df); this skill operates on time-domain
  FIDs after group-delay removal and before apodization/zero_filling/fourier_transform.
- FIDs have not yet been group-delay corrected (still fids_df straight from px.extract_fids),
  since the first points of the FID would distort the smoother's solvent estimate.
- The goal is peak alignment or binning of spectra (px.Icoshift, px.binning), which
  are later-stage steps in the Protomix workflow, not solvent removal.
tools:
- name: Protomix
  role: Provides solvent_residuals_removal, which fits and subtracts the residual
    solvent component from FIDs using discrete penalized least squares with second-order
    differences (Whittaker smoother).
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Holds the FID DataFrames in/out of the function and writes solvent_corrected_fids.csv
    and estimated_solvents.csv.
- name: numpy
  role: Numeric backend for the penalized least-squares smoothing underlying the solvent
    estimate (imported in Protomix methods).
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:dee48b4af1b1e899775c582815c9b3128e814b24be20793edf4996b07499a3e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmr-solvent-residual-signal-removal

## Summary

Removes residual solvent (e.g., water) signals from group-delay-corrected 1H-NMR FID data using Protomix's solvent_residuals_removal, which fits the solvent component via discrete penalized least squares with second-order differences (Whitaker/Whittaker smoother, Whittaker 1922; Eilers 2003) and subtracts it from each FID.

## When to use

Use when you hold a pandas DataFrame of FIDs (grpd_df) that has already been through group-delay correction, and dominant residual solvent decays must be removed before apodization and Fourier transform in a 1H-NMR metabolomics pipeline.

## When NOT to use

- Input is already frequency-domain spectra (spectra_df); this skill operates on time-domain FIDs after group-delay removal and before apodization/zero_filling/fourier_transform.
- FIDs have not yet been group-delay corrected (still fids_df straight from px.extract_fids), since the first points of the FID would distort the smoother's solvent estimate.
- The goal is peak alignment or binning of spectra (px.Icoshift, px.binning), which are later-stage steps in the Protomix workflow, not solvent removal.

## Inputs

- grpd_df: pandas DataFrame of group-delay-corrected FIDs (rows = samples, columns = time points), produced by px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)
- acqus_df: Bruker acquisition parameters DataFrame (upstream context, from px.extract_params)
- returnSolvent=True option to also retrieve the estimated solvent components

## Outputs

- fids: pandas DataFrame of solvent-corrected FIDs (same rows/columns as grpd_df), saved as solvent_corrected_fids.csv
- solvents: pandas DataFrame of estimated smooth solvent components, saved as estimated_solvents.csv

## How to apply

Run this step after group_delay_removal (px.group_delay_removal on fids_df with acqus_df yields grpd_df). Call px.solvent_residuals_removal on the grpd_df FID DataFrame with returnSolvent=True, so the Whittaker smoother—discrete penalized least squares with a second-order difference penalty—estimates a smooth baseline-like solvent decay per FID and returns both the corrected FIDs and the estimated solvent components. Store the corrected FID table as solvent_corrected_fids.csv and the solvent estimates as estimated_solvents.csv with pandas, then feed the corrected FIDs into apodization (px.apodization, e.g., LB=5 exponential). The rationale is that the strong solvent residual decay otherwise dominates the FID and propagates into the resulting spectra. The article provides no quantitative threshold, so acceptance rests on shape preservation and smoothness of the estimated solvent component.

## Related tools

- **Protomix** (Provides solvent_residuals_removal, which fits and subtracts the residual solvent component from FIDs using discrete penalized least squares with second-order differences (Whittaker smoother).) — https://github.com/mzniber/protomix
- **pandas** (Holds the FID DataFrames in/out of the function and writes solvent_corrected_fids.csv and estimated_solvents.csv.)
- **numpy** (Numeric backend for the penalized least-squares smoothing underlying the solvent estimate (imported in Protomix methods).)

## Examples

```
fids, solvents = px.solvent_residuals_removal(fid_df=grpd_df, returnSolvent=True)
```

## Evaluation signals

- Both solvent_corrected_fids.csv and estimated_solvents.csv exist after the run.
- Output DataFrames retain the same number of rows (samples) and columns (time points) as the input grpd_df.
- Each estimated solvent component is smooth and tracks the dominant residual solvent decay of its FID.
- The corrected FID preserves the remaining non-solvent signal rather than flattening it entirely.
- The step is positioned in the pipeline after group_delay_removal and before apodization (px.apodization with LB=5, exponential).

## Limitations

- The article provides no quantitative acceptance threshold for solvent removal quality; validation is limited to a shape-preservation and smoothness check.
- Removal quality depends on the upstream group-delay correction (px.group_delay_removal on fids_df with acqus_df); uncorrected first points can bias the smoother.
- Method is tied to discrete penalized least squares with second-order differences (Whittaker/Eilers), so very sharp or overlapping solvent features may not be tracked by the smooth component.

## Evidence

- [methods] solvent_residuals_removal call: "fids, solvents = px.solvent_residuals_removal(fid_df=grpd_df, returnSolvent=True)"
- [methods] upstream group-delay step: "grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)"
- [methods] Whittaker smoother basis: [evidence span withheld — anchor status: not_found]
- [readme] package scope from README: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
