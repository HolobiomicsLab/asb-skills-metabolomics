---
name: python-scientific-code-implementation
description: Use when a paper or source card describes a numeric algorithm (e.g.,
  cubic-spline baseline correction of 1D NMR spectra with a linear-spline fallback)
  that must be re-implemented in Python from numpy arrays of ppm and intensity values.
when_to_use_negative:
- The input is already a processed or baseline-corrected spectrum — subtracting a
  spline baseline again would distort peak intensities
- The goal is a different pre-processing step from the MetaboLabPy workflow (e.g.,
  automated phase correction, segmental alignment, spectral scaling, or variance stabilisation)
  rather than baseline correction
- The spectrum is 2D NMR data; the described SplineBaseline workflow is formulated
  for 1D spectra
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: MetaboLabPy
  role: Reference implementation whose SplineBaseline behaviour (linear-spline fallback
    for gaps beyond a set maximum number of data points) is being re-implemented
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Provides CubicSpline used to fit the baseline model through the selected baseline
    points
- name: numpy
  role: Holds and manipulates the ppm and intensity arrays during baseline fitting
    and subtraction
- name: pandas
  role: Reads the raw spectrum and writes the corrected ppm/intensity arrays to baseline_corrected_spectrum.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:58d073c11f9acc31ea99dbc610a3d497ff310baea4e65b1bfb00e17802a8528b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - Python
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# python-scientific-code-implementation

## Summary

Implement a described scientific algorithm (here, MetaboLabPy's SplineBaseline correction with linear-spline gap filling) as working Python code using the same toolbox stack the source package relies on (numpy, scipy, pandas). The skill turns a documented method into a runnable script with concrete inputs, parameters, and validation.

## When to use

Use when a paper or source card describes a numeric algorithm (e.g., cubic-spline baseline correction of 1D NMR spectra with a linear-spline fallback) that must be re-implemented in Python from numpy arrays of ppm and intensity values.

## When NOT to use

- The input is already a processed or baseline-corrected spectrum — subtracting a spline baseline again would distort peak intensities
- The goal is a different pre-processing step from the MetaboLabPy workflow (e.g., automated phase correction, segmental alignment, spectral scaling, or variance stabilisation) rather than baseline correction
- The spectrum is 2D NMR data; the described SplineBaseline workflow is formulated for 1D spectra

## Inputs

- Raw 1D NMR spectrum as paired ppm and intensity arrays (numpy arrays or CSV; metabolabpy example dataset)
- Sparse selection of baseline anchor points in signal-free spectral regions
- Maximum-gap parameter: the set maximum number of data points allowed between adjacent baseline points before linear interpolation kicks in

## Outputs

- Baseline-corrected ppm/intensity arrays written to baseline_corrected_spectrum.csv
- Fitted cubic-spline baseline model evaluated across the full spectral width

## How to apply

Load the raw 1D NMR spectrum as ppm and intensity arrays (e.g., from the example dataset shipped with the metabolabpy package) using numpy/pandas. Select sparse baseline anchor points in signal-free regions, then fit scipy.interpolate.CubicSpline through them to model the baseline across the full spectral width. Implement the linear-spline fallback: when two adjacent baseline points are separated by more than the configured maximum number of data points, insert intermediate baseline points along the straight line between the pair and refit — this is MetaboLabPy's mechanism to avoid over-correcting dense spectral regions. Subtract the fitted baseline from the raw intensities and save the corrected ppm/intensity arrays (e.g., baseline_corrected_spectrum.csv). Validate that signal-free regions oscillate around zero and peak intensities are preserved relative to the raw spectrum.

## Related tools

- **MetaboLabPy** (Reference implementation whose SplineBaseline behaviour (linear-spline fallback for gaps beyond a set maximum number of data points) is being re-implemented) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Provides CubicSpline used to fit the baseline model through the selected baseline points)
- **numpy** (Holds and manipulates the ppm and intensity arrays during baseline fitting and subtraction)
- **pandas** (Reads the raw spectrum and writes the corrected ppm/intensity arrays to baseline_corrected_spectrum.csv)

## Examples

```
from scipy.interpolate import CubicSpline; cs = CubicSpline(anchor_ppm, anchor_int); corrected = raw_int - cs(ppm)
```

## Evaluation signals

- The corrected spectrum's baseline oscillates around zero intensity in signal-free regions
- Resonance peak intensities in the corrected spectrum are preserved relative to the raw spectrum (no clipping of dense peak regions)
- No pair of adjacent baseline points in the refit set exceeds the configured maximum number of data points — linear-spline gap filling has filled every oversized gap
- The output CSV contains paired ppm/intensity columns covering the same ppm range as the input spectrum

## Limitations

- The maximum-gap threshold is user-configured; setting it too small over-inserts linear points, too large permits over-correction in dense spectral regions
- Baseline anchor quality depends on correct selection of signal-free regions — anchors placed on peaks corrupt the spline fit
- The MetaboLabPy behaviour is reconstructed from documentation and the re-implementation may not match version 0.9.66 internals exactly

## Evidence

- [abstract] MetaboLabPy is implemented in Python with numpy/scipy/pandas: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc"
- [readme] MetaboLabPy targets 1D/2D NMR metabolomics data processing: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] MetaboLabPy provides a complete pre-processing workflow for multivariate analysis: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
