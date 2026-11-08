---
name: precursor-mass-tolerance-filtering
description: Use when computing a sparse pairwise spectrum similarity matrix or nearest-neighbor
  candidate list and only pairs within a precursor mass tolerance (ppm or Dalton)
  are valid, as in MS/MS clustering.
when_to_use_negative:
- When comparing spectra with unknown or missing precursor mass/charge information
  — the bucketing and post-filtering cannot be applied reliably
- When analyzing metabolomics or top-down data where the default bottom-up proteomics
  tolerance assumptions and mass ranges (101–500 m/z peaks) do not apply without reconfiguration
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: Implements precursor mass bucketing (1 m/z per charge) and `precursor_tol`
    filtering of nearest-neighbor pairs before building the sparse distance matrix
  repo: https://github.com/bittremieux/falcon
- name: Faiss
  role: Approximate nearest-neighbor search whose retrieved candidates (up to 128
    neighbors, 32 cells probed) are subsequently filtered by the 20 ppm precursor
    mass tolerance
- name: SciPy
  role: Stores the tolerance-filtered distances as a sparse pairwise distance matrix
    (pairwise_distance_matrix.npz)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:78efabea0fae8934e27c348e06e0c0ba36a99db16aba14d53678ee4bf1014040
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - NumPy
  - Python 3.8
  - SciPy
  - Scikit-Learn
  - falcon
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# precursor-mass-tolerance-filtering

## Summary

Restrict spectrum-spectrum similarity comparisons to pairs whose precursor masses differ within a user-specified tolerance (e.g. 20 ppm in falcon), so the sparse pairwise cosine-distance matrix only contains biologically plausible matches. In falcon this is enforced both by bucketing spectra per precursor charge into 1 m/z precursor-mass buckets and by post-filtering retrieved nearest neighbors.

## When to use

Use when computing a sparse pairwise spectrum similarity matrix or nearest-neighbor candidate list and only pairs within a precursor mass tolerance (ppm or Dalton) are valid, as in MS/MS clustering.

## When NOT to use

- When comparing spectra with unknown or missing precursor mass/charge information — the bucketing and post-filtering cannot be applied reliably
- When analyzing metabolomics or top-down data where the default bottom-up proteomics tolerance assumptions and mass ranges (101–500 m/z peaks) do not apply without reconfiguration

## Inputs

- Preprocessed MS/MS spectra converted to feature-hashed low-dimensional vectors (MurmurHash3 mass-bin hashing, e.g. 800-dimensional)
- Precursor m/z and charge per spectrum (for 1 m/z bucket partitioning)
- Precursor mass tolerance value and unit (e.g. 20 ppm or Dalton)

## Outputs

- Sparse pairwise cosine-distance matrix restricted to precursor-tolerance-compatible pairs (SciPy sparse format, pairwise_distance_matrix.npz)
- At most 64 retained nearest neighbors per spectrum that satisfy the tolerance

## How to apply

First partition the spectra per precursor charge into 1 m/z buckets based on precursor mass, so exhaustive comparison is only attempted within narrow precursor-mass windows. After approximate nearest-neighbor retrieval (e.g. Faiss inverted index over the feature-hashed spectrum vectors, exploring up to 32 Voronoi cells and retrieving 128 neighbors), discard any candidate neighbor pair whose precursor mass difference exceeds the tolerance — 20 ppm in the falcon human draft proteome evaluation. Retain at most 64 nearest neighbors per spectrum with their cosine distances and assemble them into a sparse pairwise distance matrix (SciPy sparse format, saved as pairwise_distance_matrix.npz). In falcon this is controlled by the `precursor_tol` setting, given as a value and unit (ppm or Dalton), e.g. `--precursor_tol 20 ppm`. The rationale is that spectra of different precursor masses cannot correspond to the same peptide, so such pairs must never enter the DBSCAN clustering step.

## Related tools

- **falcon** (Implements precursor mass bucketing (1 m/z per charge) and `precursor_tol` filtering of nearest-neighbor pairs before building the sparse distance matrix) — https://github.com/bittremieux/falcon
- **Faiss** (Approximate nearest-neighbor search whose retrieved candidates (up to 128 neighbors, 32 cells probed) are subsequently filtered by the 20 ppm precursor mass tolerance)
- **SciPy** (Stores the tolerance-filtered distances as a sparse pairwise distance matrix (pairwise_distance_matrix.npz))

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Every retained spectrum pair in the sparse distance matrix has a precursor mass difference within the configured tolerance (e.g. ≤ 20 ppm)
- The distance matrix is sparse — a bounded number of stored distances per spectrum (at most 64 neighbors) rather than N×N dense
- No pair in the matrix spans different 1 m/z precursor buckets (per charge) unless their mass difference still satisfies the ppm tolerance
- Brute-force vs. ANN comparison: falcon performed on average only 47 spectrum–spectrum comparisons per spectrum vs. 1908 in brute force at a 20 ppm tolerance
- Downstream DBSCAN clustering with eps = 0.05–0.15 yields clusters consistent with pairs that passed the precursor filter

## Limitations

- Approximate nearest-neighbor searching can miss true neighbors, so tolerance filtering runs after ANN retrieval — exploring fewer cells (n_probe) or retrieving fewer neighbors reduces accuracy
- The precursor tolerance interacts with bucketing: spectra are partitioned per charge into 1 m/z buckets, so a tolerance larger than the bucket width needs care at bucket boundaries
- precursor_tol must be given with an explicit unit (ppm or Dalton); mixing units or choosing too large a tolerance inflates comparisons and can admit incorrect matches

## Evidence

- [results] falcon performed on average only 47 spectrum–spectrum comparisons per spectrum: [evidence span withheld — anchor status: not_found]
