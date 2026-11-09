---
name: fid-time-domain-preprocessing
description: Use when starting from raw Bruker-formatted FID data (extract_params
  + extract_fids outputs) and you need to correct the group-delay artifact, suppress
  residual solvent, and Fourier-transform each FID into a spectrum before phase/baseline
  steps.
when_to_use_negative:
- Input is already frequency-domain spectra (spectra_df); use phase_correction, baseline_correction,
  binning, and normalization instead.
- Input FIDs have not been extracted — run Protomix's extract_params and extract_fids
  first to obtain acqus_df and fids_df.
- Data is not Bruker-formatted NMR with an acqus file providing the group-delay parameter,
  since the number of leading points to delete cannot be determined.
edam_operation: http://edamontology.org/operation_3214
edam_topics: []
tools:
- name: Protomix
  role: 'Provides the FID-stage functions: group_delay_removal, solvent_residuals_removal,
    apodization, zero_filling, fourier_transform'
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Holds acqus_df and fids_df as DataFrames and carries intermediate FID tables
    (grpd_df, zf_df) between steps
- name: numpy
  role: Numeric array backend for the FID time-point operations and Fourier transform
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:6292f267de0f288018210951ff9cc7abd367dc9ec9af9a4747deb6f97be83212
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyicoshift
  edam_operation: http://edamontology.org/operation_3214
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fid-time-domain-preprocessing

## Summary

Time-domain (FID) preprocessing chain for Bruker 1H-NMR metabolomics data using Protomix: group-delay removal, solvent residual removal, exponential apodization, zero filling, and Fourier transformation to convert raw Free Induction Decays into frequency-domain spectra.

## When to use

Use when starting from raw Bruker-formatted FID data (extract_params + extract_fids outputs) and you need to correct the group-delay artifact, suppress residual solvent, and Fourier-transform each FID into a spectrum before phase/baseline steps.

## When NOT to use

- Input is already frequency-domain spectra (spectra_df); use phase_correction, baseline_correction, binning, and normalization instead.
- Input FIDs have not been extracted — run Protomix's extract_params and extract_fids first to obtain acqus_df and fids_df.
- Data is not Bruker-formatted NMR with an acqus file providing the group-delay parameter, since the number of leading points to delete cannot be determined.

## Inputs

- fids_df: pandas DataFrame of raw Free Induction Decays from px.extract_fids (samples as rows, time points as columns)
- acqus_df: pandas DataFrame of Bruker acquisition parameters from px.extract_params (contains the DSP group-delay setting)
- Bruker-formatted spectra directory (e.g. NMRDataset.zip from Zenodo record 13467227)

## Outputs

- grpd_df: group-delay-corrected FID DataFrame, shortened per sample by the acqus-specified number of leading points
- fids/solvents from solvent_residuals_removal (solvent trace when returnSolvent=True)
- apod_df: exponentially apodized FID DataFrame (LB=5)
- zf_df: zero-filled FID DataFrame at target_points=10000
- spectra_df: frequency-domain spectra DataFrame ready for phase_correction

## How to apply

First extract acquisition parameters (px.extract_params) and raw FIDs (px.extract_fids) from the Bruker directory, yielding acqus_df and fids_df. Run px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df), which reads the group-delay ('death time') parameter from acqus and deletes exactly that many leading points from each FID to correct the first-order phase error. Then apply px.solvent_residuals_removal (optionally returnSolvent=True), px.apodization with LB=5 and apodization_type='exponential', px.zero_filling with target_points=10000, and px.fourier_transform(fid_df=zf_df, acqus_df=acqus_df) to obtain spectra_df. The rationale is that group delay is a point-deletion correction (no complex transforms needed), while exponential apodization trades resolution for signal-to-noise before zero filling to 10000 points and FFT. Each intermediate (grpd_df, apod_df, zf_df, spectra_df) is a pandas DataFrame with samples as rows and time/frequency points as columns.

## Related tools

- **Protomix** (Provides the FID-stage functions: group_delay_removal, solvent_residuals_removal, apodization, zero_filling, fourier_transform) — https://github.com/mzniber/protomix
- **pandas** (Holds acqus_df and fids_df as DataFrames and carries intermediate FID tables (grpd_df, zf_df) between steps)
- **numpy** (Numeric array backend for the FID time-point operations and Fourier transform)

## Examples

```
acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df); grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)
```

## Evaluation signals

- grpd_df.csv exists as a CSV table whose per-sample time-domain traces are shortened by exactly the acqus-specified number of leading points (file_exists / script_runs checks).
- zf_df has target_points=10000 columns after zero filling.
- apodization used exponential type with LB=5, matching the article's parameters.
- spectra_df is a valid frequency-domain table (samples as rows) that phase_correction can consume without error.
- The full 70-sample pipeline through to normalized spectra completes in under two minutes on an 8-core laptop; large deviations suggest a stage misconfiguration.

## Limitations

- Group-delay removal deletes a fixed number of leading points read from acqus, so a missing or wrong acqus DSP parameter silently corrupts the correction.
- Exponential apodization with LB=5 trades spectral resolution for sensitivity; other apodization needs require changing apodization_type.
- Zero filling is fixed at target_points=10000 in the article's workflow; other target sizes need explicit parameter changes.

## Evidence

- [methods] group_delay_removal: "grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)"
- [methods] apodization: "apod_df = px.apodization(fids, LB=5, apodization_type='exponential')"
- [methods] zero_filling: "zf_df = px.zero_filling(fid_df=apod_df, acqus_df=acqus_df, target_points=10000)"
- [methods] fourier_transform: "spectra_df = px.fourier_transform(fid_df=zf_df, acqus_df=acqus_df)"
- [methods] solvent_residuals_removal: "fids, solvents = px.solvent_residuals_removal(fid_df=grpd_df, returnSolvent=True)"
- [readme] readme features: "- **Group Delay Removal**
- **Solvent Residuals Removal**
- **Apodization**
- **Zero Filling**
- **Fourier Transform**"
