---
name: gaussian-peak-modeling
description: Use when a 1D MS signal (e.g., XIC or ATD extracted from an MZA/HDF5
  file via mzapy) contains overlapping or noise-affected peaks that need Gaussian
  model fitting, peak-parameter estimation, or SNR-based peak quality assessment.
when_to_use_negative:
- The data is still raw multidimensional MS/IMS spectra — mzapy's peak functions operate
  on 1D signals; extract an XIC or ATD first (e.g., collect_xic_arrays_by_mz)
- The peaks are known to be non-Gaussian in shape (e.g., asymmetric chromatographic
  peaks), where find_peaks_1d_gauss model fits would be invalid
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzapy
  role: Provides the peaks module (find_peaks_1d_localmax, find_peaks_1d_gauss, calc_gauss_psnr,
    calc_peak_area, lerp_1d) used for peak detection, Gaussian fitting, and peak quantification
    on 1D MS signals
  repo: https://github.com/PNNL-m-q/mzapy
- name: scipy
  role: 'Underlying peak-detection engine: find_peaks_1d_localmax uses scipy.signal.find_peaks
    internally'
- name: numpy
  role: Generates synthetic Gaussian test signals with a seeded RNG and supplies array
    handling for signals and peak indices
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:c2c01171e7e7c49880246ca8141dca1358889a09200cb1d575c798ce76bda763
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gaussian-peak-modeling

## Summary

Fit and validate Gaussian peak models on 1D mass spectrometry signals (e.g., extracted ion chromatograms or arrival time distributions from MZA/HDF5 files) using mzapy's peaks module, which includes find_peaks_1d_gauss, calc_gauss_psnr, and calc_peak_area alongside local-maximum peak detection.

## When to use

Use when a 1D MS signal (e.g., XIC or ATD extracted from an MZA/HDF5 file via mzapy) contains overlapping or noise-affected peaks that need Gaussian model fitting, peak-parameter estimation, or SNR-based peak quality assessment.

## When NOT to use

- The data is still raw multidimensional MS/IMS spectra — mzapy's peak functions operate on 1D signals; extract an XIC or ATD first (e.g., collect_xic_arrays_by_mz)
- The peaks are known to be non-Gaussian in shape (e.g., asymmetric chromatographic peaks), where find_peaks_1d_gauss model fits would be invalid

## Inputs

- 1D intensity array vs. m/z, retention time, or arrival time axis (e.g., from Arrays_mz/Arrays_intensity datasets in an MZA HDF5 file)
- Ground-truth peak parameters (x positions, heights) for synthetic validation signals
- Peak-detection tolerances on x position and height

## Outputs

- Fitted Gaussian peak parameters (position, height, width) per detected peak
- Peak list table (e.g., peak_list.csv with columns index, x, height)
- Per-peak quality metrics (PSNR from calc_gauss_psnr) and integrated areas (calc_peak_area)

## How to apply

Generate or extract the 1D signal (e.g., a sum of resolved Gaussian peaks on an m/z-like axis with a small noise floor, or a real XIC collected with collect_xic_arrays_by_mz from MZA data). Run mzapy.peaks.find_peaks_1d_localmax, which uses scipy.signal.find_peaks internally, to obtain initial local-maximum detections, then apply mzapy.peaks.find_peaks_1d_gauss to refine detected peaks with a fitted Gaussian model. Use calc_gauss_psnr to score peak quality and calc_peak_area to quantify integrated peak area; lerp_1d/lerp_2d provide linear interpolation of signals as needed. Map fitted indices back to x positions and heights and compare against ground-truth or expected peak parameters within a defined tolerance on position and height.

## Related tools

- **mzapy** (Provides the peaks module (find_peaks_1d_localmax, find_peaks_1d_gauss, calc_gauss_psnr, calc_peak_area, lerp_1d) used for peak detection, Gaussian fitting, and peak quantification on 1D MS signals) — https://github.com/PNNL-m-q/mzapy
- **scipy** (Underlying peak-detection engine: find_peaks_1d_localmax uses scipy.signal.find_peaks internally)
- **numpy** (Generates synthetic Gaussian test signals with a seeded RNG and supplies array handling for signals and peak indices)

## Examples

```
from mzapy.peaks import find_peaks_1d_localmax, find_peaks_1d_gauss, calc_peak_area; idx = find_peaks_1d_localmax(y); peaks = find_peaks_1d_gauss(x, y, idx)
```

## Evaluation signals

- peak_list.csv exists with exactly the expected number of rows (e.g., 5 for a 5-peak synthetic signal)
- Every recovered x position matches a ground-truth peak position within the specified tolerance
- Recovered heights match ground-truth Gaussian amplitudes within tolerance on a noise-floor synthetic signal
- Each detected peak has a well-formed Gaussian fit yielding a finite PSNR from calc_gauss_psnr
- calc_peak_area returns integrated areas consistent with the fitted Gaussian parameters

## Limitations

- find_peaks_1d_localmax detects only local maxima, so shoulder peaks without a local maximum or heavily overlapped peaks may be missed or merged
- Gaussian fitting assumes peaks are well-resolved relative to the noise floor; low-SNR peaks may fail fitting or return unstable parameters

## Evidence
