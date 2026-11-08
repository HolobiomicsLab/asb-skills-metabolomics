---
name: nmr-spectral-noise-region-selection
description: Use when a raw 1D NMR spectrum (ppm scale plus intensity array) has a
  distorted baseline and you need baseline points selected only in signal-free regions
  before spline fitting and subtraction.
when_to_use_negative:
- The spectrum already has a flat baseline oscillating around zero in signal-free
  regions, so re-fitting a spline risks removing real signal
- The baseline anchors cannot be placed in signal-free regions because peaks span
  the whole spectral width — sparse anchors would sit on resonances and bias the spline
- The task is phase correction, segmental alignment, spectral scaling, or variance
  stabilisation rather than baseline removal
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0593
tools:
- name: MetaboLabPy
  role: Reference implementation whose SplineBaseline class performs the baseline
    correction with the linear-spline gap-filling fallback; example datasets from
    the metabolabpy package supply the input spectra
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Provides scipy.interpolate.CubicSpline used to fit the baseline model through
    the selected noise-region points
- name: numpy
  role: Array handling for the ppm axis, intensity array, and simulated linear-spline
    intermediate points
- name: pandas
  role: Loading the example dataset and saving the corrected ppm/intensity arrays
    to baseline_corrected_spectrum.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:b4c53b289f9a386b5625fe8671f4842f46723c53485069122df089f75d89803d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nmr-spectral-noise-region-selection

## Summary

Selects sparse anchor points in signal-free (noise) regions of a 1D NMR spectrum and fits a cubic-spline baseline through them, with a linear-spline fallback that inserts intermediate points between widely spaced anchors to avoid over-correcting dense spectral regions.

## When to use

Use when a raw 1D NMR spectrum (ppm scale plus intensity array) has a distorted baseline and you need baseline points selected only in signal-free regions before spline fitting and subtraction.

## When NOT to use

- The spectrum already has a flat baseline oscillating around zero in signal-free regions, so re-fitting a spline risks removing real signal
- The baseline anchors cannot be placed in signal-free regions because peaks span the whole spectral width — sparse anchors would sit on resonances and bias the spline
- The task is phase correction, segmental alignment, spectral scaling, or variance stabilisation rather than baseline removal

## Inputs

- Raw 1D NMR spectrum as a ppm-scale axis array and a corresponding intensity array
- Manually or heuristically chosen baseline anchor points located in signal-free spectral regions
- A configured maximum number of data points allowed between adjacent baseline anchors (gap threshold)

## Outputs

- Fitted baseline curve spanning the full spectral width
- Baseline-corrected spectrum (corrected ppm/intensity arrays)
- Corrected spectrum exported as baseline_corrected_spectrum.csv

## How to apply

Load the raw 1D spectrum as ppm and intensity arrays (e.g. from an example dataset shipped with the metabolabpy package, read with numpy/pandas). Manually select a small number of sparse baseline points in signal-free spectral regions where no resonance peaks occur; these anchors define the baseline model. Fit a cubic spline (scipy.interpolate.CubicSpline) through the anchors across the full spectral width. Apply the linear-spline fallback: whenever two adjacent baseline points are separated by more than a configured maximum number of data points, simulate intermediate baseline points along the straight line joining that pair and refit — this is what prevents over-correction in dense areas of the spectrum. Subtract the fitted baseline from the raw intensities to obtain the corrected spectrum, then persist the corrected ppm/intensity arrays (e.g. to baseline_corrected_spectrum.csv).

## Related tools

- **MetaboLabPy** (Reference implementation whose SplineBaseline class performs the baseline correction with the linear-spline gap-filling fallback; example datasets from the metabolabpy package supply the input spectra) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Provides scipy.interpolate.CubicSpline used to fit the baseline model through the selected noise-region points)
- **numpy** (Array handling for the ppm axis, intensity array, and simulated linear-spline intermediate points)
- **pandas** (Loading the example dataset and saving the corrected ppm/intensity arrays to baseline_corrected_spectrum.csv)

## Examples

```
from scipy.interpolate import CubicSpline; cs = CubicSpline(anchor_ppm, anchor_int); corrected = intensity - cs(ppm)
```

## Evaluation signals

- The corrected spectrum shows a flat baseline oscillating around zero intensity in signal-free regions
- Resonance peak intensities in the corrected spectrum are preserved relative to the raw spectrum
- Every simulated intermediate baseline point falls on the straight line joining the adjacent pair of anchors that exceeded the maximum-gap threshold
- The corrected ppm/intensity arrays are written to baseline_corrected_spectrum.csv with matching axis lengths

## Limitations

- Anchor-point choice is the critical decision: anchors placed on real resonances will drag the spline up and suppress peaks; anchors must lie in signal-free regions
- If the maximum-gap threshold between adjacent anchors is set too large in a dense spectral region, the plain cubic spline can over-correct; the linear-spline fallback only protects regions where the gap actually exceeds the configured limit
- The linear fallback cannot recover a sensible baseline in regions with no nearby anchors at all

## Evidence

- [abstract] MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
