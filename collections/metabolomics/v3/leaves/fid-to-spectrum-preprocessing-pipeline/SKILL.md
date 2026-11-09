---
name: fid-to-spectrum-preprocessing-pipeline
description: Use when starting from raw Bruker NMR acquisition directories (acqus
  files + FIDs) and a cleaned, phase-corrected, baseline-corrected spectra table (samples
  × ppm) is needed for metabolomics analysis.
when_to_use_negative:
- Input is already a Fourier-transformed spectra table — skip the FID-side steps (group
  delay removal, apodization, zero filling) and start at phase correction or later
- Data are not 1D 1H-NMR metabolomics spectra (e.g. 2D NMR or non-Bruker formats)
  — the pipeline is designed for Bruker 1H-NMR preprocessing
- Only peak alignment is needed — use px.Icoshift (pyicoshift) directly rather than
  rerunning the FID-to-spectrum chain
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: Protomix
  role: Python package providing the chained FID-to-spectrum preprocessing functions
    (extract_params, extract_fids, group_delay_removal, solvent_residuals_removal,
    apodization, zero_filling, fourier_transform, phase_correction, internal_referencing,
    baseline_correction)
  repo: https://github.com/mzniber/protomix
- name: numpy
  role: Array backend underlying the DataFrame-based signal processing steps
- name: pandas
  role: Holds FIDs and spectra as DataFrames (rows = samples, columns = time points
    or ppm) passed between pipeline steps
- name: pyicoshift
  role: Downstream peak alignment (Icoshift) applied after the spectra table is produced;
    Protomix exposes it via px.Icoshift()
  repo: https://github.com/sekro/pyicoshift
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:1fe328fd1534994e6edd5646f16a6b05281769604e245949765a637f9e3eceea
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# fid-to-spectrum-preprocessing-pipeline

## Summary

A chained Protomix workflow that converts raw Bruker 1H-NMR FIDs into analysis-ready spectra: extracting acqus parameters and FIDs, removing group delay and residual solvent, apodizing (LB=5 exponential), zero-filling (target_points=10000), Fourier transforming, phase correcting, internally referencing, and baseline correcting (airPLS).

## When to use

Use when starting from raw Bruker NMR acquisition directories (acqus files + FIDs) and a cleaned, phase-corrected, baseline-corrected spectra table (samples × ppm) is needed for metabolomics analysis.

## When NOT to use

- Input is already a Fourier-transformed spectra table — skip the FID-side steps (group delay removal, apodization, zero filling) and start at phase correction or later
- Data are not 1D 1H-NMR metabolomics spectra (e.g. 2D NMR or non-Bruker formats) — the pipeline is designed for Bruker 1H-NMR preprocessing
- Only peak alignment is needed — use px.Icoshift (pyicoshift) directly rather than rerunning the FID-to-spectrum chain

## Inputs

- Bruker NMR root directory containing acqus acquisition parameter files and FID data
- acqus_df DataFrame produced by px.extract_params
- fids_df DataFrame of Free Induction Decays (rows = samples, columns = time points)
- Protomix NMRDataset from Zenodo record 13467227 (NMRDataset.zip)

## Outputs

- spectra_df / ph_df / ir_df / bc_df: pandas DataFrames of spectra (rows = samples, columns = chemical shift in ppm)
- ph_df: phase-corrected spectra table (e.g. phase_corrected_spectra.csv with fitted_phase_angles.csv)
- bc_df: baseline-corrected spectra ready for alignment, binning, and normalization
- solvents DataFrame (optional) from px.solvent_residuals_removal when returnSolvent=True

## How to apply

Install Protomix (pip install protomix) and load the Zenodo NMRDataset (record 13467227) into a main_directory of Bruker data. Chain the calls in order: px.extract_params to read acqus acquisition parameters, px.extract_fids to build the FIDs DataFrame, px.group_delay_removal (using acqus_df), px.solvent_residuals_removal (optionally returning the solvent), px.apodization with LB=5 and apodization_type='exponential', px.zero_filling with target_points=10000, px.fourier_transform (with acqus_df) to produce the spectra_df (rows = samples, columns = ppm), px.phase_correction to optimize per-spectrum angles by minimizing dispersive contributions, px.internal_referencing, and px.baseline_correction with method='airpls'. Each step's DataFrame output feeds the next, so keep the acqus_df consistent throughout; save intermediate tables as CSV. Downstream, follow with peak alignment (px.Icoshift), binning (bin_size=0.1, method='rectangular'), and normalization before statistical analysis.

## Related tools

- **Protomix** (Python package providing the chained FID-to-spectrum preprocessing functions (extract_params, extract_fids, group_delay_removal, solvent_residuals_removal, apodization, zero_filling, fourier_transform, phase_correction, internal_referencing, baseline_correction)) — https://github.com/mzniber/protomix
- **numpy** (Array backend underlying the DataFrame-based signal processing steps)
- **pandas** (Holds FIDs and spectra as DataFrames (rows = samples, columns = time points or ppm) passed between pipeline steps)
- **pyicoshift** (Downstream peak alignment (Icoshift) applied after the spectra table is produced; Protomix exposes it via px.Icoshift()) — https://github.com/sekro/pyicoshift

## Examples

```
import protomix as px; acqus_df = px.extract_params(root_directory=main_directory); fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df); grpd_df = px.group_delay_removal(fid_df=fids_df, acqus_df=acqus_df)
```

## Evaluation signals

- After fourier_transform, spectra_df has one row per sample and ppm-axis columns consistent with acqus_df and the zero-filling target_points=10000
- After phase_correction, peaks appear in pure absorption mode with a flat baseline when plotted (e.g. with px.plot titled 'NMR Spectrum after phase correction', xlabel 'Chemical Shift (ppm)'); fitted per-spectrum angles satisfy Φcorrection = Φerror
- Each intermediate DataFrame preserves the sample row structure of the previous step (same number of rows, no samples dropped)
- Phase-corrected file (phase_corrected_spectra.csv) has the same row/column structure as the input spectra table, with fitted angles recorded in fitted_phase_angles.csv
- After baseline_correction with airPLS, spectra show minimal baseline offset and no dispersive/negative peak contributions before binning (bin_size=0.1, rectangular)

## Limitations

- Apodization, zero-filling target points, and baseline method are user-chosen parameters (LB=5, exponential, target_points=10000, airPLS) — different choices change the spectra and must be reported
- Phase correction optimizes an RMS-of-positive-peaks objective per spectrum; difficult spectra may require manual verification of absorption lineshape
- The package targets 1D 1H-NMR metabolomics data (e.g. the prostate cancer vs benign prostatic hyperplasia tutorial dataset); it is not a general-purpose NMR processing tool for other experiments

## Evidence

- [methods] filter step chain in methods: "ph_df = px.phase_correction(spectra_df=spectra_df)"
- [methods] zero filling with target points: "zf_df = px.zero_filling(fid_df=apod_df, acqus_df=acqus_df, target_points=10000)"
- [methods] apodization parameters: "apod_df = px.apodization(fids, LB=5, apodization_type='exponential')"
- [methods] baseline correction method: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [readme] README feature list: [evidence span withheld — anchor status: external_doc]
