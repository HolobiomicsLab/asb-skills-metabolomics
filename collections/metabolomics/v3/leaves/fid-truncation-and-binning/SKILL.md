---
name: fid-truncation-and-binning
description: Use when preparing a binned time-domain FID (e.g., 16,384 complex points)
  for SAND decomposition, before optimization iterations, to curb overfitting via
  FID truncation and randomized 70/20/10 train/validation/test subsampling.
when_to_use_negative:
- Input is already a processed frequency-domain peak/feature table rather than a raw
  or binned time-domain FID
- Late-time FID points carry meaningful signal rather than trailing noise, so discarding
  points after index 6,000 would lose information
- The goal is strict cross-validated model selection; SAND's partitioning is explicitly
  meant to infer a reasonable number of peaks, not to serve as cross-validation
tools:
- name: SAND
  role: Decomposition pipeline whose time-domain subsampling consumes the truncated
    and partitioned FID points
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Original SAND implementation environment; native MATLAB complex arrays hold
    the binned FID during truncation and splitting
- name: NumPy
  role: Alternative environment for reimplementing the FID loading, truncation, seeded
    permutation, and 70/20/10 split
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:27a4efb5bc75020468057088d635905050d0052669598aa39c6b130f34d8826d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NumPy
  - SAND
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fid-truncation-and-binning

## Summary

A pre-decomposition subsampling step from the SAND workflow: the binned time-domain FID (16,384 complex points) is truncated to the early-time signal-dominated region (first 6,000 points) and the selected points are randomly partitioned into 70/20/10 training/validation/test subsets to reduce overfitting during MCMC optimization.

## When to use

Use when preparing a binned time-domain FID (e.g., 16,384 complex points) for SAND decomposition, before optimization iterations, to curb overfitting via FID truncation and randomized 70/20/10 train/validation/test subsampling.

## When NOT to use

- Input is already a processed frequency-domain peak/feature table rather than a raw or binned time-domain FID
- Late-time FID points carry meaningful signal rather than trailing noise, so discarding points after index 6,000 would lose information
- The goal is strict cross-validated model selection; SAND's partitioning is explicitly meant to infer a reasonable number of peaks, not to serve as cross-validation

## Inputs

- Binned time-domain FID with 16,384 complex points (input dataset file)
- Fixed random seed for reproducible permutation
- Truncation length (6,000 complex points)

## Outputs

- Truncated FID of the first 6,000 complex points
- subsample_splits.csv with columns point_index and subset (train/validation/test) for all 6,000 subsampled points
- Training (4,200), validation (1,200), and testing (600) point subsets consumed by SAND's MCMC parameter optimization and stop-criteria checking

## How to apply

Load the binned time-domain FID containing 16,384 complex points (NumPy complex arrays for a reimplementation, MATLAB native complex arrays for the original SAND codebase). Truncate to the first 6,000 complex points, retaining the early-time region where the exponentially decaying signal dominates and trailing noise is discarded. Randomly permute the 6,000 point indices with a fixed random seed, then split into 70% training (4,200 points) for parameter optimization, 20% validation (1,200 points) for stop-criteria checking, and 10% testing (600 points) for final evaluation. Write each point's assignment to subsample_splits.csv with columns point_index and subset. The partitioning is intended to help SAND infer a reasonable number of peaks rather than serve as strict cross-validation, consistent with SAND's hybrid MCMC optimization that subsamples in both time and frequency domains.

## Related tools

- **SAND** (Decomposition pipeline whose time-domain subsampling consumes the truncated and partitioned FID points) — https://github.com/edisonomics/SAND
- **MATLAB** (Original SAND implementation environment; native MATLAB complex arrays hold the binned FID during truncation and splitting)
- **NumPy** (Alternative environment for reimplementing the FID loading, truncation, seeded permutation, and 70/20/10 split)

## Evaluation signals

- The three partitions contain exactly 4,200 / 1,200 / 600 points (70/20/10 of the 6,000 subsampled points)
- The three partitions are mutually exclusive and jointly cover all 6,000 subsampled points
- A rerun with the same random seed reproduces identical point-to-subset assignments
- No point index ≥ 6,000 appears in any partition, confirming trailing-noise truncation
- subsample_splits.csv contains exactly the columns point_index and subset, with subset values limited to train/validation/test

## Limitations

- The 70/20/10 partitioning is intended to help infer a reasonable number of peaks, not to serve as strict cross-validation
- SAND is currently tuned for high-resolution metabolomics spectra and a slow MATLAB implementation not yet parallelizable; other cases and C/C++ are future work
- Because SAND uses time-domain subsampling (random subset of time-domain points), extension to higher dimensionality and nonuniformly sampled data remains potential rather than implemented

## Evidence

- [abstract] SAND randomly divides the time-domain data into training and validation sets to help avoid overfitting.: "SAND randomly divides the time-domain data into training and validation sets to help avoid overfitting."
- [abstract] Since the SAND method uses time-domain subsampling (i.e., random subset of time-domain points), it has the potential to be extended to a higher dimensionality and nonuniformly sampled data.: "Since the SAND method uses time-domain subsampling (i.e., random subset of time-domain points), it has the potential to be extended to a higher dimensionality and nonuniformly sampled data."
- [readme] SAND automatic process and deconvolute 1D NMR spectra into peak tables. Similar workflow works for simulated and expermental datasets.: "SAND automatic process and deconvolute 1D NMR spectra into peak tables. Similar workflow works for simulated and expermental datasets."
