---
name: cubic-spline-interpolation
description: Use when a raw 1D NMR spectrum (ppm scale + intensity array) shows a
  rolling or offset baseline and peak shapes must be preserved while flattening signal-free
  regions to zero intensity.
when_to_use_negative:
- The spectrum is 2D or the task is isotopomer/HSQC + GC-MS analysis — the SplineBaseline
  workflow and its linear-spline fallback are defined for 1D NMR spectra
- The baseline is already flat around zero in signal-free regions — spline subtraction
  would add variance rather than remove baseline error
- Baseline anchors are needed inside dense resonance regions where signal-free points
  cannot be selected — over-correction risk is precisely what the fallback mitigates,
  not eliminates
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: MetaboLabPy
  role: Reference implementation of SplineBaseline, whose linear-spline fallback this
    skill reproduces
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Provides CubicSpline used to fit the baseline model through the selected baseline
    points
- name: numpy
  role: Array handling for ppm/intensity arrays and simulating intermediate baseline
    points along linear gaps
- name: pandas
  role: Loading the example dataset and saving baseline_corrected_spectrum.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:e231a1d35215699752c93ecf2e28041f5e894512df9afb7b5c17d3a7946543a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cubic-spline-interpolation

## Summary

Fits a cubic spline (scipy.interpolate.CubicSpline) through sparse baseline points selected in signal-free regions of a 1D NMR spectrum, with a linear-spline fallback that inserts intermediate baseline points along straight lines when adjacent anchors are more than a configured maximum number of data points apart, preventing over-correction in dense spectral regions.

## When to use

Use when a raw 1D NMR spectrum (ppm scale + intensity array) shows a rolling or offset baseline and peak shapes must be preserved while flattening signal-free regions to zero intensity.

## When NOT to use

- The spectrum is 2D or the task is isotopomer/HSQC + GC-MS analysis — the SplineBaseline workflow and its linear-spline fallback are defined for 1D NMR spectra
- The baseline is already flat around zero in signal-free regions — spline subtraction would add variance rather than remove baseline error
- Baseline anchors are needed inside dense resonance regions where signal-free points cannot be selected — over-correction risk is precisely what the fallback mitigates, not eliminates

## Inputs

- Raw 1D NMR spectrum as paired ppm and intensity arrays (e.g. numpy arrays or CSV read with pandas)
- Sparse baseline anchor points (ppm, intensity pairs) selected in signal-free regions
- Maximum-gap parameter: the set maximum number of data points allowed between two adjacent baseline points before linear-spline gap filling triggers

## Outputs

- Baseline-corrected ppm/intensity arrays (saved as baseline_corrected_spectrum.csv)
- Fitted baseline curve evaluated across the full spectral width

## How to apply

Load the raw 1D spectrum's ppm and intensity arrays (e.g. from the example dataset shipped with the metabolabpy package) using numpy/pandas. Select sparse baseline anchor points only in signal-free spectral regions, then fit a scipy.interpolate.CubicSpline through those anchors across the full spectral width. Implement MetaboLabPy's SplineBaseline safeguard: for each pair of adjacent baseline points separated by more than the configured maximum number of data points, simulate intermediate baseline points along the straight line joining the pair and refit — this linear-spline gap filling stops the cubic spline from over-correcting dense areas of the spectrum. Subtract the fitted baseline from the raw intensities to obtain the corrected spectrum.

## Related tools

- **MetaboLabPy** (Reference implementation of SplineBaseline, whose linear-spline fallback this skill reproduces) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Provides CubicSpline used to fit the baseline model through the selected baseline points)
- **numpy** (Array handling for ppm/intensity arrays and simulating intermediate baseline points along linear gaps)
- **pandas** (Loading the example dataset and saving baseline_corrected_spectrum.csv)

## Examples

```
from scipy.interpolate import CubicSpline; cs = CubicSpline(baseline_ppm, baseline_int); corrected = raw_int - cs(ppm)
```

## Evaluation signals

- Corrected spectrum has a flat baseline oscillating around zero intensity in signal-free regions
- Resonance peak intensities are preserved relative to the raw spectrum (no over-correction of dense regions)
- Every gap between adjacent baseline anchors larger than the configured maximum data-point count has straight-line intermediate points inserted before the refit
- Output file baseline_corrected_spectrum.csv contains paired ppm and corrected-intensity columns with the same ppm grid as the input

## Limitations

- Over-correction in dense areas of the spectrum occurs if the linear-spline fallback is skipped or the maximum data-point gap between adjacent baseline points is set too large
- Anchor selection is manual/subjective — points must lie in signal-free regions or peaks will be subtracted
- The exact maximum-gap threshold is configurable; no universal default is specified in the source material

## Evidence

- [abstract] MetaboLabPy package context: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] Python toolboxes used: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] Package purpose: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
