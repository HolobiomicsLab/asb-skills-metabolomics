---
name: bucketed-data-matrix-construction
description: Use when preprocessed, window-selected spectra (samples × chemical-shift
  points) are too high-dimensional for multivariate statistics and you need integrated
  per-bin intensities at fixed bin widths (e.g. 0.1 ppm) before PCA.
when_to_use_negative:
- Input is already a bucketed/feature table (e.g. previous bin_df output) rather than
  a full-resolution spectra matrix — re-binning would compound integration.
- Spectra have not yet passed peak alignment (pyicoshift Icoshift step) and window
  selection, since residual peak shifts and regions outside the selected spectral
  window should be handled before binning.
- Variable-width or adaptive binning is required — the Protomix binning function described
  here performs equidistant binning only.
edam_topics:
- http://edamontology.org/topic_3321
tools:
- name: Protomix
  role: Provides the binning function (px.binning) that performs equidistant binning
    and rectangular-method integration of the area under the spectral curve within
    each bin
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Loads the preprocessed spectra_df and handles the bucketed data matrix (sample
    indices as rows, bin positions as columns) and CSV export
- name: numpy
  role: Numeric support for the spectra matrices handled during binning
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:1856d7c177cc507ffe2015ebbfa36b716c6c911dbf273a62bbeb07a85f92d6d5
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
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bucketed-data-matrix-construction

## Summary

Reduces high-dimensional 1H-NMR spectra into a bucketed (binned) data matrix by partitioning the chemical shift axis into equidistant bins (e.g. 0.1 ppm) and integrating the area under the spectral curve within each bin via Protomix's binning function, producing a compact sample-by-bin matrix suitable for downstream multivariate analysis such as PCA.

## When to use

Use when preprocessed, window-selected spectra (samples × chemical-shift points) are too high-dimensional for multivariate statistics and you need integrated per-bin intensities at fixed bin widths (e.g. 0.1 ppm) before PCA.

## When NOT to use

- Input is already a bucketed/feature table (e.g. previous bin_df output) rather than a full-resolution spectra matrix — re-binning would compound integration.
- Spectra have not yet passed peak alignment (pyicoshift Icoshift step) and window selection, since residual peak shifts and regions outside the selected spectral window should be handled before binning.
- Variable-width or adaptive binning is required — the Protomix binning function described here performs equidistant binning only.

## Inputs

- spectra_df: preprocessed, window-selected 1H-NMR spectra as a pandas DataFrame (samples as rows, chemical-shift points as columns), e.g. from the Zenodo NMRDataset pipeline output (binned_spectra.csv workflow input)
- bin_size: bin width in ppm (0.1 ppm in the published workflow)
- method: integration method for the area under the curve within each bin ('rectangular')

## Outputs

- bin_df: bucketed data matrix (samples as rows, 0.1 ppm bin positions as columns) of non-negative integrated intensities per bin, preserving sample indices
- binned_spectra.csv: saved CSV table of the binned matrix

## How to apply

Load the preprocessed, window-selected spectra matrix (spectra_df) — the output of the Protomix pipeline (fourier_transform, phase_correction, internal_referencing, baseline_correction, region/window selection) — as a pandas DataFrame indexed by sample. Partition the chemical shift axis into equidistant bins of bin_size=0.1 ppm and call Protomix's binning function with method='rectangular' so that, for each bin, the area under the spectral curve is integrated over the bin interval using the rectangular method. The function assembles a bucketed data matrix with samples as rows and bin positions as columns, preserving sample indices. Save the result (e.g. to binned_spectra.csv) and validate that columns correspond to consecutive 0.1 ppm bins spanning the selected spectral window and that every value is a non-negative integrated intensity per bin.

## Related tools

- **Protomix** (Provides the binning function (px.binning) that performs equidistant binning and rectangular-method integration of the area under the spectral curve within each bin) — https://github.com/mzniber/protomix
- **pandas** (Loads the preprocessed spectra_df and handles the bucketed data matrix (sample indices as rows, bin positions as columns) and CSV export)
- **numpy** (Numeric support for the spectra matrices handled during binning)

## Examples

```
bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')
```

## Evaluation signals

- binned_spectra.csv exists and parses as a valid CSV table
- Columns correspond to consecutive 0.1 ppm bins spanning the selected spectral window
- Every cell is a non-negative integrated intensity per bin
- Sample rows/indices from spectra_df are preserved in bin_df (no samples dropped or reordered)
- The number of columns in bin_df matches the bin count implied by the selected window width divided by bin_size=0.1 ppm

## Limitations

- Equidistant binning may split or merge peaks depending on bin boundaries; residual peak shifts not corrected by prior Icoshift alignment will smear intensity across neighboring bins.
- The rectangular integration method is an approximation of the area under the curve within each bin interval.
- Binning discards within-bin spectral detail, so it is a dimensionality-reduction step for multivariate statistics, not a substitute for full-resolution spectral analysis.

## Evidence

- [methods] Binning of spectra for statistical analysis: [evidence span withheld — anchor status: not_found]
- [readme] Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
