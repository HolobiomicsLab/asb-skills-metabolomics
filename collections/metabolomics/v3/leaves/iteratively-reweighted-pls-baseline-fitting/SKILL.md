---
name: iteratively-reweighted-pls-baseline-fitting
description: Use when 1H-NMR spectra (rows = samples, columns = chemical shifts in
  ppm) still show baseline drift after internal referencing — e.
when_to_use_negative:
- The spectra have already been baseline-corrected (e.g. you already hold a bc_df
  from a prior px.baseline_correction call) — re-correcting risks distorting genuine
  signal.
- The data are still FIDs (fids_df, grpd_df, apod_df, zf_df) rather than spectra —
  this skill operates on the frequency-domain spectra_df after Fourier transform,
  phase correction, and internal referencing.
- You need a baseline algorithm outside ALS/ARPLS/AIRPLS — Protomix implements only
  these three from pyBaselines, so other strategies require using pyBaselines directly.
edam_topics:
- http://edamontology.org/topic_3892
tools:
- name: Protomix
  role: Provides the baseline_correction function that fits and subtracts the ALS/ARPLS/AIRPLS
    baseline from the spectra DataFrame.
  repo: https://github.com/mzniber/protomix
- name: pyBaselines
  role: Source package from which Protomix's ALS, ARPLS, and AIRPLS baseline correction
    methods are adapted.
- name: pandas
  role: Holds the input and output spectra DataFrames (rows = samples, columns = ppm)
    passed to and returned from baseline_correction.
- name: numpy
  role: Numerical backend used by Protomix for the iterative penalized least squares
    fitting.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbae192
    title: 'Protomix: a Python package for 1H-NMR metabolomics data preprocessing'
  source_content_hash: sha256:3f029aac7c6d592e1da319b1b5518c0c9a54b117f4f71326c0913126db52607a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mzniber/protomix
  tools:
  - Protomix
  - Python
  - numpy
  - pandas
  - pyBaselines
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3892
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Iteratively-reweighted PLS baseline fitting

## Summary

Baseline correction of 1H-NMR spectra via penalized least squares schemes — asymmetric least squares (ALS), adaptive reweighted PLS (ARPLS), and automatic iterative reweighted PLS (AIRPLS) — as implemented in Protomix's baseline_correction function, adapted from the pyBaselines package.

## When to use

Use when 1H-NMR spectra (rows = samples, columns = chemical shifts in ppm) still show baseline drift after internal referencing — e.g. from incomplete relaxation, pulse sequence imperfections, magnetic field inhomogeneities, solvent effects, phase errors, or macromolecules — and a corrected spectra DataFrame is needed before binning/normalization.

## When NOT to use

- The spectra have already been baseline-corrected (e.g. you already hold a bc_df from a prior px.baseline_correction call) — re-correcting risks distorting genuine signal.
- The data are still FIDs (fids_df, grpd_df, apod_df, zf_df) rather than spectra — this skill operates on the frequency-domain spectra_df after Fourier transform, phase correction, and internal referencing.
- You need a baseline algorithm outside ALS/ARPLS/AIRPLS — Protomix implements only these three from pyBaselines, so other strategies require using pyBaselines directly.

## Inputs

- spectra_df: pandas DataFrame of phase-corrected, internally referenced 1H-NMR spectra (rows = samples, columns = chemical shifts in ppm), e.g. the ir_df produced by px.internal_referencing
- method: one of 'asls', 'arpls', 'airpls'

## Outputs

- bc_df: pandas DataFrame of baseline-corrected spectra with the same row/column layout as the input (columns = chemical shifts in ppm, rows = samples)
- Optionally, a combined CSV (baseline_corrected_spectra.csv) holding one column group per correction method

## How to apply

Start from the internally referenced spectra DataFrame (ir_df) produced by px.internal_referencing, where columns are chemical shifts in ppm and rows are samples. Call px.baseline_correction(spectra_df=ir_df, method='asls') for asymmetric least squares, method='arpls' for adaptive reweighted penalized least squares, or method='airpls' for automatic iterative reweighted penalized least squares — the three methods Protomix adapts from pyBaselines; the tutorial pipeline uses bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls'). The function fits and subtracts a smooth baseline from each spectrum, returning a corrected spectra DataFrame. To compare strategies, run all three methods and assemble the corrected matrices as one column group per method, then write to baseline_corrected_spectra.csv. Because only these three methods are wrapped, users wanting other baseline strategies should explore the pyBaselines package directly.

## Related tools

- **Protomix** (Provides the baseline_correction function that fits and subtracts the ALS/ARPLS/AIRPLS baseline from the spectra DataFrame.) — https://github.com/mzniber/protomix
- **pyBaselines** (Source package from which Protomix's ALS, ARPLS, and AIRPLS baseline correction methods are adapted.)
- **pandas** (Holds the input and output spectra DataFrames (rows = samples, columns = ppm) passed to and returned from baseline_correction.)
- **numpy** (Numerical backend used by Protomix for the iterative penalized least squares fitting.)

## Examples

```
bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')
```

## Evaluation signals

- The output file baseline_corrected_spectra.csv exists and contains corrected spectra matrices with rows = samples and columns = chemical shifts in ppm.
- Each method's corrected spectra show the baseline drifts removed and a smoother baseline compared to the input ir_df, as specified for the Baseline Correction feature.
- Running asls, arpls, and airpls yields three corrected matrices that can be assembled as one column group per method in a single DataFrame.
- The corrected output preserves the input DataFrame layout: same number of rows (samples) and ppm columns as the internally referenced spectra_df.
- Negative-values or residual drift in flat, signal-free ppm regions is reduced relative to the pre-correction spectra.

## Limitations

- Only three baseline correction methods (ALS, ARPLS, AIRPLS) are wrapped; other pyBaselines strategies must be explored directly from pyBaselines.
- Baseline distortions stem from multiple physical sources — incomplete relaxation, pulse sequence imperfections, magnetic field inhomogeneities, solvent effects, phase errors, and macromolecules — so a single fitted baseline may not remove all distortion modes.
- The Protomix repository context shows no changelog present, limiting traceability of method changes.

## Evidence

- [methods] Protomix's baseline_correction function adapts methods from pyBaselines: "bc_df = px.baseline_correction(spectra_df=ir_df, method='airpls')"
- [readme] Baseline Correction is a listed Protomix feature: [evidence span withheld — anchor status: external_doc]
- [abstract] Baseline correction sits inside Protomix's preprocessing suite for metabolomics: "Protomix offers a set of automated, efficient, and user-friendly signal-preprocessing steps, tailored to streamline and enhance the preprocessing phase in metabolomics studies."
- [readme] Protomix is a Python package for NMR data preprocessing: "Protomix is a comprehensive Python package designed for preprocessing Nuclear Magnetic Resonance (NMR) data."
