---
name: nmr-spectra-integral-binning
description: Use when a preprocessed, window-selected NMR spectra matrix (spectra_df,
  samples x chemical shift points) is still too high-dimensional for multivariate
  statistics and you need a bucketed feature table with one integrated intensity per
  fixed ppm bin, while coarsely tolerating residual peak shifts.
when_to_use_negative:
- The input is already a reduced feature table or binned matrix rather than a point-wise
  spectra DataFrame — re-binning would compound bucketing artifacts.
- Residual peak shifts are large and systematic — equidistant binning only coarsely
  handles residual displacement; run Icoshift peak alignment (px.Icoshift()) before
  binning instead of relying on bins to fix alignment.
- High spectral resolution must be preserved for peak-level interpretation — binning
  deliberately reduces dimensionality and cannot recover per-peak information.
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Protomix
  role: Provides px.binning, which performs the equidistant binning and rectangular-method
    integration of the spectra into a bucketed matrix
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Loads the preprocessed spectra_df / ws_df and saves the resulting binned matrix
    to CSV
- name: numpy
  role: Numeric backend for the spectra arrays and per-bin integration in the Protomix
    pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:10973220161f75be6604be137557bf6cbc56e1a1ae92762b6a634a768d2f24ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nmr-spectra-integral-binning

## Summary

Equidistant spectral binning for 1H-NMR metabolomics: partition the chemical shift axis into fixed-width bins (e.g. 0.1 ppm) and integrate the spectral area under the curve within each bin using Protomix's binning function, producing a reduced-dimension bucketed matrix for multivariate analysis such as PCA.

## When to use

Use when a preprocessed, window-selected NMR spectra matrix (spectra_df, samples x chemical shift points) is still too high-dimensional for multivariate statistics and you need a bucketed feature table with one integrated intensity per fixed ppm bin, while coarsely tolerating residual peak shifts.

## When NOT to use

- The input is already a reduced feature table or binned matrix rather than a point-wise spectra DataFrame — re-binning would compound bucketing artifacts.
- Residual peak shifts are large and systematic — equidistant binning only coarsely handles residual displacement; run Icoshift peak alignment (px.Icoshift()) before binning instead of relying on bins to fix alignment.
- High spectral resolution must be preserved for peak-level interpretation — binning deliberately reduces dimensionality and cannot recover per-peak information.

## Inputs

- Window-selected 1H-NMR spectra DataFrame (spectra_df / ws_df): samples as rows, chemical shift points as columns, from the Protomix pipeline applied to the Zenodo NMRDataset (record 13467227)
- bin_size parameter (e.g. 0.1 ppm)
- integration method parameter (e.g. 'rectangular')

## Outputs

- Binned (bucketed) data matrix: samples as rows, consecutive 0.1 ppm bin positions as columns, each value a non-negative integrated intensity per bin
- binned_spectra.csv export preserving sample indices

## How to apply

Load the window-selected spectra DataFrame (ws_df) produced by the Protomix preprocessing chain (extract_fids, fourier_transform, phase_correction, internal_referencing, baseline_correction, window selection) with pandas. Call px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular') to partition the chemical shift axis into equidistant 0.1 ppm bins and integrate the area under the spectral curve per bin using the rectangular (rectangular integration) method; this bucketing also absorbs small residual peak displacements left after Icoshift peak alignment. Assemble the resulting binned matrix with samples as rows and bin positions as columns, preserving the original sample indices. The output is suitable directly for downstream multivariate analysis such as PCA, so save it as a CSV (e.g. binned_spectra.csv) once validated.

## Related tools

- **Protomix** (Provides px.binning, which performs the equidistant binning and rectangular-method integration of the spectra into a bucketed matrix) — https://github.com/mzniber/protomix
- **pandas** (Loads the preprocessed spectra_df / ws_df and saves the resulting binned matrix to CSV)
- **numpy** (Numeric backend for the spectra arrays and per-bin integration in the Protomix pipeline)

## Examples

```
bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')
```

## Evaluation signals

- binned_spectra.csv exists and is a valid CSV table.
- Columns correspond to consecutive 0.1 ppm bins spanning the selected spectral window.
- Every binned value is a non-negative integrated intensity per bin.
- Sample rows preserve the original sample indices of spectra_df (samples as rows, bins as columns).
- Dimensionality is reduced from the number of chemical shift points to the number of 0.1 ppm bins, and the matrix is suitable for downstream PCA.

## Limitations

- Equidistant binning only handles residual peak shifts coarsely; it is intended as a pre-multivariate-analysis step (e.g. before PCA), not a substitute for proper peak alignment such as Icoshift.
- The rectangular integration method approximates the area under the curve within each bin, so bin width (0.1 ppm here) controls the resolution/shift-tolerance trade-off.
- Downstream multivariate statistics such as PCA are mentioned as the intended use; other analysis goals are not covered by the cited evidence.

## Evidence

- [methods] Protomix's binning mechanism performs equidistant binning: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [methods] Binning of spectra for statistical analysis: "Protomix is a Python package designed for the preprocessing of 1H NMR metabolomics data"
- [readme] Protomix is a comprehensive Python package for NMR preprocessing: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
- [readme] Features list includes Binning for downstream analysis: [evidence span withheld — anchor status: external_doc]
