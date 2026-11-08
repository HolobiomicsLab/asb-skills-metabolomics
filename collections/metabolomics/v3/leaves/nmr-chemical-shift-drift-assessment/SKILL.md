---
name: nmr-chemical-shift-drift-assessment
description: Use when a series of 1D-NMR spectra (e.g. a Daphnia magna extract series)
  shows chemical-shift drift between spectra and you need to quantify misalignment
  against a user-selected reference within a chosen spectral region before and after
  segmental alignment.
when_to_use_negative:
- The spectra series is already aligned or shows no chemical-shift drift relative
  to the chosen reference, so pre-alignment correlations would not be in the low (<0.3)
  regime this assessment expects.
- The input is 2D-NMR or tracer/GC-MS isotopomer data rather than a 1D-NMR spectra
  series for multivariate pre-processing.
- No spectrum in the series has sufficient signal-to-noise to serve as a trustworthy
  alignment reference for the selected region.
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the segmental alignment algorithm used to remove chemical-shift drift
    relative to the reference spectrum in the selected region
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Python toolbox underpinning MetaboLabPy's numerical computation for the spectra
    series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: pandas
  role: Python toolbox used with MetaboLabPy for tabulating correlation coefficients
    into alignment_correlations.csv
  repo: https://github.com/ludwigc/MetaboLabPy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:f44f36deb940858e279c00d923e87b033ff829137704b7ab12b43d7a5a538a88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# nmr-chemical-shift-drift-assessment

## Summary

Quantifies chemical-shift drift in a series of 1D-NMR spectra by computing Pearson correlation coefficients of each spectrum against a high-SNR reference spectrum before and after MetaboLabPy segmental alignment, with the drift-removal outcome judged by a rise from generally <0.3 to >0.95.

## When to use

Use when a series of 1D-NMR spectra (e.g. a Daphnia magna extract series) shows chemical-shift drift between spectra and you need to quantify misalignment against a user-selected reference within a chosen spectral region before and after segmental alignment.

## When NOT to use

- The spectra series is already aligned or shows no chemical-shift drift relative to the chosen reference, so pre-alignment correlations would not be in the low (<0.3) regime this assessment expects.
- The input is 2D-NMR or tracer/GC-MS isotopomer data rather than a 1D-NMR spectra series for multivariate pre-processing.
- No spectrum in the series has sufficient signal-to-noise to serve as a trustworthy alignment reference for the selected region.

## Inputs

- Series of 1D-NMR spectra (e.g. Daphnia magna extract spectra) loadable in MetaboLabPy 0.9.66
- User-selected spectral alignment region across the spectra
- Alignment reference spectrum (the spectrum with the highest signal-to-noise ratio)

## Outputs

- alignment_correlations.csv tabulating pre- and post-alignment Pearson correlation coefficients of each spectrum vs. the reference
- Aligned 1D-NMR spectra series with drift removed relative to the reference

## How to apply

Load the 1D-NMR spectra series into MetaboLabPy 0.9.66 (Python with numpy/scipy/pandas backend) and select the spectral region across the spectra that exhibits chemical-shift drift. Choose the spectrum with the highest signal-to-noise ratio as the alignment reference. Run MetaboLabPy's segmental alignment algorithm on the selected region to remove chemical-shift drift relative to the reference. Before alignment, compute Pearson correlation coefficients between each spectrum and the reference; after alignment, recompute the same coefficients. Tabulate both sets of coefficients and write them to alignment_correlations.csv; the assessment succeeds if pre-alignment correlations are generally below 0.3 and post-alignment correlations exceed 0.95, matching the reported segmental alignment outcome.

## Related tools

- **MetaboLabPy 0.9.66** (Provides the segmental alignment algorithm used to remove chemical-shift drift relative to the reference spectrum in the selected region) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Python toolbox underpinning MetaboLabPy's numerical computation for the spectra series) — https://github.com/ludwigc/MetaboLabPy
- **pandas** (Python toolbox used with MetaboLabPy for tabulating correlation coefficients into alignment_correlations.csv) — https://github.com/ludwigc/MetaboLabPy

## Examples

```
metabolabpy --help
```

## Evaluation signals

- Pre-alignment Pearson correlation coefficients between each spectrum and the reference spectrum in the selected segment are generally below 0.3.
- Post-alignment Pearson correlation coefficients against the reference exceed 0.95.
- alignment_correlations.csv contains both pre- and post-alignment coefficient sets for every spectrum in the series.
- The reference spectrum used for correlation is the spectrum with the highest signal-to-noise ratio in the series.
- Correlations are computed within the same user-selected spectral region before and after segmental alignment.

## Limitations

- The success criterion (pre-alignment <0.3 rising to >0.95) is demonstrated on a single dataset (Daphnia magna extract 1D-NMR spectra) and may not generalize to other samples or drift magnitudes.
- Pearson correlation against a single reference spectrum is a coarse drift metric; small or region-dependent shifts may not move correlations across the 0.3/0.95 thresholds.
- Segmental alignment is one step in MetaboLabPy's broader pre-processing workflow (automated phase correction, spectral scaling, variance stabilisation), and results depend on prior processing choices.

## Evidence

- [results] reported alignment outcome: [evidence span withheld — anchor status: not_found]
- [abstract] alignment step in workflow: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [readme] tool scope: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
