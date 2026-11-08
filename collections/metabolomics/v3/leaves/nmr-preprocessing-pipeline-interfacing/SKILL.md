---
name: nmr-preprocessing-pipeline-interfacing
description: Use when raw Bruker 1H-NMR metabolomics data (acqus parameters + FIDs,
  e.g. a Zenodo NMRDataset zip) must be converted into a sample x bin intensity table
  for multivariate analysis such as PCA.
when_to_use_negative:
- Input is already a binned/feature table or spectra matrix that only needs statistical
  analysis — binning would redundantly re-bucket it
- Data are not vendor Bruker 1H-NMR FIDs (Protomix's extraction functions target Bruker
  acqus/FID records)
- Only peak alignment of already-processed spectra is required — use pyicoshift (px.Icoshift)
  directly rather than the full extraction-through-binning chain
tools:
- name: Protomix
  role: Provides every preprocessing function invoked in the chain (extract_params,
    extract_fids, group_delay_removal, solvent_residuals_removal, apodization, zero_filling,
    fourier_transform, phase_correction, internal_referencing, baseline_correction,
    Icoshift, binning)
  repo: https://github.com/mzniber/protomix
- name: pyicoshift
  role: Python rewrite of the icoshift MATLAB algorithm wrapped by Protomix's px.Icoshift()
    for peak alignment before binning
  repo: https://github.com/sekro/pyicoshift
- name: pandas
  role: DataFrame carrier for spectra/binned matrices and for saving the bucketed
    matrix to CSV
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:eb17df8e576fe967c0ffb1e5ef23d1272723915da984283eec6f18922e7d07f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  - pyicoshift
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# nmr-preprocessing-pipeline-interfacing

## Summary

Orchestrates Protomix's function chain to turn raw Bruker 1H-NMR FIDs into a reduced-dimension bucketed feature matrix: extracting acquisition parameters, FIDs, and applying FID/spectrum-level corrections through equidistant binning (px.binning, bin_size=0.1, rectangular integration) ready for PCA.

## When to use

Use when raw Bruker 1H-NMR metabolomics data (acqus parameters + FIDs, e.g. a Zenodo NMRDataset zip) must be converted into a sample x bin intensity table for multivariate analysis such as PCA.

## When NOT to use

- Input is already a binned/feature table or spectra matrix that only needs statistical analysis — binning would redundantly re-bucket it
- Data are not vendor Bruker 1H-NMR FIDs (Protomix's extraction functions target Bruker acqus/FID records)
- Only peak alignment of already-processed spectra is required — use pyicoshift (px.Icoshift) directly rather than the full extraction-through-binning chain

## Inputs

- Directory of Bruker 1H-NMR records containing acqus acquisition-parameter files and FID data (e.g. Zenodo record 13467227 NMRDataset.zip)
- Intermediate pandas DataFrames passed step-to-step: acqus_df, fids_df, fid_df, spectra_df, ws_df

## Outputs

- acqus_df (extracted acquisition parameters)
- fids_df (extracted Free Induction Decays)
- Preprocessed spectra DataFrame (phase-corrected, referenced, airPLS baseline-corrected, window-selected)
- Binned data matrix (samples as rows, consecutive 0.1 ppm bins as columns, e.g. binned_spectra.csv)

## How to apply

Run the Protomix chain in order: px.extract_params(root_directory=...) to read Bruker acquisition parameters into acqus_df, px.extract_fids(...) for FIDs, then group_delay_removal, solvent_residuals_removal (returnSolvent=True), apodization (LB=5, exponential), zero_filling (target_points=10000), fourier_transform, phase_correction, internal_referencing, and baseline_correction (method='airpls'). Align residual peak shifts with px.Icoshift(), apply window selection to restrict to the spectral window of interest, then reduce dimensionality via px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular'), which integrates area under the curve within each 0.1 ppm bin. Keep samples as rows and bin positions as columns, preserving the sample index from spectra_df, and save the result (e.g. binned_spectra.csv) for downstream PCA. Install via pip (python3 -m pip install protomix); pandas DataFrames are the interchange format between every step.

## Related tools

- **Protomix** (Provides every preprocessing function invoked in the chain (extract_params, extract_fids, group_delay_removal, solvent_residuals_removal, apodization, zero_filling, fourier_transform, phase_correction, internal_referencing, baseline_correction, Icoshift, binning)) — https://github.com/mzniber/protomix
- **pyicoshift** (Python rewrite of the icoshift MATLAB algorithm wrapped by Protomix's px.Icoshift() for peak alignment before binning) — https://github.com/sekro/pyicoshift
- **pandas** (DataFrame carrier for spectra/binned matrices and for saving the bucketed matrix to CSV)

## Examples

```
bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')
```

## Evaluation signals

- Every chained call returns a pandas DataFrame that feeds the next function without key/index loss (e.g. fids_df built with acqus_df)
- binned_spectra.csv exists, parses as valid CSV, and its columns correspond to consecutive 0.1 ppm bins spanning the selected spectral window
- Every binned value is a non-negative integrated intensity per bin (rectangular area-under-the-curve integration)
- Row count and sample index of the binned matrix match the window-selected spectra_df samples
- Solvent removal returns both cleaned FIDs and the modeled solvent when returnSolvent=True

## Limitations

- Rectangular equidistant binning at bin_size=0.1 ppm is a coarse dimensionality reduction; peak shifts are handled by prior icoshift alignment, so misaligned peaks can straddle bin boundaries
- Protomix is written for Python 3 / PyPI; the article README lists features without documenting per-function failure modes, and no changelog is present ('No changelog found.')
- pyicoshift's multi-threading support for optimal shift finding is labeled experimental, affecting alignment throughput/robustness

## Evidence

- [readme] Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data"
- [methods] python3 -m pip install protomix: "python3 -m pip install protomix"
- [methods] px.extract_fids call: "fids_df = px.extract_fids(root_directory=main_directory, acqus_df=acqus_df)"
- [methods] baseline correction method: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [methods] binning call: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [abstract] package purpose from abstract: "we have developed Protomix—a Python package designed for metabolomics research"
