---
name: overfitting-control-in-mcmc-spectral-modeling
description: Use when fitting time-domain 1D NMR FID data (e.g., 16,384 complex points)
  with an MCMC or iterative peak-decomposition model and you need to infer a reasonable
  number of peaks without overfitting.
when_to_use_negative:
- The goal is rigorous k-fold cross-validation or generalization-error estimation
  — the article states this partitioning is intended to infer a reasonable number
  of peaks, not to serve as strict cross-validation.
- Input is already a processed/feature table or frequency-domain peak list rather
  than raw time-domain FID points requiring subsampling.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3420
tools:
- name: SAND
  role: Performs the hybrid MCMC decomposition with time- and frequency-domain subsampling;
    the train/validation split is part of its resampling module to avoid overfitting
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Original SAND implementation using MATLAB native complex arrays for the FID
    and the partitioning step
- name: NMRbox
  role: Cloud environment through which SAND (and hence this resampling workflow)
    is run via a single command
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:0e7725fe4af4bf1632c7bca3f48a3e2af769dd41b673ee7abacda819af3cf083
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/edisonomics/SAND
  tools:
  - MATLAB
  - NMRbox
  - SAND
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# overfitting-control-in-mcmc-spectral-modeling

## Summary

Randomly partitions binned time-domain FID points (optionally after truncating trailing-noise points) into 70/20/10 training/validation/testing sets so that SAND's Markov chain Monte Carlo decomposition of 1D NMR spectra is optimized, stopped, and evaluated on disjoint data, avoiding overfitting of the peak model.

## When to use

Use when fitting time-domain 1D NMR FID data (e.g., 16,384 complex points) with an MCMC or iterative peak-decomposition model and you need to infer a reasonable number of peaks without overfitting.

## When NOT to use

- The goal is rigorous k-fold cross-validation or generalization-error estimation — the article states this partitioning is intended to infer a reasonable number of peaks, not to serve as strict cross-validation.
- Input is already a processed/feature table or frequency-domain peak list rather than raw time-domain FID points requiring subsampling.

## Inputs

- binned time-domain FID of 16,384 complex points (NumPy complex array or MATLAB complex array)
- optional truncation length (e.g., first 6,000 complex points)
- fixed random seed for the permutation

## Outputs

- subsample_splits.csv with columns point_index and subset (train/validation/test)
- 70/20/10 partitions of the subsampled time-domain points feeding SAND's MCMC parameter optimization, stop-criteria checking, and final evaluation

## How to apply

Load the binned complex time-domain FID (16,384 complex points in the SAND workflow) and optionally truncate trailing-noise points — the SAND workflow keeps the first 6,000 points where the exponentially decaying signal dominates. Randomly permute the selected point indices with a fixed random seed and split them into a 70% training set used for parameter optimization (e.g., 4,200 of 6,000), a 20% validation set used for stop-criteria checking (1,200), and a 10% testing set for final evaluation (600). The partitioning is intended to infer a reasonable number of peaks rather than serve as strict cross-validation, and random subsampling in both time and frequency domains is part of SAND's hybrid MCMC optimization. Write the per-point subset assignment to subsample_splits.csv (columns point_index, subset) so the split is auditable and reproducible under the same seed.

## Related tools

- **SAND** (Performs the hybrid MCMC decomposition with time- and frequency-domain subsampling; the train/validation split is part of its resampling module to avoid overfitting) — https://github.com/edisonomics/SAND
- **MATLAB** (Original SAND implementation using MATLAB native complex arrays for the FID and the partitioning step)
- **NMRbox** (Cloud environment through which SAND (and hence this resampling workflow) is run via a single command)

## Examples

```
idx = np.random.default_rng(42).permutation(6000); splits = np.array(['train']*4200 + ['validation']*1200 + ['test']*600)[np.argsort(idx.argsort())]
```

## Evaluation signals

- The three partitions contain exactly 4,200 / 1,200 / 600 points when 6,000 points are subsampled.
- Partitions are mutually exclusive and jointly cover all 6,000 subsampled points.
- Rerunning with the same random seed reproduces identical subset assignments in subsample_splits.csv.
- subsample_splits.csv has exactly two columns, point_index and subset, with subset values in {train, validation, test}.
- Truncation retains only the first 6,000 of 16,384 complex points, i.e., the early-time region where the exponentially decaying signal dominates.

## Limitations

- The 70/20/10 split is for inferring a reasonable number of peaks, not strict cross-validation; it should not be reported as a generalization estimate.
- SAND is currently tuned only for high-resolution metabolomics spectra and implemented in slow, non-parallelizable MATLAB; other spectral cases are not yet accommodated.
- Truncating the FID (e.g., to 6,000 points) discards later points, which is appropriate when the tail is noise-dominated but may drop usable signal for slowly decaying resonances.

## Evidence

- [full_text] The resampling module optionally truncates trailing-noise FID points, then randomly assigns the selected points into a 70% training set for parameter optimization, a 20% validation set for stop-criteria checking, and a 10% testing set for final evaluation; this partitioning is intended to infer a reasonable number of peaks rather than serve as strict cross-validation.: [evidence span withheld — anchor status: not_found]
