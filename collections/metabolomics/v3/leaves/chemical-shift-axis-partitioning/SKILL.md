---
name: chemical-shift-axis-partitioning
description: Use when preprocessed, window-selected 1H-NMR spectra (spectra_df) still
  contain residual peak shifts and must be reduced to a lower-dimensional bucketed
  feature table before multivariate statistical analysis.
when_to_use_negative:
- Input is already a binned/feature table rather than a full-resolution spectra_df
  matrix
- The spectra have not yet been preprocessed and window-selected (e.g. no phase correction,
  internal referencing, or baseline correction has been applied)
- Peak-shift correction is needed without dimensionality reduction — use Icoshift
  peak alignment instead of binning
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: Protomix
  role: Provides the px.binning function that performs equidistant binning of NMR
    spectra by integrating area under the curve per bin (e.g. bin_size=0.1, method='rectangular')
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Loads the window-selected spectra matrix and assembles/saves the bucketed
    data matrix (binned_spectra.csv)
- name: numpy
  role: Numerical support for the spectra matrix operations
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:4b4c58f8e56c4a0eccf0d047a7eab52926c44966b71c7f59c89c2748105744de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# chemical-shift-axis-partitioning

## Summary

Reduces the high dimensionality of 1H-NMR spectra by partitioning the chemical shift axis into equidistant bins (e.g. 0.1 ppm) and integrating the area under the curve within each bin via Protomix's binning function, yielding a bucketed data matrix for downstream multivariate analysis such as PCA.

## When to use

Use when preprocessed, window-selected 1H-NMR spectra (spectra_df) still contain residual peak shifts and must be reduced to a lower-dimensional bucketed feature table before multivariate statistical analysis.

## When NOT to use

- Input is already a binned/feature table rather than a full-resolution spectra_df matrix
- The spectra have not yet been preprocessed and window-selected (e.g. no phase correction, internal referencing, or baseline correction has been applied)
- Peak-shift correction is needed without dimensionality reduction — use Icoshift peak alignment instead of binning

## Inputs

- Preprocessed, window-selected 1H-NMR spectra matrix (spectra_df / ws_df) with samples as rows and chemical shift points as columns
- bin_size parameter (e.g. 0.1 ppm)
- integration method parameter (e.g. 'rectangular')

## Outputs

- Bucketed data matrix (bin_df) with samples as rows and consecutive bin positions (e.g. 0.1 ppm bins) as columns, preserving sample indices
- CSV file of the binned matrix (e.g. binned_spectra.csv)

## How to apply

Load the preprocessed, window-selected spectra matrix (spectra_df) with pandas and partition the chemical shift axis into equidistant bins of bin_size=0.1 ppm, so each bin typically contains one or more NMR peaks. For each bin, integrate the area under the spectral curve within the bin interval using the rectangular method via Protomix's binning function. Assemble the resulting bucketed data matrix with samples as rows and bin positions as columns, preserving sample indices, then save it (e.g. to binned_spectra.csv). Binning acts as a dimensionality-reduction step after preprocessing (phase correction, internal referencing, baseline correction, window selection) and complements peak alignment with Icoshift, which handles residual peak shifts prior to statistical analysis such as PCA.

## Related tools

- **Protomix** (Provides the px.binning function that performs equidistant binning of NMR spectra by integrating area under the curve per bin (e.g. bin_size=0.1, method='rectangular')) — https://github.com/mzniber/protomix
- **pandas** (Loads the window-selected spectra matrix and assembles/saves the bucketed data matrix (binned_spectra.csv))
- **numpy** (Numerical support for the spectra matrix operations)

## Examples

```
bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')
```

## Evaluation signals

- binned_spectra.csv exists and is a valid CSV table
- Columns correspond to consecutive 0.1 ppm bins spanning the selected spectral window
- Each binned value is a non-negative integrated intensity per bin
- Sample indices/rows are preserved between spectra_df and the bucketed matrix
- The number of columns equals the spectral window width divided by bin_size (0.1 ppm), confirming equidistant partitioning

## Limitations

- Rectangular integration is a simple area approximation within each bin; finer integration methods or smaller bin sizes may be needed for sharper peaks
- Binning does not by itself correct residual peak shifts; peak alignment via Icoshift (pyicoshift) is the complementary step in the Protomix workflow

## Evidence

- [other] Protomix's binning mechanism performs equidistant binning of NMR spectra by splitting the data into small regions (often containing one or more NMR peaks) and integrating the area under the curve within each bin, yielding a reduced-dimension bucketed data matrix suitable for downstream multivariate analysis such as PCA.: [evidence span withheld — anchor status: not_found]
- [methods] bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular'): "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [methods] Binning of spectra for statistical analysis: [evidence span withheld — anchor status: not_found]
- [readme] README feature list including Binning: [evidence span withheld — anchor status: external_doc]
