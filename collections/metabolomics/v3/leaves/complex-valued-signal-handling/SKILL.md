---
name: complex-valued-signal-handling
description: Use when the input is an unprocessed 1D time-domain FID of complex points
  (e.g., 16,384 complex samples) that must be truncated and randomly partitioned into
  train/validation/test subsets before peak-decomposition fitting, to avoid overfitting
  and infer a reasonable number of peaks.
when_to_use_negative:
- The input is already a processed frequency-domain spectrum or a peak/feature table
  — SAND's subsampling operates on the raw time-domain FID, not on spectra
- The signal is nonexponential or the data are multidimensional — SAND's current version
  is tuned for high-resolution 1D metabolomics spectra and does not yet natively accommodate
  these cases
- Strict k-fold cross-validation is required — the 70/20/10 split is intended to infer
  a reasonable number of peaks, not to serve as formal cross-validation
edam_operation: http://edamontology.org/operation_3218
tools:
- name: SAND
  role: Performs the complex-valued time-domain subsampling and Markov chain Monte
    Carlo decomposition into peak tables
  repo: github.com/edisonomics/SAND
- name: MATLAB
  role: Original SAND implementation in which the FID is loaded as native MATLAB complex
    arrays for truncation and partitioning (to be replaced by a faster C/C++ version)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:9dbce827644486112d09f90a7b3f10e6c36fa7622885d0a180c28e9d4af73df5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  - SAND
  edam_operation: http://edamontology.org/operation_3218
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# complex-valued-signal-handling

## Summary

Handles raw, complex-valued time-domain NMR FID points (real+imaginary pairs) prior to model-based decomposition in SAND: retain the early-decay region, subsample, and partition points so that Markov chain Monte Carlo fitting operates on complex amplitudes rather than phase-corrected spectra.

## When to use

Use when the input is an unprocessed 1D time-domain FID of complex points (e.g., 16,384 complex samples) that must be truncated and randomly partitioned into train/validation/test subsets before peak-decomposition fitting, to avoid overfitting and infer a reasonable number of peaks.

## When NOT to use

- The input is already a processed frequency-domain spectrum or a peak/feature table — SAND's subsampling operates on the raw time-domain FID, not on spectra
- The signal is nonexponential or the data are multidimensional — SAND's current version is tuned for high-resolution 1D metabolomics spectra and does not yet natively accommodate these cases
- Strict k-fold cross-validation is required — the 70/20/10 split is intended to infer a reasonable number of peaks, not to serve as formal cross-validation

## Inputs

- Binned time-domain FID of 16,384 complex points from an input dataset file (NumPy-loadable or MATLAB complex array)
- Optional truncation length (e.g., first 6,000 complex points) and random seed

## Outputs

- subsample_splits.csv with columns point_index and subset (train/validation/test) assigning each of the 6,000 subsampled complex points
- Mutually exclusive 70/20/10 partitions: 4,200 training, 1,200 validation, 600 testing complex points used for MCMC parameter optimization and stop-criteria checking

## How to apply

Load the binned time-domain FID as complex arrays (NumPy complex arrays for a reimplementation, or MATLAB native complex arrays for the original SAND codebase) — SAND operates on the raw complex points, not magnitude-only data. Optionally truncate the trailing-noise region: for the SAND workflow the FID of 16,384 complex points is cut to the first 6,000 complex points, retaining the early-time region where the exponentially decaying signal dominates. Randomly permute the 6,000 retained point indices with a fixed seed and split them 70/20/10 into training (4,200), validation (1,200), and testing (600) subsets; the training set drives parameter optimization, the validation set checks stop criteria, and the testing set gives final evaluation. The partitioning is intended to infer a reasonable number of peaks rather than serve as strict cross-validation. Write the per-point assignment to subsample_splits.csv (columns point_index, subset) so the subsampling in time and frequency domains is auditable and reproducible.

## Related tools

- **SAND** (Performs the complex-valued time-domain subsampling and Markov chain Monte Carlo decomposition into peak tables) — github.com/edisonomics/SAND
- **MATLAB** (Original SAND implementation in which the FID is loaded as native MATLAB complex arrays for truncation and partitioning (to be replaced by a faster C/C++ version))

## Examples

```
import numpy as np; fid = np.loadtxt('fid.csv', dtype=complex)[:6000]; idx = np.random.default_rng(seed=42).permutation(6000); train, val, test = idx[:4200], idx[4200:5400], idx[5400:]
```

## Evaluation signals

- The three partitions contain exactly 4,200 / 1,200 / 600 points
- Partitions are mutually exclusive and jointly cover all 6,000 subsampled points (every point_index appears once in subsample_splits.csv)
- Rerunning the split with the same fixed random seed reproduces identical point-to-subset assignments
- Truncation retains exactly the first 6,000 of the original 16,384 complex points, and the loaded arrays carry real and imaginary components (not magnitude-only values)

## Limitations

- Current SAND implementation is slow MATLAB code; a more readily parallelizable C/C++ version is planned but not yet available
- The current version of SAND is tuned for high-resolution metabolomics spectra; other cases are not yet accommodated
- SAND can potentially be expanded to include nonexponential signals and multidimensional applications, but these are not yet supported

## Evidence

- [abstract] SAND randomly divides the time-domain data into training and validation sets to help avoid overfitting.: "SAND randomly divides the time-domain data into training and validation sets to help avoid overfitting."
- [abstract] Since the SAND method uses time-domain subsampling (i.e., random subset of time-domain points), it has the potential to be extended to a higher dimensionality and nonuniformly sampled data.: "Since the SAND method uses time-domain subsampling (i.e., random subset of time-domain points), it has the potential to be extended"
- [abstract] The SAND approach uses hybrid optimization with Markov chain Monte Carlo methods, employing subsampling in both time and frequency domains.: "The SAND approach uses hybrid optimization with Markov chain Monte Carlo methods, employing subsampling in both time and frequency domains."
- [readme] SAND automatic process and deconvolute 1D NMR spectra into peak tables.: "SAND automatic process and deconvolute 1D NMR spectra into peak tables."
