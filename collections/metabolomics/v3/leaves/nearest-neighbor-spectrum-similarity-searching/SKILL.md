---
name: nearest-neighbor-spectrum-similarity-searching
description: Use when clustering very large tandem MS/MS datasets (e.g. 25 million
  spectra from a human draft proteome) where exhaustive pairwise spectrum comparison
  is quadratically expensive and an approximate sparse distance matrix suffices.
when_to_use_negative:
- When the dataset is small enough that exhaustive pairwise comparison is tractable
  and exact similarities are required, since approximate nearest neighbor searching
  can slightly miss neighbors
- When data are metabolomics or top-down spectra, because default preprocessing (min_peaks
  5, min_mz 101, max_mz 500, mass range 250 Da) is intended for bottom-up proteomics
  and would discard valid spectra
- When a similarity measure other than cosine distance is needed (e.g. probabilistic
  scores or fragment rarity metrics), as the hashing and inverted index conserves
  cosine similarity specifically
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: falcon
  role: Implements feature hashing, nearest neighbor indexing, sparse pairwise distance
    matrix computation, and DBSCAN clustering of MS/MS spectra
  repo: https://github.com/bittremieux/falcon
- name: Python 3.8
  role: Implementation language for falcon; minimum required version (Python 3.8+)
- name: ThermoRawFileParser 1.2.3
  role: Converts raw Thermo files to MGF peak files used as falcon input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/rcm.9153
    title: Large‐scale tandem mass spectrum clustering using fast nearest neighbor
      searching
  source_content_hash: sha256:7bb6d24b23bb8072c95334d704f94464021d25d57c1b866c771b6a8d278d5a23
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/bittremieux/falcon
  tools:
  - Python 3.8
  - ThermoRawFileParser 1.2.3
  - falcon
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# nearest-neighbor-spectrum-similarity-searching

## Summary

Efficiently computes spectrum similarities for millions of MS/MS spectra by converting high-resolution spectra to low-dimensional vectors via feature hashing, building nearest neighbor indexes (Voronoi diagram inverted indexes), and computing a sparse pairwise cosine distance matrix instead of exhaustive pairwise comparisons, as implemented in falcon.

## When to use

Use when clustering very large tandem MS/MS datasets (e.g. 25 million spectra from a human draft proteome) where exhaustive pairwise spectrum comparison is quadratically expensive and an approximate sparse distance matrix suffices.

## When NOT to use

- When the dataset is small enough that exhaustive pairwise comparison is tractable and exact similarities are required, since approximate nearest neighbor searching can slightly miss neighbors
- When data are metabolomics or top-down spectra, because default preprocessing (min_peaks 5, min_mz 101, max_mz 500, mass range 250 Da) is intended for bottom-up proteomics and would discard valid spectra
- When a similarity measure other than cosine distance is needed (e.g. probabilistic scores or fragment rarity metrics), as the hashing and inverted index conserves cosine similarity specifically

## Inputs

- MS/MS peak files in mzML, mzXML, or MGF format (e.g. MGF files converted with ThermoRawFileParser 1.2.3)
- falcon configuration: precursor_tol (e.g. 20 ppm), fragment_tol (e.g. 0.05 Da), eps cosine distance threshold, nearest neighbor indexing settings (n_probe, n_neighbors, n_neighbors_ann, low_dim)

## Outputs

- Sparse pairwise cosine distance matrix over nearest neighbors
- Nearest neighbor inverted indexes (Voronoi diagram via k-means) per precursor m/z bucket
- CSV cluster assignments mapping each MS/MS spectrum to its cluster label
- Optional MGF file of cluster representative (consensus) spectra

## How to apply

First, preprocess and convert spectra to sparse vectors using small mass bins, then hash these high-dimensional vectors to lower dimensions with MurmurHash3 feature hashing, which conserves cosine similarity. Partition spectra into 1 m/z buckets based on precursor mass (per precursor charge), and within each bucket partition vectors into subspaces to build a Voronoi diagram encoded as an inverted index via k-means clustering. Compute a sparse pairwise distance matrix by retrieving only cosine distances to a limited number of nearest neighbors (tuned via n_probe, n_neighbors, n_neighbors_ann, low_dim), trading speed against accuracy. Feed the sparse distance matrix into DBSCAN density-based clustering with a cosine distance threshold eps (e.g. 0.05–0.15 for pure clusters), using falcon settings such as precursor_tol 20 ppm and fragment_tol 0.05 Da. This avoids exhaustively comparing all spectra within relevant precursor buckets, which is the key to falcon's near-linear runtime scaling.

## Related tools

- **falcon** (Implements feature hashing, nearest neighbor indexing, sparse pairwise distance matrix computation, and DBSCAN clustering of MS/MS spectra) — https://github.com/bittremieux/falcon
- **Python 3.8** (Implementation language for falcon; minimum required version (Python 3.8+))
- **ThermoRawFileParser 1.2.3** (Converts raw Thermo files to MGF peak files used as falcon input)

## Examples

```
falcon peak/*.mzml falcon --export_representatives --precursor_tol 20 ppm --fragment_tol 0.05 --eps 0.10
```

## Evaluation signals

- At a cosine distance threshold of 0.05, the clustering should yield approximately 1% incorrectly clustered spectra
- Runtime should scale near-linearly with dataset size (10 to 2212 peak files, 25 million spectra) rather than quadratically with the number of spectra
- Clusters should be highly pure (spectra corresponding to only a single peptide) when eps is within the recommended 0.05–0.15 range
- Changing n_probe or n_neighbors downward should measurably reduce runtime but slightly degrade clustering accuracy, confirming the index is being used
- Output CSV must contain one cluster label per input MS/MS spectrum, and exported MGF representatives must match the number of clusters

## Limitations

- Nearest neighbor searching is approximate: exploring fewer inverted index lists (lower n_probe) or querying fewer neighbors runs faster but gives slightly less accurate clustering
- The scaling method (e.g. square root intensity transformation) influences the cosine distance threshold eps that must be chosen
- Default preprocessing (min 5 peaks, 250 m/z range, m/z window 101–500) is tuned for bottom-up proteomics and needs adjustment for metabolomics or top-down data
- The full extent of the effect of chimeric spectra on spectrum clustering and identification remains an open research question

## Evidence

- [methods] a feature hashing scheme is used to convert high-resolution MS/MS spectra to low-dimensional vectors: [evidence span withheld — anchor status: not_found]
