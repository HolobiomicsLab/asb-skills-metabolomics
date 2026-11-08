---
name: nmr-baseline-correction
description: Use when a raw 1D NMR spectrum (ppm scale and intensity array) shows
  a curved or offset baseline that must be subtracted prior to multivariate statistical
  analysis, and dense peak regions risk being over-corrected by a naive spline fit.
when_to_use_negative:
- Input is already a baseline-corrected or variance-stabilised spectrum destined directly
  for multivariate analysis
- Task is 2D NMR processing, segmental alignment, spectral scaling, or automated phase
  correction rather than baseline flattening
- The goal is isotopomer distribution derivation from NMR multiplet and GC-MS data
  (metabolabpytools scope)
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MetaboLabPy
  role: Reference implementation whose SplineBaseline class defines the cubic-spline-plus-linear-spline-fallback
    baseline correction workflow
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Provides CubicSpline for fitting the baseline through the selected anchor
    points
  repo: https://github.com/ludwigc/MetaboLabPy
- name: numpy
  role: Array handling for ppm/intensity arrays and interpolation of intermediate
    baseline points
  repo: https://github.com/ludwigc/MetaboLabPy
- name: pandas
  role: Reading the example dataset and writing the corrected ppm/intensity arrays
    to CSV
  repo: https://github.com/ludwigc/MetaboLabPy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:6d01c133fa1f4d89303748a9d6f9a84c196415960992c1ea4e555088fb3ce958
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nmr-baseline-correction

## Summary

Reconstructs MetaboLabPy's SplineBaseline approach: anchor a cubic spline on sparse signal-free baseline points, then apply a 'linear-spline' fallback that inserts intermediate baseline points along straight lines between adjacent anchors separated by more than a set maximum number of data points, preventing over-correction in dense spectral regions of 1D NMR spectra.

## When to use

Use when a raw 1D NMR spectrum (ppm scale and intensity array) shows a curved or offset baseline that must be subtracted prior to multivariate statistical analysis, and dense peak regions risk being over-corrected by a naive spline fit.

## When NOT to use

- Input is already a baseline-corrected or variance-stabilised spectrum destined directly for multivariate analysis
- Task is 2D NMR processing, segmental alignment, spectral scaling, or automated phase correction rather than baseline flattening
- The goal is isotopomer distribution derivation from NMR multiplet and GC-MS data (metabolabpytools scope)

## Inputs

- Raw 1D NMR spectrum as paired ppm and intensity arrays (numpy arrays or CSV)
- User-selected sparse baseline points in signal-free spectral regions
- Configured maximum number of data points allowed between adjacent baseline points

## Outputs

- Baseline-corrected ppm/intensity arrays (e.g. baseline_corrected_spectrum.csv)
- Fitted baseline model (cubic spline with linear-spline gap filling)

## How to apply

Load the raw 1D spectrum as ppm and intensity arrays (e.g. from the example dataset shipped with the metabolabpy package) using numpy/pandas. Select sparse baseline anchor points in signal-free spectral regions, then fit a cubic spline through them with scipy.interpolate.CubicSpline to model the baseline across the full spectral width. Implement the linear-spline fallback: for each pair of adjacent baseline points separated by more than the configured maximum number of data points, insert intermediate baseline points along the straight line joining the pair before refitting — this is how SplineBaseline avoids over-correcting dense spectral regions. Subtract the fitted baseline from the raw intensities and save the corrected ppm/intensity arrays (e.g. to baseline_corrected_spectrum.csv).

## Related tools

- **MetaboLabPy** (Reference implementation whose SplineBaseline class defines the cubic-spline-plus-linear-spline-fallback baseline correction workflow) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Provides CubicSpline for fitting the baseline through the selected anchor points) — https://github.com/ludwigc/MetaboLabPy
- **numpy** (Array handling for ppm/intensity arrays and interpolation of intermediate baseline points) — https://github.com/ludwigc/MetaboLabPy
- **pandas** (Reading the example dataset and writing the corrected ppm/intensity arrays to CSV) — https://github.com/ludwigc/MetaboLabPy

## Examples

```
from scipy.interpolate import CubicSpline; cs = CubicSpline(baseline_ppm, baseline_int); corrected = raw_int - cs(ppm)
```

## Evaluation signals

- Corrected spectrum has a flat baseline oscillating around zero intensity in signal-free regions
- Resonance peak intensities are preserved relative to the raw spectrum
- No over-correction occurs in dense spectral regions wherever adjacent baseline points were more than the maximum allowed data-point gap apart
- Every gap between adjacent baseline points larger than the configured maximum is bridged by linearly interpolated intermediate points
- Output CSV contains the same ppm axis as the input with corrected intensity values of the same shape

## Limitations

- Quality depends on the sparse baseline points being chosen in genuinely signal-free spectral regions; anchors placed on peaks will distort the baseline
- Linear-spline gap filling only guards against over-correction when the maximum data-point separation parameter is set appropriately for the spectral density
- The README gives no changelog or method-level documentation for SplineBaseline parameters; consult the source code

## Evidence

- [other] SplineBaseline avoids over-correction of dense spectral regions via a 'linear-spline' fallback: when two adjacent baseline points are more than a set maximum number of data points apart, intermediate baseline points are simulated along a line between them.: [evidence span withheld — anchor status: not_found]
- [abstract] The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc.: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas"
