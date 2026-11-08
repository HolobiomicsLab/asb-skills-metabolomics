---
name: sparse-cosine-distance-matrix-computation
description: Use when clustering millions of MS/MS spectra requires pairwise cosine
  distances but exhaustive all-pairs comparison is infeasible — e.g., 25M spectra
  at a 20 ppm precursor mass tolerance.
when_to_use_negative:
- Datasets small enough that exhaustive all-pairs comparison is tractable — falcon
  itself falls back to brute-force for buckets with fewer than 100 spectra.
- Spectra lacking precursor m/z and charge metadata, since bucketing and the 20 ppm
  neighbor filter depend on precursor mass.
- Analyses requiring a non-cosine similarity measure (e.g., MaRaCluster's fragment
  rarity score or spectra-cluster's probabilistic score) — feature hashing conserves
  cosine similarity only.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Implements the full pipeline — feature-hashed vectors, Voronoi inverted indexes
    via Faiss, and sparse cosine distance matrix assembly — with tuning via n_probe,
    n_neighbors/n_neighbors_ann, and low_dim
  repo: https://github.com/bittremieux/falcon
- name: Faiss
  role: Executes the k-means Voronoi-diagram inverted index queries, probing at most
    32 neighboring cells to retrieve the 128 nearest neighbors per spectrum
- name: Scikit-Learn
  role: k-means clustering determines the Voronoi cell representative vectors within
    each 1 m/z precursor-mass bucket
- name: SciPy
  role: Supplies the sparse matrix format used to assemble and save the <=64-neighbor
    cosine distance matrix (pairwise_distance_matrix.npz)
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:1f6a93ae0741c93366a9a3f11e9096097ad68a4d9764fac87475b4a5f82d0e7d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - NumPy
  - Pyteomics
  - Python 3.8
  - SciPy
  - Scikit-Learn
  - ThermoRawFileParser
  - falcon
  - fastcluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# sparse-cosine-distance-matrix-computation

## Summary

Computes a sparse pairwise cosine-distance matrix between millions of MS/MS spectra by approximate nearest-neighbor search over k-means Voronoi inverted indexes (Faiss), retaining cosine distances to at most 64 neighbors within a 20 ppm precursor tolerance so that DBSCAN clustering avoids exhaustive all-pairs comparison.

## When to use

Use when clustering millions of MS/MS spectra requires pairwise cosine distances but exhaustive all-pairs comparison is infeasible — e.g., 25M spectra at a 20 ppm precursor mass tolerance.

## When NOT to use

- Datasets small enough that exhaustive all-pairs comparison is tractable — falcon itself falls back to brute-force for buckets with fewer than 100 spectra.
- Spectra lacking precursor m/z and charge metadata, since bucketing and the 20 ppm neighbor filter depend on precursor mass.
- Analyses requiring a non-cosine similarity measure (e.g., MaRaCluster's fragment rarity score or spectra-cluster's probabilistic score) — feature hashing conserves cosine similarity only.

## Inputs

- Preprocessed MS/MS spectra (precursor and low-intensity noise peaks removed, >=5 peaks spanning >=250 m/z, square-root-transformed intensities) binned at 0.05 m/z
- Feature-hashed 800-dimensional spectrum vectors (MurmurHash3 over 0.05 m/z mass bins)
- Precursor m/z and charge per spectrum (for 1 m/z bucketing and 20 ppm neighbor filtering)

## Outputs

- Sparse pairwise cosine distance matrix (<=64 retained distances per spectrum) in SciPy sparse format, saved as pairwise_distance_matrix.npz
- Retained spectrum–spectrum pairs, each satisfying the 20 ppm precursor mass tolerance

## How to apply

Convert each preprocessed spectrum (0.05 m/z bins, square-root-transformed intensities, precursor peaks removed) into an 800-dimensional vector via MurmurHash3 feature hashing, which conserves cosine similarity so hashed distances approximate true full-resolution cosine distances. Per precursor charge, partition the vectors into 1 m/z precursor-mass buckets and build a Voronoi-diagram inverted index with k-means (dynamically sized up to 65536 cells) in Faiss, falling back to brute-force search for buckets with fewer than 100 spectra. Query each vector by exploring at most 32 neighboring Voronoi cells (n_probe) and retrieving 128 nearest neighbors (n_neighbors_ann). Omit candidates whose precursor mass difference exceeds 20 ppm and retain the cosine distances to at most 64 nearest neighbors (n_neighbors). Assemble the retained distances into a SciPy sparse matrix saved as pairwise_distance_matrix.npz for downstream DBSCAN; on the 25M-spectrum human draft proteome this averaged 47 comparisons per spectrum versus 1908 for brute force at a 20 ppm tolerance.

## Related tools

- **falcon** (Implements the full pipeline — feature-hashed vectors, Voronoi inverted indexes via Faiss, and sparse cosine distance matrix assembly — with tuning via n_probe, n_neighbors/n_neighbors_ann, and low_dim) — https://github.com/bittremieux/falcon
- **Faiss** (Executes the k-means Voronoi-diagram inverted index queries, probing at most 32 neighboring cells to retrieve the 128 nearest neighbors per spectrum)
- **Scikit-Learn** (k-means clustering determines the Voronoi cell representative vectors within each 1 m/z precursor-mass bucket)
- **SciPy** (Supplies the sparse matrix format used to assemble and save the <=64-neighbor cosine distance matrix (pairwise_distance_matrix.npz))

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Every retained spectrum pair satisfies the 20 ppm precursor mass tolerance; pairs beyond tolerance are absent from the matrix.
- Stored distances are bounded per spectrum (<=64); falcon averaged 47 comparisons per spectrum versus 1908 for brute force at a 20 ppm tolerance.
- Distances are cosine distances on 800-dimensional hashed vectors, approximating full-resolution cosine similarity (feature hashing conserves cosine similarity).
- Buckets with fewer than 100 spectra use brute-force search; larger buckets use k-means Voronoi indexes with up to 65536 cells and <=32 probed neighbors.
- Feeding the matrix to DBSCAN at cosine distance 0.05 reproduces approximately 1% incorrectly clustered spectra on the 25M-spectrum human draft proteome benchmark.

## Limitations

- Speed/accuracy trade-off in ANN settings: exploring fewer index lists (n_probe) or querying fewer neighbors (n_neighbors_ann) runs faster but risks missing true nearest neighbors.
- The low-dimensional hashed vectors (low_dim, 800 in the paper) only approximate the true cosine distance; larger vectors improve accuracy at the cost of search time and memory.
- Only neighbors within the precursor mass tolerance (20 ppm) are retained, and the intensity scaling choice (square root, logarithm, rank, none) changes distances and the eps threshold needed for pure clusters.

## Evidence
