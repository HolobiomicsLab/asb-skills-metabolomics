---
name: local-maximum-peak-detection
description: Use when you need to locate and quantify discrete peaks in a 1D intensity
  array (such as an extracted ion chromatogram from MZA data) and can validate detections
  against known or ground-truth peak positions and heights.
when_to_use_negative:
- Peaks are overlapped or not well-resolved — local-maximum detection cannot deconvolve
  co-eluting features; use a peak-fitting approach such as mzapy.peaks.find_peaks_1d_gauss
  instead
- Input is 2D data (e.g., an IMS drift-time by m/z matrix) rather than a 1D trace
  — this skill applies to 1D signals only
- Raw vendor files (.d, .raw, .mzML) that have not yet been converted to MZA/HDF5
  — run the mza converter first
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: mzapy
  role: Provides find_peaks_1d_localmax in the mzapy.peaks module, the local-maximum
    peak-finding function being applied
  repo: https://github.com/PNNL-m-q/mzapy
- name: numpy
  role: Generates the synthetic 1D Gaussian test signal with a seeded RNG and supplies
    the array data on the m/z-like axis
- name: scipy
  role: scipy.signal.find_peaks performs the underlying local-maximum detection invoked
    inside find_peaks_1d_localmax
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:a150953662c35c88962a0db520cc78b013deedce8c434e08b1e5142939995a82
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# local-maximum-peak-detection

## Summary

Detects peaks in 1D mass-spectrometry signals (e.g., extracted ion chromatograms or m/z-like axes) by identifying local maxima with mzapy.peaks.find_peaks_1d_localmax, one of mzapy's peak-finding utilities alongside find_peaks_1d_gauss and calc_peak_area.

## When to use

Use when you need to locate and quantify discrete peaks in a 1D intensity array (such as an extracted ion chromatogram from MZA data) and can validate detections against known or ground-truth peak positions and heights.

## When NOT to use

- Peaks are overlapped or not well-resolved — local-maximum detection cannot deconvolve co-eluting features; use a peak-fitting approach such as mzapy.peaks.find_peaks_1d_gauss instead
- Input is 2D data (e.g., an IMS drift-time by m/z matrix) rather than a 1D trace — this skill applies to 1D signals only
- Raw vendor files (.d, .raw, .mzML) that have not yet been converted to MZA/HDF5 — run the mza converter first

## Inputs

- 1D intensity array (e.g., numpy array of signal intensities on an m/z-like or RT axis)
- Optional 1D x-axis array for mapping returned indices to positions
- For validation: synthetic signal parameters (known Gaussian peak x positions and heights, noise floor level)

## Outputs

- Detected peak indices into the input array
- peak_list.csv with columns index, x, height (one row per validated detected peak)

## How to apply

First generate or obtain a 1D signal on an m/z-like axis; for validation, build a synthetic signal in numpy with a seeded RNG as a sum of 5 well-resolved Gaussian peaks at known x positions and heights plus a small noise floor, and record the ground-truth parameters. Call mzapy.peaks.find_peaks_1d_localmax on the signal (it wraps scipy.signal.find_peaks internally) to detect local-maximum peaks. Map the returned peak indices back to x positions and heights, then compare each detected peak to the ground-truth list within a stated tolerance on both x position and height. Write the validated detections to peak_list.csv with columns index, x, height. Well-resolved peaks are essential: local-maximum detection cannot separate overlapped peaks, and the noise floor sets the risk of spurious detections.

## Related tools

- **mzapy** (Provides find_peaks_1d_localmax in the mzapy.peaks module, the local-maximum peak-finding function being applied) — https://github.com/PNNL-m-q/mzapy
- **numpy** (Generates the synthetic 1D Gaussian test signal with a seeded RNG and supplies the array data on the m/z-like axis)
- **scipy** (scipy.signal.find_peaks performs the underlying local-maximum detection invoked inside find_peaks_1d_localmax)

## Examples

```
from mzapy.peaks import find_peaks_1d_localmax; peak_indices = find_peaks_1d_localmax(signal)
```

## Evaluation signals

- peak_list.csv exists and contains exactly 5 rows for the 5-peak synthetic signal
- Every recovered x position in peak_list.csv matches a ground-truth Gaussian peak position within the specified tolerance
- Recovered peak heights match the known Gaussian amplitudes within tolerance
- No spurious detections are attributed to the noise floor
- Output schema check: columns are exactly index, x, height

## Limitations

- Local-maximum detection presupposes well-resolved peaks; overlapped or shoulder peaks will not be separated
- A noise floor can produce spurious local maxima, so detections must be validated against ground truth or tolerances before acceptance
- For ion mobility spectra, m/z may be stored as mzbins indices into a shared Full_mz_array rather than direct m/z values, which must be resolved before peak finding

## Evidence

- [readme] Arrays_intensity (HDF5 group): contains 1D arrays with intensity values.: [evidence span withheld — anchor status: external_doc]
- [readme] Example for spectrum with Scan value 630: Arrays_intensity/630, Arrays_mzbin/630.: [evidence span withheld — anchor status: external_doc]
