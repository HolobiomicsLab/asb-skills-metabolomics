---
name: time-domain-subsampling-of-nmr-fids
description: Use when preparing a binned time-domain FID (e.g., 16,384 complex points)
  for model-based spectral decomposition where overfitting must be controlled before
  optimization iterations begin.
when_to_use_negative:
- Input is already processed frequency-domain spectra or a peak/feature table rather
  than raw time-domain FID points — this subsampling applies only before decomposition
  iterations
- The goal is strict k-fold cross-validation of a predictive model; the paper explicitly
  states this 70/20/10 split infers a reasonable number of peaks rather than serving
  as strict cross-validation
- Data is multidimensional or nonuniformly sampled — the paper notes SAND's extension
  to these cases is a future possibility, not yet supported
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: SAND
  role: Decomposition software whose workflow consumes the subsampled, partitioned
    time-domain points during hybrid MCMC optimization
  repo: https://github.com/edisonomics/SAND
- name: MATLAB
  role: Original implementation language in which the SAND resampling/partitioning
    module operates on complex FID arrays
- name: NMRbox
  role: Cloud environment where SAND (and thus the subsampling workflow) is hosted
    and run via a single terminal command
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c03078
    title: 'SAND: Automated Time-Domain Modeling of NMR Spectra Applied to Metabolite
      Quantification'
  source_content_hash: sha256:ef3e3b4b181a6575f05ec309cfbbeca99b97a95c5d7ee74bb340047d9a611ff7
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# time-domain-subsampling-of-nmr-fids

## Summary

Random subsampling and partitioning of time-domain NMR FID points into 70% training, 20% validation, and 10% testing sets, as used in the SAND workflow to reduce overfitting during Markov chain Monte Carlo decomposition.

## When to use

Use when preparing a binned time-domain FID (e.g., 16,384 complex points) for model-based spectral decomposition where overfitting must be controlled before optimization iterations begin.

## When NOT to use

- Input is already processed frequency-domain spectra or a peak/feature table rather than raw time-domain FID points — this subsampling applies only before decomposition iterations
- The goal is strict k-fold cross-validation of a predictive model; the paper explicitly states this 70/20/10 split infers a reasonable number of peaks rather than serving as strict cross-validation
- Data is multidimensional or nonuniformly sampled — the paper notes SAND's extension to these cases is a future possibility, not yet supported

## Inputs

- Binned time-domain FID of 1D NMR data (e.g., 16,384 complex points) loaded from the input dataset file
- Optional truncation parameter defining the number of retained early-time complex points (e.g., 6,000)
- Fixed random seed for reproducible permutation

## Outputs

- Training subset (~70% of subsampled points, e.g., 4,200) used for parameter optimization
- Validation subset (~20%, e.g., 1,200) used for stop-criteria checking
- Testing subset (~10%, e.g., 600) used for final evaluation
- subsample_splits.csv with columns point_index and subset (train/validation/test)

## How to apply

Load the binned time-domain FID containing 16,384 complex points from the input dataset file (e.g., NumPy complex arrays in a reimplementation, or MATLAB native complex arrays in the original SAND codebase). Optionally truncate the trailing-noise region, retaining the early-time portion where the exponentially decaying signal dominates (the SAND workflow illustrates truncation to the first 6,000 complex points). Randomly permute the selected point indices with a fixed random seed and split them 70/20/10 into training (parameter optimization), validation (stop-criteria checking), and testing (final evaluation) subsets — e.g., 4,200/1,200/600 points from 6,000. Write the per-point partition assignment (point_index, subset) to a CSV such as subsample_splits.csv. Note the partitioning is intended to infer a reasonable number of peaks rather than serve as strict cross-validation.

## Related tools

- **SAND** (Decomposition software whose workflow consumes the subsampled, partitioned time-domain points during hybrid MCMC optimization) — https://github.com/edisonomics/SAND
- **MATLAB** (Original implementation language in which the SAND resampling/partitioning module operates on complex FID arrays)
- **NMRbox** (Cloud environment where SAND (and thus the subsampling workflow) is hosted and run via a single terminal command)

## Examples

```
python resample_fid.py --fid binned_fid.npy --truncate 6000 --splits 70/20/10 --seed 42 --out subsample_splits.csv
```

## Evaluation signals

- The three partitions contain exactly the expected counts (e.g., 4,200 train / 1,200 validation / 600 test from 6,000 subsampled points)
- Partitions are mutually exclusive and jointly cover all subsampled points
- A rerun with the same fixed random seed reproduces identical partition assignments in subsample_splits.csv
- subsample_splits.csv has exactly the columns point_index and subset with values limited to train/validation/test
- Truncation retains only the early-time region where the exponentially decaying signal dominates (e.g., first 6,000 of 16,384 points)

## Limitations

- The partitioning is intended to infer a reasonable number of peaks, not to serve as strict cross-validation
- SAND's current version is tuned for high-resolution metabolomics spectra; other data types are not yet accommodated
- The MATLAB implementation is slow and not yet parallelizable; a faster C/C++ version is planned
- Trailing-noise FID truncation is optional, so results depend on whether/how many trailing points are discarded

## Evidence

- [methods] The resampling module optionally truncates trailing-noise FID points, then randomly assigns the selected points into a 70% training set for parameter optimization, a 20% validation set for stop-criteria checking, and a 10% testing set for final evaluation: [evidence span withheld — anchor status: not_found]
