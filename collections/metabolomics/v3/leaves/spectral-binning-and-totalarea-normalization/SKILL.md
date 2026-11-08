---
name: spectral-binning-and-totalarea-normalization
description: Use when aligned, window-selected spectra (spectra_df rows indexed by
  ppm) must be converted to a fixed feature table for statistical analysis or classification,
  e.g. after Icoshift alignment in a Protomix 1H-NMR metabolomics pipeline.
when_to_use_negative:
- Input is already a binned or feature-level table (e.g. a metabolite concentration
  matrix) — binning only applies to raw ppm-grid spectra.
- Downstream analysis requires full-resolution spectra or line-shape detail — 0.1
  ppm rectangular binning discards within-bin spectral information.
- Peak alignment (Icoshift) has not yet been performed — binning misaligned spectra
  will smear peaks across adjacent bins.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_0593
tools:
- name: Protomix
  role: Provides the binning (bin_size=0.1, method='rectangular') and normalization
    (method='TotalArea') functions that turn aligned spectra into the final feature
    table
  repo: https://github.com/mzniber/protomix
- name: pyicoshift
  role: Upstream Icoshift peak alignment whose output quality determines whether 0.1
    ppm binning correctly pools peak intensities
  repo: https://github.com/sekro/pyicoshift
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:c18876111bddc1dca284e5f39b347fee3a2bab83ca3e7199136fde459445fdf1
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# spectral-binning-and-totalarea-normalization

## Summary

Reduce preprocessed 1H-NMR spectra (after window_selection and peak alignment) into a compact binned feature table and scale intensities with TotalArea normalization, using Protomix's binning and normalization functions on a 70-sample prostate cancer/BPH urine dataset.

## When to use

Use when aligned, window-selected spectra (spectra_df rows indexed by ppm) must be converted to a fixed feature table for statistical analysis or classification, e.g. after Icoshift alignment in a Protomix 1H-NMR metabolomics pipeline.

## When NOT to use

- Input is already a binned or feature-level table (e.g. a metabolite concentration matrix) — binning only applies to raw ppm-grid spectra.
- Downstream analysis requires full-resolution spectra or line-shape detail — 0.1 ppm rectangular binning discards within-bin spectral information.
- Peak alignment (Icoshift) has not yet been performed — binning misaligned spectra will smear peaks across adjacent bins.

## Inputs

- window-selected spectra DataFrame (rows = 70 urine samples, columns = ppm grid after Icoshift alignment and window_selection)
- Protomix px module imported in Python 3 with numpy/pandas available

## Outputs

- binned spectra DataFrame at 0.1 ppm bin size (bin_df)
- TotalArea-normalized spectra table (e.g. preprocessed_spectra_normalized.csv)

## How to apply

Run Protomix's rectangular binning on the window-selected spectra DataFrame with bin_size=0.1 ppm and method='rectangular' (px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')), which pools intensities into 0.1 ppm-wide buckets and absorbs residual small chemical-shift drift after Icoshift alignment. Then apply px.normalization with method='TotalArea' so each spectrum is scaled by its total integrated area, compensating for concentration and overall signal-strength differences between urine samples. Write the normalized table (e.g. preprocessed_spectra_normalized.csv) as the final preprocessed feature matrix ready for downstream PCA or classification. Correctness is judged by checking the table shape matches sample count, bin columns fall within the selected ppm window, and each row's total area is constant after normalization.

## Related tools

- **Protomix** (Provides the binning (bin_size=0.1, method='rectangular') and normalization (method='TotalArea') functions that turn aligned spectra into the final feature table) — https://github.com/mzniber/protomix
- **pyicoshift** (Upstream Icoshift peak alignment whose output quality determines whether 0.1 ppm binning correctly pools peak intensities) — https://github.com/sekro/pyicoshift

## Examples

```
bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular'); normalized_df = px.normalization(binned_df=bin_df, method='TotalArea')
```

## Evaluation signals

- The binned DataFrame bin_df has one row per sample (70) and columns spanning the selected ppm window at 0.1 ppm resolution
- After px.normalization with method='TotalArea', every spectrum's summed intensity is equal (or constant) across samples within numerical tolerance
- No negative intensities remain in the normalized table (negative_values_zeroing was applied upstream)
- The written preprocessed_spectra_normalized.csv parses cleanly with pandas and preserves the sample index/order of the input spectra DataFrame
- The full preprocessing pipeline (including binning/normalization) completes in under two minutes on an 8-core laptop for the 70-sample dataset

## Limitations

- Rectangular binning at bin_size=0.1 ppm is a coarse resolution choice; narrower peaks or closely spaced metabolite signals may be merged into a single bin
- TotalArea normalization is sensitive to regions that were removed (4.5–6.1 ppm water/urea region) — removed regions reduce total area and change the scaling factor
- Binning must run on window-selected, aligned spectra; running it before window_selection or alignment degrades feature quality

## Evidence

- [methods] Binning at 0.1 ppm with rectangular method precedes normalization: "bin_df = px.binning(spectra_df=ws_df, bin_size=0.1, method='rectangular')"
- [methods] Binning is intended as a preprocessing step feeding statistical analysis: [evidence span withheld — anchor status: not_found]
- [readme] README confirms binning and normalization are supported Protomix features: "- **Binning**
- **Normalization**"
- [other] Task card documents the TotalArea normalization and CSV output: [evidence span withheld — anchor status: not_found]
