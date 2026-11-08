---
name: spectral-preprocessing-for-multivariate-analysis
description: Use when a series of raw 1D-NMR spectra (ppm scale + intensity arrays)
  must be cleaned of phase, baseline, and alignment artefacts before export to multivariate
  statistical analysis.
when_to_use_negative:
- Input is already a pre-processed spectra matrix or feature table ready for multivariate
  analysis
- Data are 2D-NMR or HSQC + GC-MS isotopomer data handled by metabolabpytools' train/predict
  workflow rather than 1D spectral pre-processing
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MetaboLabPy
  role: Implements the full 1D-NMR pre-processing workflow including SplineBaseline
    correction with linear-spline gap filling
  repo: https://github.com/ludwigc/MetaboLabPy
- name: metabolabpy
  role: Python package providing the SplineBaseline class and example NMR datasets
  repo: https://github.com/ludwigc/metabolabpy
- name: numpy
  role: Array handling for ppm and intensity arrays during pre-processing
- name: scipy
  role: CubicSpline interpolation through selected baseline points
- name: pandas
  role: Loading spectra and exporting corrected ppm/intensity arrays to CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:633b4654472d1be208fed89aae5660814fe4e560e14456a4bd60709518187425
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - metabolabpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# spectral-preprocessing-for-multivariate-analysis

## Summary

A MetaboLabPy-based workflow for pre-processing series of 1D-NMR spectra — phase correction, baseline correction (with cubic-spline and linear-spline gap filling), segmental alignment, spectral scaling, and variance stabilisation — to prepare data matrices for multivariate statistical analysis.

## When to use

Use when a series of raw 1D-NMR spectra (ppm scale + intensity arrays) must be cleaned of phase, baseline, and alignment artefacts before export to multivariate statistical analysis.

## When NOT to use

- Input is already a pre-processed spectra matrix or feature table ready for multivariate analysis
- Data are 2D-NMR or HSQC + GC-MS isotopomer data handled by metabolabpytools' train/predict workflow rather than 1D spectral pre-processing

## Inputs

- Raw 1D-NMR spectrum (ppm scale and intensity array) from the example dataset included with the metabolabpy package
- Series of 1D-NMR spectra for batch pre-processing

## Outputs

- Phase-corrected, baseline-corrected, aligned and scaled 1D-NMR spectra (e.g. baseline_corrected_spectrum.csv with ppm/intensity columns)
- Pre-processed spectra matrix ready for multivariate statistical analysis

## How to apply

Load the raw 1D NMR spectrum (ppm scale and intensity array) using numpy/pandas, then apply MetaboLabPy's pre-processing chain in order: automated phase correction, baseline correction, segmental alignment, spectral scaling, and variance stabilisation. For baseline correction, select sparse anchor points in signal-free regions, fit a cubic spline (scipy.interpolate.CubicSpline) through them, and apply the 'linear-spline' fallback: when two adjacent baseline points are more than a set maximum number of data points apart, simulate intermediate baseline points along a straight line between them before refitting, which prevents over-correction in dense spectral regions. Subtract the fitted baseline from the raw intensities, then export the corrected spectra to platforms for downstream multivariate analysis. Validation relies on a corrected baseline that oscillates around zero in signal-free regions while preserving resonance peak intensities relative to the raw spectrum.

## Related tools

- **MetaboLabPy** (Implements the full 1D-NMR pre-processing workflow including SplineBaseline correction with linear-spline gap filling) — https://github.com/ludwigc/MetaboLabPy
- **metabolabpy** (Python package providing the SplineBaseline class and example NMR datasets) — https://github.com/ludwigc/metabolabpy
- **numpy** (Array handling for ppm and intensity arrays during pre-processing)
- **scipy** (CubicSpline interpolation through selected baseline points)
- **pandas** (Loading spectra and exporting corrected ppm/intensity arrays to CSV)

## Examples

```
from scipy.interpolate import CubicSpline; cs = CubicSpline(baseline_ppm, baseline_int); corrected = raw_int - cs(ppm)
```

## Evaluation signals

- Corrected spectrum has a flat baseline oscillating around zero intensity in signal-free regions
- Resonance peak intensities are preserved relative to the raw spectrum (no over-correction of dense spectral regions)
- No residual gaps in the baseline model: every pair of adjacent baseline points is within the configured maximum number of data points after linear-spline fallback insertion
- Output file (e.g. baseline_corrected_spectrum.csv) contains matched ppm/intensity arrays with the same number of data points as the raw input

## Limitations

- Over-correction of dense spectral regions occurs if the linear-spline fallback is not applied when adjacent baseline points exceed the maximum point separation
- Baseline anchor points must be placed in genuinely signal-free regions; misplacement degrades the spline model
- Baseline correction is one step of a larger pipeline (phase correction, segmental alignment, spectral scaling, variance stabilisation must also be applied for multivariate analysis)

## Evidence

- [abstract] Complete 1D-NMR pre-processing workflow: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] Pre-processing steps in the chain: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] Toolchain grounding: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] MetaboLabPy scope: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [methods] Task card finding on linear-spline fallback: [evidence span withheld — anchor status: not_found]
