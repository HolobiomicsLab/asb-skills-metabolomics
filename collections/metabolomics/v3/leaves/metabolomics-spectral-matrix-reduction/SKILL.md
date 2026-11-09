---
name: metabolomics-spectral-matrix-reduction
description: Use when you have preprocessed, window-selected 1H-NMR spectra (spectra_df
  with samples as rows and chemical shifts as columns) and need to reduce dimensionality
  for multivariate analysis such as PCA.
when_to_use_negative:
- Input is already a reduced feature table or bucketed matrix — re-binning would distort
  integrated intensities
- 'Raw FIDs or unpreprocessed spectra: binning assumes prior steps (phase_correction,
  internal_referencing, baseline_correction via airPLS, window_selection) have been
  applied'
- When peak-level resolution must be preserved (e.g. targeted quantification of specific
  metabolite peaks) — equidistant binning averages across bins and loses per-peak
  detail
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Protomix
  role: Provides the px.binning function that performs equidistant binning with rectangular-method
    integration of spectral area under the curve per bin
  repo: https://github.com/mzniber/protomix
- name: pandas
  role: Loads the preprocessed spectra_df, assembles and preserves sample indices
    in the binned matrix, and writes binned_spectra.csv
- name: numpy
  role: Numeric backbone for the spectral arrays over which per-bin integration is
    computed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:222c048fd7dd4233ae97d510f87d860284d63f0195d5dfc5e4762f1e3a6196da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-spectral-matrix-reduction

## Summary

Reduces high-dimensionality 1H-NMR spectra to a bucketed feature matrix by partitioning the chemical shift axis into equidistant bins (e.g. 0.1 ppm via Protomix's px.binning) and integrating the spectral area under the curve within each bin, yielding a compact sample-by-bin matrix suitable for downstream multivariate statistics such as PCA.

## When to use

Use when you have preprocessed, window-selected 1H-NMR spectra (spectra_df with samples as rows and chemical shifts as columns) and need to reduce dimensionality for multivariate analysis such as PCA.

## When NOT to use

- Input is already a reduced feature table or bucketed matrix — re-binning would distort integrated intensities
- Raw FIDs or unpreprocessed spectra: binning assumes prior steps (phase_correction, internal_referencing, baseline_correction via airPLS, window_selection) have been applied
- When peak-level resolution must be preserved (e.g. targeted quantification of specific metabolite peaks) — equidistant binning averages across bins and loses per-peak detail

## Inputs

- Preprocessed, window-selected spectra matrix (spectra_df / ws_df) with samples as rows and chemical shift (ppm) columns, e.g. from Protomix's window_selection step
- bin_size parameter (e.g. 0.1 ppm)
- integration method option (e.g. 'rectangular')

## Outputs

- Binned (bucketed) data matrix with samples as rows and consecutive 0.1 ppm bin positions as columns, preserving sample indices (e.g. binned_spectra.csv)

## How to apply

Load the preprocessed, window-selected spectra matrix (spectra_df) — e.g. the output of Protomix's window_selection step on the Zenodo NMRDataset pipeline — with pandas. Partition the chemical shift axis into equidistant bins of bin_size=0.1 ppm using px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular'), which integrates the area under the curve within each bin interval by the rectangular method; bins often contain one or more NMR peaks, and this bucketing absorbs residual peak shifts not fully corrected by prior Icoshift alignment. Assemble the resulting bucketed data matrix with samples as rows and bin positions as columns, preserving the original sample indices, and save it (e.g. binned_spectra.csv) for downstream multivariate analysis. Choose bin_size as the key decision point: smaller bins preserve resolution but retain more shift sensitivity, larger bins improve robustness at the cost of spectral detail.

## Related tools

- **Protomix** (Provides the px.binning function that performs equidistant binning with rectangular-method integration of spectral area under the curve per bin) — https://github.com/mzniber/protomix
- **pandas** (Loads the preprocessed spectra_df, assembles and preserves sample indices in the binned matrix, and writes binned_spectra.csv)
- **numpy** (Numeric backbone for the spectral arrays over which per-bin integration is computed)

## Examples

```
import protomix as px; bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')
```

## Evaluation signals

- binned_spectra.csv exists and parses as a valid CSV table
- Columns correspond to consecutive equidistant 0.1 ppm bins spanning the selected spectral window
- Every entry is a non-negative integrated intensity per bin
- Sample row indices of the binned matrix match the input spectra_df sample indices
- The binned matrix has far fewer columns than the original spectra matrix (successful dimensionality reduction for PCA)

## Limitations

- Equidistant binning integrates broad regions and can mix signals from adjacent peaks within a single bin; bin_size=0.1 ppm trades resolution for shift robustness
- Binning is a downstream step that assumes residual peak shifts were already handled by prior alignment (Icoshift) and window selection; it does not itself correct shifts
- Rectangular integration is an approximation of the area under the curve and depends on the bin_size/method parameters

## Evidence

- [methods] Binning of spectra for statistical analysis: [evidence span withheld — anchor status: not_found]
- [methods] binning invocation: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [abstract] Protomix preprocessing scope: "Protomix offers a set of automated, efficient, and user-friendly signal-preprocessing steps, tailored to streamline and enhance the preprocessing phase in metabolomics studies."
- [readme] Binning listed as a feature: [evidence span withheld — anchor status: external_doc]
