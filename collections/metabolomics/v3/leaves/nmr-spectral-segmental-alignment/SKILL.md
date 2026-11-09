---
name: nmr-spectral-segmental-alignment
description: Use when a series of 1D-NMR spectra (e.g. a metabolomics sample set)
  shows chemical-shift drift within a spectral region and peaks must be aligned to
  a reference spectrum before multivariate statistical analysis.
when_to_use_negative:
- Spectra already show high correlation with the reference (>0.95) in the selected
  region, i.e. there is no chemical-shift drift to correct
- The candidate reference spectrum lacks sufficient signal in the selected region
  — segmental alignment will not occur and a different reference must be chosen
- The analysis goal is isotopomer distribution derivation from NMR multiplet and GC-MS
  data — use the metabolabpytools isotopomer routines instead
edam_topics: []
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the segmental alignment algorithm applied to the selected spectral
    region of the 1D-NMR spectra series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: metabolabpy
  role: Installable PyPI package whose nmrData machinery loads the spectra and exposes
    the alignment routine
  repo: https://github.com/ludwigc/metabolabpy
- name: Python
  role: Implementation language used to run the alignment workflow and compute pre/post-alignment
    correlations
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Array backend for shifting and correlating spectral segments
- name: scipy
  role: Backend toolbox used by MetaboLabPy for spectral processing
- name: pandas
  role: Tabulates pre/post-alignment correlation coefficients and shift tables into
    CSV outputs
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:74c8f514a2c84a7fe671b03716a0beb6a4ded214fc1af50d964a339b48a623d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - Python
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# nmr-spectral-segmental-alignment

## Summary

Removes chemical-shift drift across a series of 1D-NMR spectra by computing, for each spectrum in a user-selected region, the spectral shift that maximizes simple Pearson correlation with a chosen reference spectrum, as implemented in MetaboLabPy's segmental alignment.

## When to use

Use when a series of 1D-NMR spectra (e.g. a metabolomics sample set) shows chemical-shift drift within a spectral region and peaks must be aligned to a reference spectrum before multivariate statistical analysis.

## When NOT to use

- Spectra already show high correlation with the reference (>0.95) in the selected region, i.e. there is no chemical-shift drift to correct
- The candidate reference spectrum lacks sufficient signal in the selected region — segmental alignment will not occur and a different reference must be chosen
- The analysis goal is isotopomer distribution derivation from NMR multiplet and GC-MS data — use the metabolabpytools isotopomer routines instead

## Inputs

- Series of 1D-NMR spectra (e.g. the Daphnia magna 1D-NMR spectra series used in the MetaboLabPy workflow demonstration)
- User-selected spectral region (segment) to align
- Reference spectrum chosen from the series, ideally the highest signal-to-noise spectrum

## Outputs

- Aligned 1D-NMR spectral series with chemical-shift drift removed relative to the reference
- Per-spectrum table of optimum shifts (segmental_alignment_shifts.csv)
- Tabulated pre- and post-alignment Pearson correlation coefficients against the reference (alignment_correlations.csv)

## How to apply

Load the series of 1D-NMR spectra into MetaboLabPy (v0.9.66, Python with numpy/scipy/pandas backend) and designate one spectrum — preferably the one with the highest signal-to-noise ratio — as the alignment reference. Select the user-defined spectral region containing the drifting resonances. For each spectrum in the series, compute simple correlation between the selected segment and the reference across candidate spectral shifts, and record the shift that maximizes correlation as the optimum. Skip alignment for any spectrum/segment where the reference region lacks sufficient signal, since no meaningful correlation optimum can be found there. Apply the optimum shifts to overlay all spectra on the reference; success is indicated when Pearson correlation coefficients against the reference rise from generally below 0.3 (pre-alignment) to above 0.95 (post-alignment).

## Related tools

- **MetaboLabPy 0.9.66** (Provides the segmental alignment algorithm applied to the selected spectral region of the 1D-NMR spectra series) — https://github.com/ludwigc/MetaboLabPy
- **metabolabpy** (Installable PyPI package whose nmrData machinery loads the spectra and exposes the alignment routine) — https://github.com/ludwigc/metabolabpy
- **Python** (Implementation language used to run the alignment workflow and compute pre/post-alignment correlations) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Array backend for shifting and correlating spectral segments)
- **scipy** (Backend toolbox used by MetaboLabPy for spectral processing)
- **pandas** (Tabulates pre/post-alignment correlation coefficients and shift tables into CSV outputs)

## Examples

```
metabolabpy --help
```

## Evaluation signals

- Pearson correlation coefficients between each spectrum and the reference in the selected region are generally below 0.3 before alignment
- After alignment, the same correlations exceed 0.95, matching the reported outcome for the Daphnia extract demonstration
- The output CSV contains exactly one optimum-shift entry per processed spectrum
- Spectra/segments where the reference lacks signal are skipped (no shift applied) rather than given a spurious optimum
- Expert visual review confirms aligned segments in the selected region overlay the reference spectrum

## Limitations

- No alignment occurs when the reference signal is lacking in the chosen segment, so reference choice (highest signal-to-noise spectrum) directly determines success
- The optimum shift is determined per segment by simple correlation with the reference, so it corrects only the drift within the user-selected region, not global mis-alignment
- No changelog found for the software (article discussion), making version-specific behavioural tracking of the alignment routine difficult

## Evidence

- [abstract] Segmental alignment in the MetaboLabPy workflow: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation"
- [abstract] Software version and purpose: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] Python backend: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] Package scope: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
