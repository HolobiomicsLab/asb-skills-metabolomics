---
name: train-validation-test-partitioning
description: Use when fitting time-domain NMR decomposition models (e.g., SAND on
  binned FID complex points) and you need to prevent overfitting during Markov chain
  Monte Carlo parameter optimization by holding out point subsets.
when_to_use_negative:
- Input is already a processed frequency-domain peak table or feature table rather
  than raw time-domain FID points
- The analysis requires strict k-fold cross-validation for generalization estimates;
  SAND's 70/20/10 split is explicitly intended to infer a reasonable number of peaks,
  not to serve as strict cross-validation
tools:
- name: SAND
  role: Workflow that consumes the train/validation/test partitioning during hybrid
    MCMC decomposition of time-domain NMR data
  repo: github.com/edisonomics/SAND
- name: MATLAB
  role: Original SAND codebase language in which the complex-array resampling/partitioning
    is implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:94682255059791de1ac983560855016a0bd6c243f577f7e83fd11d15a3314c73
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MATLAB
  - SAND
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# train-validation-test-partitioning

## Summary

Randomly partitions subsampled time-domain FID points into 70% training, 20% validation, and 10% testing sets within the SAND workflow to reduce overfitting and infer a reasonable number of peaks.

## When to use

Use when fitting time-domain NMR decomposition models (e.g., SAND on binned FID complex points) and you need to prevent overfitting during Markov chain Monte Carlo parameter optimization by holding out point subsets.

## When NOT to use

- Input is already a processed frequency-domain peak table or feature table rather than raw time-domain FID points
- The analysis requires strict k-fold cross-validation for generalization estimates; SAND's 70/20/10 split is explicitly intended to infer a reasonable number of peaks, not to serve as strict cross-validation

## Inputs

- Binned time-domain FID of 16,384 complex points from the input dataset file (NumPy complex array or MATLAB native complex array)
- Optional truncation parameter (e.g., first 6,000 complex points) and a fixed random seed

## Outputs

- subsample_splits.csv with columns point_index and subset (train/validation/test) for each of the 6,000 subsampled points
- Training (4,200 points), validation (1,200 points), and testing (600 points) point-index subsets

## How to apply

Load the binned time-domain FID (e.g., 16,384 complex points), then optionally truncate trailing-noise points (e.g., keep the first 6,000 where the exponentially decaying signal dominates). Randomly permute the selected point indices with a fixed random seed and split them into 70% training (4,200 points) for parameter optimization, 20% validation (1,200 points) for stop-criteria checking, and 10% testing (600 points) for final evaluation. The rationale, per the SAND paper, is that randomly dividing the time-domain data helps avoid overfitting during hybrid MCMC optimization with time-domain subsampling; the authors note this partitioning is intended to infer a reasonable number of peaks rather than serve as strict cross-validation. Record the subset assignment for each point so the split is auditable and reproducible under the same seed.

## Related tools

- **SAND** (Workflow that consumes the train/validation/test partitioning during hybrid MCMC decomposition of time-domain NMR data) — github.com/edisonomics/SAND
- **MATLAB** (Original SAND codebase language in which the complex-array resampling/partitioning is implemented)

## Examples

```
python scripts/resample_fid.py --fid fid_16384.npy --truncate 6000 --seed 42 --splits 70/20/10 --output subsample_splits.csv
```

## Evaluation signals

- The three partitions contain exactly 4,200 / 1,200 / 600 points when 6,000 subsampled points are used
- Partitions are mutually exclusive and jointly cover all 6,000 subsampled points
- A rerun with the same fixed random seed reproduces identical point-to-subset assignments
- subsample_splits.csv schema is valid: point_index and subset columns, with subset limited to train/validation/test

## Limitations

- The 70/20/10 partitioning is intended to infer a reasonable number of peaks, not to serve as strict cross-validation
- SAND (and thus this partitioning scheme) is currently tuned for high-resolution metabolomics spectra; the MATLAB implementation is slow and not yet parallelizable, with a C/C++ replacement planned

## Evidence

- [other] Truncate the FID to the first 6,000 complex points, retaining the early-time region where the exponentially decaying signal dominates.: [evidence span withheld — anchor status: not_found]
- [other] Randomly permute the 6,000 point indices with a fixed random seed and split them into 70% training (4,200 points), 20% validation (1,200 points), and 10% testing (600 points) subsets.: [evidence span withheld — anchor status: not_found]
