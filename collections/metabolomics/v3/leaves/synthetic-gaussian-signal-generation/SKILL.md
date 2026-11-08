---
name: synthetic-gaussian-signal-generation
description: 'Use when validating mzapy.peaks peak finders against signals of known
  composition: generate summed-Gaussian 1D signals with recorded ground-truth parameters
  to quantify detection and parameter-recovery error.'
when_to_use_negative:
- The input is real experimental MZA/HDF5 mass spectrometry data where peaks are unknown
  — synthetic generation is for validation, not for analyzing actual LC-IMS-MS signals
- The target peaks are heavily overlapping or poorly resolved, since the documented
  reliability of sequential Gaussian fitting holds for clean, well-resolved signals
- Evaluating mzapy capabilities unrelated to 1D peak finding, e.g. MZA conversion,
  mass/CCS calibration, or isotopic m/z prediction
tools:
- name: mzapy
  role: Function under test — its find_peaks_1d_gauss and find_peaks_1d_localmax peak
    finders are run on the synthetic signals and their outputs validated
  repo: https://github.com/PNNL-m-q/mzapy
- name: numpy
  role: Generates the seeded RNG noise floor and the summed-Gaussian synthetic signal
    arrays
- name: scipy
  role: Internal dependency of find_peaks_1d_localmax (uses scipy.signal.find_peaks)
    for local-maximum detection
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:6641c52895cd87400e76e7a38640725627b35fe4a0b647ee61c7e9d94f2725b2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  - numpy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# synthetic-gaussian-signal-generation

## Summary

Builds synthetic 1D signals as sums of Gaussian peaks with known center, height, and width (plus a controlled noise floor) on an m/z-like axis, to serve as ground-truth test inputs for validating mzapy peak-finding utilities such as find_peaks_1d_gauss and find_peaks_1d_localmax.

## When to use

Use when validating mzapy.peaks peak finders against signals of known composition: generate summed-Gaussian 1D signals with recorded ground-truth parameters to quantify detection and parameter-recovery error.

## When NOT to use

- The input is real experimental MZA/HDF5 mass spectrometry data where peaks are unknown — synthetic generation is for validation, not for analyzing actual LC-IMS-MS signals
- The target peaks are heavily overlapping or poorly resolved, since the documented reliability of sequential Gaussian fitting holds for clean, well-resolved signals
- Evaluating mzapy capabilities unrelated to 1D peak finding, e.g. MZA conversion, mass/CCS calibration, or isotopic m/z prediction

## Inputs

- Ground-truth Gaussian parameter list (center, height, width per peak, e.g. 5 well-resolved peaks on an m/z-like axis)
- Axis sampling specification: dense and sparse point spacings over the m/z-like range
- Noise floor magnitude and a numpy RNG seed for reproducibility

## Outputs

- Synthetic 1D signal arrays (x axis and summed-Gaussian intensity with noise)
- Ground-truth peak parameter table (center, height, width)
- fitted_peak_parameters.csv with ground-truth and fitted parameters plus absolute/relative errors per peak and per parameter
- peak_list.csv with columns index, x, height for local-maximum detections

## How to apply

Sample a 1D m/z-like axis at dense and sparse point spacings, then sum Gaussian functions with pre-recorded center, height, and width values, adding a small noise floor drawn from a seeded numpy RNG for reproducibility. Feed the synthetic signal to the function under test — mzapy.peaks.find_peaks_1d_gauss, which performs successive rounds of least-squares Gaussian fits on the residuals of the previous fit until a stopping criterion (peak height below an absolute or relative threshold, or a maximum number of peaks) is reached, or mzapy.peaks.find_peaks_1d_localmax, which wraps scipy.signal.find_peaks. Extract fitted peak parameters (mean x value, height, width) or detected indices, map indices back to x positions and heights, and compare against the ground truth within a stated tolerance on x position and height. Compute absolute/relative errors per peak and per parameter and persist ground-truth plus fitted values to CSV (e.g. fitted_peak_parameters.csv, peak_list.csv). The rationale is that clean, well-resolved synthetic peaks give an upper-bound check that the documented fitting/detection behavior recovers known parameters before applying the functions to real MZA-derived LC-IMS-MS signals.

## Related tools

- **mzapy** (Function under test — its find_peaks_1d_gauss and find_peaks_1d_localmax peak finders are run on the synthetic signals and their outputs validated) — https://github.com/PNNL-m-q/mzapy
- **numpy** (Generates the seeded RNG noise floor and the summed-Gaussian synthetic signal arrays)
- **scipy** (Internal dependency of find_peaks_1d_localmax (uses scipy.signal.find_peaks) for local-maximum detection)

## Examples

```
import numpy as np; rng = np.random.default_rng(seed); y = sum(h*np.exp(-0.5*((x-mu)/w)**2) for mu,h,w in gt) + rng.normal(0, sigma, x.size)
```

## Evaluation signals

- peak_list.csv contains exactly the expected number of rows (e.g. 5 for a 5-peak synthetic signal)
- Every recovered x position matches a ground-truth peak position within the specified tolerance
- Fitted centers, heights, and widths recover ground-truth values within reasonable absolute/relative error tolerance for well-formed synthetic peaks
- find_peaks_1d_gauss stopping behavior is respected: fitting halts when peak height falls below the absolute/relative threshold or the maximum peak count is reached
- Both dense and sparse axis spacings are tested and parameter recovery is reported per spacing

## Limitations

- Sequential Gaussian fitting produces reliable parameter estimates primarily for clean, well-resolved signals; overlapping or noisy peaks are outside the validated regime
- Synthetic well-formed Gaussian peaks do not capture the heterogeneity of real multidimensional IMS-MS spectra stored in MZA format
- Findings hold for 1D signals; extension to 2D processing would require other mzapy utilities such as lerp_2d

## Evidence

- [readme] Python example scripts require numpy among dependencies: [evidence span withheld — anchor status: external_doc]
