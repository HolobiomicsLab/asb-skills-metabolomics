---
name: approximate-nearest-neighbor-indexing
description: Use when clustering or comparing millions of MS/MS spectra where exhaustive
  all-vs-all pairwise comparison is computationally infeasible and a sparse distance
  matrix to each spectrum's nearest neighbors suffices.
when_to_use_negative:
- Small datasets where exhaustive pairwise comparison is cheap enough and approximate
  search would only add recall risk
- Metabolomics or top-down spectra with the default bottom-up proteomics preprocessing
  settings (min 5 peaks, 101-500 m/z range) still in place without adjustment
- Analyses requiring exact cosine similarities between all spectrum pairs rather than
  an approximate sparse neighborhood
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: 'Implements the full approximate nearest neighbor indexing pipeline: feature
    hashing, bucketing, inverted index construction, and sparse distance matrix computation'
  repo: https://github.com/bittremieux/falcon
- name: Faiss
  role: Backend used to build and query the k-means/Voronoi-diagram inverted indexes
    per precursor-mass bucket
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:98336fcb72dc675adb4501655925557958ad59ed54f99cb2550dd0a2d30257c0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Faiss
  - Pyteomics
  - Python 3.8
  - Scikit-Learn
  - ThermoRawFileParser
  - falcon
  - fastcluster
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# approximate-nearest-neighbor-indexing

## Summary

A technique from falcon that converts high-resolution MS/MS spectra to low-dimensional vectors via feature hashing, then builds Voronoi-diagram inverted indexes per precursor-mass bucket to find approximate nearest neighbors without exhaustive pairwise comparison.

## When to use

Use when clustering or comparing millions of MS/MS spectra where exhaustive all-vs-all pairwise comparison is computationally infeasible and a sparse distance matrix to each spectrum's nearest neighbors suffices.

## When NOT to use

- Small datasets where exhaustive pairwise comparison is cheap enough and approximate search would only add recall risk
- Metabolomics or top-down spectra with the default bottom-up proteomics preprocessing settings (min 5 peaks, 101-500 m/z range) still in place without adjustment
- Analyses requiring exact cosine similarities between all spectrum pairs rather than an approximate sparse neighborhood

## Inputs

- Preprocessed MS/MS peak files (mzML, mzXML, or MGF)
- Per-spectrum precursor m/z and charge for bucket partitioning

## Outputs

- Sparse pairwise cosine distance matrix (at most 64 nearest neighbors per spectrum)
- Nearest neighbor indexes (Voronoi-diagram inverted indexes per precursor-mass bucket)

## How to apply

Preprocess each spectrum (remove precursor-ion peaks, drop noise peaks below 10% of base peak intensity, restrict to the 50 most intense peaks, square-root transform intensities), then bin peaks at 0.05 m/z and apply feature hashing with 32-bit MurmurHash3 to vectors of length 800. Partition vectors per precursor charge into 1 m/z precursor-mass buckets and build a Voronoi-diagram inverted index per bucket using k-means (up to 65536 cells) with Faiss. During querying, explore at most 32 neighboring cells and retrieve the 128 nearest neighbors, omitting neighbors whose precursor mass differs by more than 20 ppm. Store cosine distances to at most 64 nearest neighbors in a sparse pairwise distance matrix, which is then clustered with DBSCAN (eps 0.05 for ~1% incorrectly clustered spectra). Tuning n_probe, n_neighbors, and low_dim trades clustering speed against accuracy.

## Related tools

- **falcon** (Implements the full approximate nearest neighbor indexing pipeline: feature hashing, bucketing, inverted index construction, and sparse distance matrix computation) — https://github.com/bittremieux/falcon
- **Faiss** (Backend used to build and query the k-means/Voronoi-diagram inverted indexes per precursor-mass bucket)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- At a cosine distance threshold of 0.05, the fraction of incorrectly clustered spectra is approximately 1%, as reported in the paper
- Exploring more neighboring cells during querying should decrease the chance of missing a nearest neighbor, monotonically improving recall at the cost of longer search time
- Feature hashing must conserve cosine similarity between hashed vectors so they approximate similarity of the original spectra
- Each spectrum should retain distances to its nearest neighbors only after precursor mass filtering at 20 ppm; neighbors exceeding the tolerance are omitted
- On the 25-million-spectrum human draft proteome dataset, full indexing plus clustering completed in roughly five hours

## Limitations

- Querying fewer lists or neighbors runs faster but gives slightly less accurate clustering results
- Larger low_dim vectors approximate the true cosine distance more accurately but increase nearest neighbor search time and memory requirements
- The full effect of chimeric spectra on spectrum clustering and identification remains an open research question
- How MS/MS spectrum preprocessing influences various similarity measures was not evaluated

## Evidence

- [readme] First, high-resolution spectra are binned and converted to low-dimensional vectors using feature hashing.: "First, high-resolution spectra are binned and converted to low-dimensional vectors using feature hashing."
- [methods] The Voronoi diagram is encoded by an inverted index: "The Voronoi diagram is encoded by an inverted index, with each Voronoi cell defined by a single vector, determined using k-means clustering"
- [methods] This efficient similarity searching is used to construct a sparse pairwise distance matrix that contains the cosine distances between each spectrum and a limited number of its nearest neighbors: "This efficient similarity searching is used to construct a sparse pairwise distance matrix that contains the cosine distances between each spectrum and a limited number of its nearest neighbors"
- [readme] n_probe: The maximum number of lists in the inverted index to inspect during querying.: "The accuracy and speed of similarity searching is governed by the number of neighboring cells to explore during searching: exploring more cells during searching decreases the chance of missing a"
- [methods] clustering result with approximately 1% incorrectly clustered spectra was obtained with a cosine distance threshold of 0.05: "The clustering result with approximately 1% incorrectly clustered spectra was obtained with a cosine distance threshold of 0.05."
