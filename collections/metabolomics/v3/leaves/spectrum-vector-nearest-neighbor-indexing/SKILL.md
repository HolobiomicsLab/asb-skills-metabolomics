---
name: spectrum-vector-nearest-neighbor-indexing
description: Use when pairwise spectral similarity must be computed for large MS/MS
  datasets (millions of spectra) where exhaustive all-vs-all comparison within a precursor
  mass tolerance is computationally infeasible.
when_to_use_negative:
- When the dataset is small enough that exact brute-force all-vs-all cosine comparison
  is affordable and approximate neighbor search would only add accuracy risk
- When spectra lack reliable precursor m/z/charge information, since bucketing by
  precursor mass (1 m/z buckets per charge) is required for the index
- When a similarity measure other than cosine distance is needed — falcon's hashing
  and inverted index are built around cosine similarity, and the paper notes alternative
  scores (spectra-cluster's probabilistic score, MaRaCluster's fragment rarity score)
  were not established here
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: falcon
  role: 'Implements the full pipeline: feature hashing, bucketing, inverted-index
    nearest neighbor searching, and sparse distance matrix construction'
  repo: https://github.com/bittremieux/falcon
- name: Faiss
  role: Queries each hashed spectrum vector against the Voronoi-diagram inverted index,
    exploring neighboring cells to retrieve nearest neighbors
- name: Scikit-Learn
  role: k-means clustering to determine the Voronoi cell representative vectors for
    the inverted index
- name: NumPy
  role: Numerical handling of hashed spectrum vectors and precursor mass bucketing
- name: SciPy
  role: Stores the retained cosine distances as a sparse pairwise distance matrix
    (pairwise_distance_matrix.npz)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:3b5bf84f4e56e77f9f4d61a1a3820bfb3d258a66a855ee31e5549a8e9e7c3861
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spectrum-vector-nearest-neighbor-indexing

## Summary

Build approximate nearest-neighbor indexes over feature-hashed MS/MS spectrum vectors to compute a sparse pairwise cosine-distance matrix, avoiding exhaustive all-vs-all spectrum comparison within a precursor mass tolerance. This is the core speed mechanism of the falcon spectrum clustering tool.

## When to use

Use when pairwise spectral similarity must be computed for large MS/MS datasets (millions of spectra) where exhaustive all-vs-all comparison within a precursor mass tolerance is computationally infeasible.

## When NOT to use

- When the dataset is small enough that exact brute-force all-vs-all cosine comparison is affordable and approximate neighbor search would only add accuracy risk
- When spectra lack reliable precursor m/z/charge information, since bucketing by precursor mass (1 m/z buckets per charge) is required for the index
- When a similarity measure other than cosine distance is needed — falcon's hashing and inverted index are built around cosine similarity, and the paper notes alternative scores (spectra-cluster's probabilistic score, MaRaCluster's fragment rarity score) were not established here

## Inputs

- Preprocessed MS/MS spectra (peak lists from mzML, mzXML, or MGF peak files) with precursor m/z and charge
- Settings: mass bin width (0.05 m/z), low-dimensional vector length (low_dim, e.g. 800), precursor mass tolerance (e.g. 20 ppm), n_probe (32), n_neighbors (64), n_neighbors_ann (128)

## Outputs

- Sparse pairwise cosine-distance matrix in SciPy sparse format (pairwise_distance_matrix.npz) containing distances to at most 64 nearest neighbors per spectrum, restricted to pairs within the precursor mass tolerance

## How to apply

First convert preprocessed spectra to sparse vectors using 0.05 m/z mass bins, then compress them to low-dimensional vectors (e.g. 800 dimensions) via MurmurHash3 feature hashing, which conserves cosine similarity. Partition the hashed vectors per precursor charge into 1 m/z buckets based on precursor mass; within each bucket build a Voronoi-diagram inverted index where each cell is defined by a vector determined using k-means clustering (sklearn), falling back to brute-force search for buckets with fewer than 100 spectra. Query each vector with Faiss, exploring at most 32 neighboring Voronoi cells (n_probe) and retrieving 128 nearest neighbors (n_neighbors_ann), then filter neighbors by a precursor mass tolerance (e.g. 20 ppm) and retain cosine distances to at most 64 nearest neighbors (n_neighbors). Assemble the retained distances into a SciPy sparse pairwise distance matrix. The rationale is that restricting comparisons to buckets plus approximate ANN search reduces comparisons to ~47 per spectrum versus 1908 brute-force at a 20 ppm precursor tolerance, while eps-level cosine distances (e.g. 0.05 threshold) remain accurate enough for downstream DBSCAN clustering.

## Related tools

- **falcon** (Implements the full pipeline: feature hashing, bucketing, inverted-index nearest neighbor searching, and sparse distance matrix construction) — https://github.com/bittremieux/falcon
- **Faiss** (Queries each hashed spectrum vector against the Voronoi-diagram inverted index, exploring neighboring cells to retrieve nearest neighbors)
- **Scikit-Learn** (k-means clustering to determine the Voronoi cell representative vectors for the inverted index)
- **NumPy** (Numerical handling of hashed spectrum vectors and precursor mass bucketing)
- **SciPy** (Stores the retained cosine distances as a sparse pairwise distance matrix (pairwise_distance_matrix.npz))

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- The saved matrix is sparse: each spectrum has a bounded number of stored distances (at most 64 retained neighbors per spectrum)
- Every retained spectrum pair satisfies the precursor mass tolerance filter (e.g. mass difference ≤ 20 ppm)
- Average number of spectrum–spectrum comparisons per spectrum is far below brute force (falcon: ~47 per spectrum vs 1908 brute-force at 20 ppm precursor tolerance)
- Hashed-vector cosine similarity approximates original-spectrum cosine similarity, since feature hashing conserves cosine similarity
- Downstream DBSCAN clustering on the matrix yields highly pure clusters at the chosen eps (e.g. ~1% incorrectly clustered spectra at cosine distance threshold 0.05)

## Limitations

- Accuracy versus speed is governed by n_probe (neighboring Voronoi cells explored) and the number of neighbors queried; inspecting fewer lists or querying fewer neighbors runs faster but gives slightly less accurate clustering
- low_dim controls the fidelity of the hashed vectors: larger vectors approximate the true cosine distance more accurately at the expense of longer search time and memory
- Buckets with fewer than 100 spectra use brute-force search instead of the inverted index
- The effect of spectrum preprocessing on similarity measures was not evaluated, and alternative similarity measures (probabilistic, fragment rarity, machine-learning-derived) might capture spectrum similarity more sensitively

## Evidence
