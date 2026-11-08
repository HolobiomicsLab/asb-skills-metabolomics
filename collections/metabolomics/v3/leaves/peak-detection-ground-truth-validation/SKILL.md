---
name: peak-detection-ground-truth-validation
description: Use when you need to verify that a peak-finding routine (e.g. mzapy.peaks.find_peaks_1d_localmax
  or find_peaks_1d_gauss) detects the correct number of peaks with accurate x positions
  and heights before trusting it on real MS data.
when_to_use_negative:
- The signal is raw multidimensional IMS-MS data (2D RT × m/z or arrival-time arrays)
  — this skill validates 1D local-maximum peak finding, not 2D feature detection over
  MZA/HDF5 spectra
- The goal is quantifying peak area or fitting Gaussian parameters (use mzapy.peaks.calc_peak_area
  or find_peaks_1d_gauss instead of index-based local-maximum validation)
- Peaks in the real signal are not well-resolved; the synthetic ground-truth validation
  assumes resolved maxima that local-maximum detection can separate
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzapy
  role: Provides the peak-finding utilities being validated, notably mzapy.peaks.find_peaks_1d_localmax,
    exposed alongside find_peaks_1d_gauss, lerp_1d, and calc_peak_area
  repo: https://github.com/PNNL-m-q/mzapy
- name: numpy
  role: Generates the seeded synthetic Gaussian peak signal and noise floor, and maps
    detected peak indices to x positions and heights
- name: scipy
  role: 'Underlies the detection step: find_peaks_1d_localmax uses scipy.signal.find_peaks
    internally to detect local-maximum peaks'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:c88a964268774a28994178de54a19cf8450fb6bb71e4953e1a2b47894ce6bcc6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# peak-detection-ground-truth-validation

## Summary

Validates a peak-finding implementation by generating a synthetic 1D signal of known Gaussian peaks (e.g. on an m/z-like axis), running mzapy.peaks.find_peaks_1d_localmax on it, and comparing recovered peak indices/positions/heights against the recorded ground truth within a specified tolerance.

## When to use

Use when you need to verify that a peak-finding routine (e.g. mzapy.peaks.find_peaks_1d_localmax or find_peaks_1d_gauss) detects the correct number of peaks with accurate x positions and heights before trusting it on real MS data.

## When NOT to use

- The signal is raw multidimensional IMS-MS data (2D RT × m/z or arrival-time arrays) — this skill validates 1D local-maximum peak finding, not 2D feature detection over MZA/HDF5 spectra
- The goal is quantifying peak area or fitting Gaussian parameters (use mzapy.peaks.calc_peak_area or find_peaks_1d_gauss instead of index-based local-maximum validation)
- Peaks in the real signal are not well-resolved; the synthetic ground-truth validation assumes resolved maxima that local-maximum detection can separate

## Inputs

- Synthetic 1D signal (numpy array): sum of 5 well-resolved Gaussian peaks at known x positions and heights on an m/z-like axis plus a small noise floor, generated with a seeded numpy RNG
- Ground-truth peak parameter list (x positions and heights of the synthetic Gaussians)
- Tolerances on x position and peak height for match validation

## Outputs

- peak_list.csv with columns index, x, height containing validated peak detections
- Per-peak match report comparing detected vs. ground-truth x positions and heights within tolerance

## How to apply

Generate a synthetic 1D signal with a seeded numpy RNG as a sum of well-resolved Gaussian peaks at known x positions and heights on an m/z-like axis plus a small noise floor, and record the ground-truth peak parameters. Run mzapy.peaks.find_peaks_1d_localmax on the synthetic signal — this utility wraps scipy.signal.find_peaks to detect local-maximum peaks. Map the returned peak indices back to x positions and heights, then compare each detected peak against the ground-truth list using explicit tolerances on x position and height. Write validated detections to peak_list.csv with columns index, x, height. Success requires exactly the expected number of rows and every recovered x position matching a ground-truth position within tolerance; this separates algorithmic failures (missed/false peaks) from parameter-tuning issues.

## Related tools

- **mzapy** (Provides the peak-finding utilities being validated, notably mzapy.peaks.find_peaks_1d_localmax, exposed alongside find_peaks_1d_gauss, lerp_1d, and calc_peak_area) — https://github.com/PNNL-m-q/mzapy
- **numpy** (Generates the seeded synthetic Gaussian peak signal and noise floor, and maps detected peak indices to x positions and heights)
- **scipy** (Underlies the detection step: find_peaks_1d_localmax uses scipy.signal.find_peaks internally to detect local-maximum peaks)

## Examples

```
import numpy as np; from mzapy.peaks import find_peaks_1d_localmax; x = np.linspace(100, 200, 10001); signal = sum(h*np.exp(-(x-m)**2/(2*w**2)) for m,h,w in truths) + rng.normal(0, 5, x.size); idx, props = find_peaks_1d_localmax(x, signal)
```

## Evaluation signals

- peak_list.csv exists and contains exactly 5 rows (one per synthetic ground-truth peak)
- Every recovered x position in peak_list.csv matches a ground-truth peak position within the specified tolerance
- Recovered heights match the known Gaussian amplitudes within the height tolerance
- No false-positive peaks are reported for regions of the signal that contain only the noise floor
- Results are reproducible across runs given the same seeded RNG for signal generation

## Limitations

- Validation only holds for well-resolved peaks; overlapping or coalescing Gaussian peaks can merge into a single local maximum and fail the ground-truth match
- The synthetic noise floor must stay small relative to peak heights, otherwise local-maximum detection can produce spurious peaks in noise
- find_peaks_1d_localmax is a local-maximum detector on 1D signals; it does not perform Gaussian fitting or peak-area integration (those are handled by find_peaks_1d_gauss and calc_peak_area)

## Evidence
