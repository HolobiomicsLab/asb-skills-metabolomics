---
name: sequential-gaussian-peak-fitting
description: Use when a 1D array sampled at known x-coordinates (e.g. an extracted
  ion chromatogram or ATD from an MZA/HDF5 file) contains overlapping or multiple
  peaks whose center (mean x), height, and width you want to estimate numerically.
when_to_use_negative:
- The signal is 2D/3D MS-IMS data needing full imaging rather than 1D peak parameterization
  — mzapy's calibration/CCS or ATD/XIC collection routines are the entry point, not
  sequential 1D Gaussian fitting
- The task is only locating peaks, not fitting center/height/width parameters — mzapy.peaks.find_peaks_1d_localmax
  is the lighter-weight alternative
- Data are raw vendor files (Agilent .d, Thermo .raw, mzML) not yet converted — run
  the MZA converter first to produce MZA/HDF5 files
tools:
- name: mzapy
  role: Python package whose mzapy.peaks module provides find_peaks_1d_gauss, the
    sequential least-squares Gaussian peak-fitting implementation, plus helper peak
    functions (find_peaks_1d_localmax, calc_peak_area, calc_gauss_psnr, lerp_1d)
  repo: https://github.com/PNNL-m-q/mzapy
- name: MZA
  role: 'Source of 1D signal arrays: vendor MS data converted to HDF5-based MZA format,
    from which chromatograms and arrival time distributions are extracted before peak
    fitting'
  repo: https://github.com/PNNL-m-q/mza
- name: Python
  role: Implementation language; synthetic 1D signals are built and fitted-parameter
    errors computed with Python (numpy/matplotlib are listed as dependencies)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:887d5fc8a0ef40dd28a2402d0b0f63f858593daba729166da3167b34fae4a100
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - MZA
  - Python
  - mzapy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# sequential-gaussian-peak-fitting

## Summary

Finds and quantifies peaks in 1D signals (e.g. extracted ion chromatograms or arrival time distributions) using mzapy.peaks.find_peaks_1d_gauss, which applies successive rounds of least-squares Gaussian fits on residuals until a stopping criterion is met.

## When to use

Use when a 1D array sampled at known x-coordinates (e.g. an extracted ion chromatogram or ATD from an MZA/HDF5 file) contains overlapping or multiple peaks whose center (mean x), height, and width you want to estimate numerically.

## When NOT to use

- The signal is 2D/3D MS-IMS data needing full imaging rather than 1D peak parameterization — mzapy's calibration/CCS or ATD/XIC collection routines are the entry point, not sequential 1D Gaussian fitting
- The task is only locating peaks, not fitting center/height/width parameters — mzapy.peaks.find_peaks_1d_localmax is the lighter-weight alternative
- Data are raw vendor files (Agilent .d, Thermo .raw, mzML) not yet converted — run the MZA converter first to produce MZA/HDF5 files

## Inputs

- 1D x-coordinate array (e.g. retention time, m/z, or ion mobility arrival time points)
- 1D intensity/signal array sampled at those x-coordinates (e.g. an XIC from collect_xic_arrays_by_mz or an ATD from collect_atd_arrays_by_rt_mz)
- Stopping-criterion controls: absolute or relative peak-height threshold, maximum number of peaks

## Outputs

- List of fitted peaks for every recovered peak
- Per-peak fitted parameters: mean x value (center), height, and width from each least-squares Gaussian fit
- Optionally, per-peak and per-parameter absolute/relative errors when ground-truth values are available

## How to apply

Pass the 1D x-coordinates and intensity array to mzapy.peaks.find_peaks_1d_gauss. The function runs successive rounds of least-squares Gaussian fits, fitting the residuals of the previous round, accumulating peaks until a stopping criterion is reached: fitted peak height falls below an absolute or relative threshold, or a maximum number of peaks is found. Extract the fitted parameters for each recovered peak — mean x value, height, and width. Validate on synthetic signals built by summing Gaussians with known center, height, and width at both dense and sparse point spacings, comparing fitted values to ground truth per peak and per parameter (absolute/relative errors); on clean, well-resolved peaks the sequential fitting reliably recovers ground-truth values. Complementary mzapy.peaks helpers include find_peaks_1d_localmax for local-maximum detection, calc_peak_area, calc_gauss_psnr, and lerp_1d/lerp_2d for interpolation.

## Related tools

- **mzapy** (Python package whose mzapy.peaks module provides find_peaks_1d_gauss, the sequential least-squares Gaussian peak-fitting implementation, plus helper peak functions (find_peaks_1d_localmax, calc_peak_area, calc_gauss_psnr, lerp_1d)) — https://github.com/PNNL-m-q/mzapy
- **MZA** (Source of 1D signal arrays: vendor MS data converted to HDF5-based MZA format, from which chromatograms and arrival time distributions are extracted before peak fitting) — https://github.com/PNNL-m-q/mza
- **Python** (Implementation language; synthetic 1D signals are built and fitted-parameter errors computed with Python (numpy/matplotlib are listed as dependencies))

## Examples

```
from mzapy.peaks import find_peaks_1d_gauss; peaks, params = find_peaks_1d_gauss(x, y)
```

## Evaluation signals

- Fitted centers, heights, and widths recover known ground-truth Gaussian parameters within reasonable absolute/relative error tolerance on synthetic well-resolved peaks
- The fit terminates via the documented stopping criterion: peak height below the absolute or relative threshold, or the maximum number of peaks reached (no runaway residual fitting)
- Every returned peak carries the full parameter triplet (mean x value, height, width); missing or non-finite parameters indicate a failed fit
- Behavior at dense vs. sparse point spacings remains consistent for the same underlying Gaussian peaks
- Recovered peak count matches the number of summed Gaussians in the synthetic signal (no spurious extra peaks above the height threshold)

## Limitations

- Designed for clean, well-formed, well-resolved 1D peaks; heavily overlapped, noisy, or irregular signals are not validated by the documented behavior
- Sparse point sampling degrades parameter recovery relative to dense sampling, so accuracy depends on the x-spacing of the extracted signal
- Stopping criteria are threshold-driven (absolute/relative height cutoff or max peak count), so low-height genuine peaks below threshold are intentionally not returned

## Evidence

- [readme] MZA converter inputs: [evidence span withheld — anchor status: external_doc]
