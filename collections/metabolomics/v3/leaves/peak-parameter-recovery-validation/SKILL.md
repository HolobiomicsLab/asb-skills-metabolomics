---
name: peak-parameter-recovery-validation
description: Use when validating that a peak-finding/peak-fitting function recovers
  correct peak positions, intensities, and widths — e.g. testing find_peaks_1d_gauss
  on synthetic 1D signals before applying it to real chromatographic or IMS data.
when_to_use_negative:
- Input is raw vendor-format MS data (e.g. Agilent '.d', Thermo '.raw', mzML) that
  first needs conversion to MZA/HDF5 via the mza converter before any peak fitting
  can run
- Signals are noisy, poorly resolved, or overlapping peaks where sequential residual
  fitting is not expected to yield reliable parameter estimates
- Peak finding on 2D or IMS-resolved data — find_peaks_1d_gauss applies only to 1D
  arrays such as extracted ion chromatograms or arrival time distributions
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzapy
  role: Provides mzapy.peaks.find_peaks_1d_gauss for sequential least-squares Gaussian
    peak fitting on 1D signals
  repo: https://github.com/PNNL-m-q/mzapy
- name: Python
  role: Implementation language for building synthetic Gaussian signals, running fits,
    and computing parameter errors
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.3c01653
    title: '<i>mzapy</i>: An Open-Source Python Library Enabling Efficient Extraction
      and Processing of Ion Mobility Spectrometry-Mass Spectrometry Data in the MZA
      File Format'
  source_content_hash: sha256:47a595f83bd26c6ca786403d98070d60bb3c6594364ba67dd4d3e3aebe887203
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PNNL-m-q/mzapy
  tools:
  - Python
  - mzapy
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# peak-parameter-recovery-validation

## Summary

Benchmark an iterative Gaussian peak-finding routine (mzapy.peaks.find_peaks_1d_gauss) by fitting synthetic 1D signals built from Gaussians with known center, height, and width, then comparing fitted parameters against ground truth to quantify recovery error.

## When to use

Use when validating that a peak-finding/peak-fitting function recovers correct peak positions, intensities, and widths — e.g. testing find_peaks_1d_gauss on synthetic 1D signals before applying it to real chromatographic or IMS data.

## When NOT to use

- Input is raw vendor-format MS data (e.g. Agilent '.d', Thermo '.raw', mzML) that first needs conversion to MZA/HDF5 via the mza converter before any peak fitting can run
- Signals are noisy, poorly resolved, or overlapping peaks where sequential residual fitting is not expected to yield reliable parameter estimates
- Peak finding on 2D or IMS-resolved data — find_peaks_1d_gauss applies only to 1D arrays such as extracted ion chromatograms or arrival time distributions

## Inputs

- Synthetic 1D signals (numpy arrays) built as sums of Gaussians with known center, height, and width
- Sampling grids at dense and sparse point spacings
- Stopping-criterion parameters for find_peaks_1d_gauss (minimum peak height threshold, absolute or relative; max number of peaks)

## Outputs

- fitted_peak_parameters.csv containing ground-truth and fitted peak parameters (center, height, width) with absolute/relative errors per peak and per parameter
- Per-peak fitted parameter estimates: mean x value, height, width

## How to apply

Construct synthetic 1D signals by summing Gaussian functions with known center (mean x), height, and width, sampled at both dense and sparse point spacings to probe resolution sensitivity. Run mzapy.peaks.find_peaks_1d_gauss, which performs successive rounds of least-squares Gaussian fits on the residuals of the previous fit until a stopping criterion is met — peak height below an absolute or relative threshold, or a maximum number of peaks found. Extract the fitted parameters (mean x value, height, width) for every recovered peak. Compare fitted values against ground truth per peak and per parameter, computing absolute and relative errors, and write ground-truth plus fitted parameters with errors to fitted_peak_parameters.csv. Acceptance is that fitted centers, heights, and widths recover ground truth within reasonable tolerance for clean, well-resolved synthetic peaks.

## Related tools

- **mzapy** (Provides mzapy.peaks.find_peaks_1d_gauss for sequential least-squares Gaussian peak fitting on 1D signals) — https://github.com/PNNL-m-q/mzapy
- **Python** (Implementation language for building synthetic Gaussian signals, running fits, and computing parameter errors)

## Examples

```
from mzapy.peaks import find_peaks_1d_gauss; x, y = build_synthetic_gaussians(); peaks = find_peaks_1d_gauss(x, y, min_height_threshold, max_n_peaks)
```

## Evaluation signals

- Fitted centers, heights, and widths recover ground-truth Gaussian values within reasonable tolerance on clean, well-resolved synthetic peaks
- Number of peaks recovered matches the number of Gaussians summed into the synthetic signal (no spurious or missed peaks) until the stopping criterion triggers
- Absolute and relative errors per peak/parameter are tabulated and small for both dense and sparse sampling grids
- fitted_peak_parameters.csv is produced with columns for ground-truth values, fitted values, and errors

## Limitations

- Reliable parameter estimates are expected for clean, well-formed, well-resolved synthetic peaks; performance on noisy or co-eluting real-world peaks is not established by this validation
- Sequential residual fitting stops on a peak-height threshold (absolute or relative) or max-peak count, so small or weak peaks may be missed depending on threshold choice
- Sparse sampling degrades recovery accuracy compared with dense sampling

## Evidence

- [methods] successive rounds of least-squares Gaussian fits on the residuals of the previous fit until a stopping criterion is reached: [evidence span withheld — anchor status: not_found]
