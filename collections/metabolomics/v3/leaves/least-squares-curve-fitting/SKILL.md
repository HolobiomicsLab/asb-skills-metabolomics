---
name: least-squares-curve-fitting
description: Use when a 1D signal (e.g., an extracted ion chromatogram or arrival
  time distribution from MZA/HDF5 data) contains Gaussian-shaped peaks whose centers,
  heights, and widths must be quantified, not just located.
when_to_use_negative:
- You only need approximate peak positions rather than fitted Gaussian parameters
  — mzapy.peaks.find_peaks_1d_localmax is the non-parametric alternative in the same
  module
- Peaks are severely overlapping or non-Gaussian, so sequential least-squares Gaussian
  fitting on residuals is not a reliable model
- Data are still in vendor formats (Agilent .d, Thermo .raw, mzML) and must first
  be converted to MZA/HDF5 with the mza converter
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzapy
  role: Provides mzapy.peaks.find_peaks_1d_gauss, which runs the successive least-squares
    Gaussian fits on residuals with height-threshold / max-peak stopping criteria
  repo: https://github.com/PNNL-m-q/mzapy
- name: Python
  role: Implementation language; mzapy is pure Python with minimal standardized dependencies
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:dc01457c8bbc5772a689c363009edcd6eb9d8488a1f1703c1ec4810e4ff4a96f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# least-squares-curve-fitting

## Summary

Fit peaks in a 1D signal with successive rounds of least-squares Gaussian fitting using mzapy.peaks.find_peaks_1d_gauss, extracting fitted mean, height, and width for every recovered peak. This operationalizes mzapy's peak-finding utility beyond raw data extraction for MS/IMS signals.

## When to use

Use when a 1D signal (e.g., an extracted ion chromatogram or arrival time distribution from MZA/HDF5 data) contains Gaussian-shaped peaks whose centers, heights, and widths must be quantified, not just located.

## When NOT to use

- You only need approximate peak positions rather than fitted Gaussian parameters — mzapy.peaks.find_peaks_1d_localmax is the non-parametric alternative in the same module
- Peaks are severely overlapping or non-Gaussian, so sequential least-squares Gaussian fitting on residuals is not a reliable model
- Data are still in vendor formats (Agilent .d, Thermo .raw, mzML) and must first be converted to MZA/HDF5 with the mza converter

## Inputs

- 1D signal sampled as x and intensity arrays (dense or sparse point spacing), e.g., summed Gaussian peaks with known ground-truth center, height, and width
- Stopping-criterion settings: absolute or relative peak-height threshold and/or maximum number of peaks

## Outputs

- Fitted peak parameters (mean x value, height, width) per recovered peak
- Absolute/relative error table per peak and per parameter, written to fitted_peak_parameters.csv

## How to apply

Build or obtain the 1D signal (e.g., sum Gaussians with known center, height, and width, sampled at dense and sparse point spacings) and run mzapy.peaks.find_peaks_1d_gauss on it. The function performs successive rounds of least-squares Gaussian fits on the residuals of the previous fit until a stopping criterion is reached: peak height falling below an absolute or relative threshold, or a maximum number of peaks found. For each fitted peak, extract the three fitted parameters — mean x value, height, and width — and compare them against ground-truth values, computing absolute and relative errors per peak and per parameter. Write the ground-truth and fitted parameters with errors to fitted_peak_parameters.csv. Acceptance is that centers, heights, and widths recover ground truth within reasonable tolerance for well-formed synthetic peaks; sequential Gaussian fitting is expected to give reliable parameter estimates for clean, well-resolved signals.

## Related tools

- **mzapy** (Provides mzapy.peaks.find_peaks_1d_gauss, which runs the successive least-squares Gaussian fits on residuals with height-threshold / max-peak stopping criteria) — https://github.com/PNNL-m-q/mzapy
- **Python** (Implementation language; mzapy is pure Python with minimal standardized dependencies)

## Examples

```
from mzapy.peaks import find_peaks_1d_gauss; fitted_params = find_peaks_1d_gauss(x, y)
```

## Evaluation signals

- Fitted centers, heights, and widths recover ground-truth Gaussian values within reasonable tolerance (small absolute/relative errors) on well-formed synthetic peaks
- Fitting terminates only when a stopping criterion is met: fitted peak height below the absolute/relative threshold or the maximum number of peaks reached
- Performance holds at both dense and sparse point spacings when ground-truth Gaussians are resampled
- Ground-truth and fitted parameters with per-peak, per-parameter errors are present in fitted_peak_parameters.csv

## Limitations

- Reliable parameter estimates are documented for clean, well-resolved signals; crowded or noisy residual rounds may degrade fits
- Stopping depends on height thresholds and a max-peak cap, so low-amplitude true peaks can be missed if thresholds are set too aggressively
- The model assumes Gaussian peak shape (three parameters: center, height, width); other peak shapes are outside this function's scope

## Evidence

- [other] Run mzapy.peaks.find_peaks_1d_gauss on each synthetic signal, which performs successive rounds of least-squares Gaussian fits on the residuals: [evidence span withheld — anchor status: not_found]
- [other] peak height below an absolute or relative threshold, or a maximum number of peaks found: [evidence span withheld — anchor status: not_found]
- [other] fitted centers, heights, and widths recover the ground-truth values within reasonable tolerance: [evidence span withheld — anchor status: not_found]
