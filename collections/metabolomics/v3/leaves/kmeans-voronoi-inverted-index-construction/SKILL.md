---
name: kmeans-voronoi-inverted-index-construction
description: Use when a sparse pairwise cosine-distance matrix over many MS/MS spectra
  within a precursor mass tolerance must be computed efficiently, and exhaustive pairwise
  comparison (e.g. 1908 comparisons/spectrum at 20 ppm) is infeasible.
when_to_use_negative:
- Buckets contain very few spectra (fewer than ~100), where brute-force search is
  the appropriate choice and the inverted index adds no benefit
- The analysis goal is exact all-pairs distances, since the Voronoi index gives an
  approximate search that can miss nearest neighbors when too few cells are probed
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: falcon
  role: Spectrum clustering tool whose nearest-neighbor indexing stage constructs
    the Voronoi-diagram inverted indexes over hashed spectrum vectors
  repo: https://github.com/bittremieux/falcon
- name: Faiss
  role: Library used to query each spectrum vector against the inverted index, exploring
    neighboring Voronoi cells to retrieve nearest neighbors
- name: Scikit-Learn
  role: Provides the k-means clustering that determines the Voronoi cell representative
    vectors
- name: NumPy
  role: Numerical backend for hashed spectrum vectors and distance computations
- name: SciPy
  role: Stores the resulting sparse pairwise cosine-distance matrix in SciPy sparse
    format
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:35cf74e1bc9291e9e073595614070f0df7d89d6fb58f98e28c353e0fedfea74e
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kmeans-voronoi-inverted-index-construction

## Summary

Build a Voronoi-diagram inverted index over hashed MS/MS spectrum vectors, where each Voronoi cell is defined by a k-means-derived representative vector, enabling fast approximate nearest-neighbor similarity search without exhaustive all-pairs spectrum comparison.

## When to use

Use when a sparse pairwise cosine-distance matrix over many MS/MS spectra within a precursor mass tolerance must be computed efficiently, and exhaustive pairwise comparison (e.g. 1908 comparisons/spectrum at 20 ppm) is infeasible.

## When NOT to use

- Buckets contain very few spectra (fewer than ~100), where brute-force search is the appropriate choice and the inverted index adds no benefit
- The analysis goal is exact all-pairs distances, since the Voronoi index gives an approximate search that can miss nearest neighbors when too few cells are probed

## Inputs

- Low-dimensional hashed spectrum vectors (e.g. 800-dim MurmurHash3 feature-hashed vectors from 0.05 m/z bins)
- Precursor m/z and charge annotations for each MS/MS spectrum
- Index configuration parameters: number of Voronoi cells (from bucket size N), n_probe (e.g. 32), n_neighbors_ann (e.g. 128), n_neighbors (e.g. 64)

## Outputs

- Per-bucket Voronoi-diagram inverted indexes (Faiss indexes mapping cell representative vectors to member spectrum vectors)
- Nearest-neighbor candidate lists per spectrum with cosine distances
- A sparse pairwise cosine-distance matrix (e.g. pairwise_distance_matrix.npz, SciPy sparse format) restricted to precursor-mass-tolerant pairs

## How to apply

After converting preprocessed spectra to low-dimensional vectors via MurmurHash3 feature hashing (e.g. 0.05 m/z bins compressed to 800 dimensions), partition the vectors per precursor charge into 1 m/z buckets based on precursor mass. Within each bucket, run k-means clustering so each cluster centroid defines a Voronoi cell encoded as an inverted index, setting the number of cells dynamically from bucket size N (up to ~65536 cells); fall back to brute-force search for buckets with fewer than ~100 spectra. Query each vector with Faiss, exploring at most 32 neighboring Voronoi cells (n_probe) and retrieving 128 nearest neighbors (n_neighbors_ann) before keeping the cosine distances to at most 64 neighbors per spectrum. The rationale is that exploring more neighboring cells decreases the chance of missing a true nearest neighbor in the high-dimensional space at the expense of search time, and buckets by precursor m/z restrict candidate pairs so that only spectra within the precursor mass tolerance (e.g. 20 ppm) are compared.

## Related tools

- **falcon** (Spectrum clustering tool whose nearest-neighbor indexing stage constructs the Voronoi-diagram inverted indexes over hashed spectrum vectors) — https://github.com/bittremieux/falcon
- **Faiss** (Library used to query each spectrum vector against the inverted index, exploring neighboring Voronoi cells to retrieve nearest neighbors)
- **Scikit-Learn** (Provides the k-means clustering that determines the Voronoi cell representative vectors)
- **NumPy** (Numerical backend for hashed spectrum vectors and distance computations)
- **SciPy** (Stores the resulting sparse pairwise cosine-distance matrix in SciPy sparse format)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- Compare the number of spectrum–spectrum comparisons per spectrum against brute force: falcon averaged ~47 comparisons/spectrum vs ~1908 brute-force at 20 ppm precursor tolerance
- Verify the output distance matrix is sparse, with a bounded number of stored distances per spectrum (at most 64 neighbors retained)
- Check that every retained spectrum pair satisfies the precursor mass tolerance (e.g. 20 ppm) — pairs exceeding it must be omitted
- Increase n_probe and confirm clustering results converge (fewer missed nearest neighbors as more Voronoi cells are explored)
- Confirm brute-force fallback is triggered for buckets with fewer than 100 spectra

## Limitations

- Approximate search can miss nearest neighbors: exploring fewer Voronoi cells (lower n_probe) runs faster but yields slightly less accurate clustering
- Querying fewer neighbors (lower n_neighbors/n_neighbors_ann) runs faster but reduces clustering accuracy; n_neighbors_ann should be >= n_neighbors
- The feature-hashed low-dimensional vectors only approximate the true cosine distance; larger vectors (higher low_dim) improve accuracy at the expense of search time and memory
- Index settings trade clustering time versus accuracy and changing them is only recommended for advanced users

## Evidence

- [results] Using its advanced nearest neighbor searching approach, falcon performed on average only 47 spectrum–spectrum comparisons per spectrum, compared with 1908 comparisons per spectrum in a brute-force computation at a 20ppm precursor mass tolerance.: [evidence span withheld — anchor status: not_found]
