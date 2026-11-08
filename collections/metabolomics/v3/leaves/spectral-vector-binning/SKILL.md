---
name: spectral-vector-binning
description: Use when spectra must be transformed into fixed-dimensional numeric vectors
  (e.g. to build an ANN index or compute coarse cosine similarities) prior to spectral
  library candidate selection or similarity search.
when_to_use_negative:
- Input spectra are already processed and you only need to verify preprocessing, not
  re-vectorize
- You need fine-grained fragment-level matching rather than coarse candidate retrieval
  — the 1 Da binned vectors are too coarse for final SSM scoring, which requires the
  shifted dot product on un-binned peaks
- The library is small enough that exact brute-force comparison of all spectra is
  tractable, removing the need for an ANN vector representation
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ANN-SoLo
  role: Implements the spectrum preprocessing and 1 Da binning/vectorization that
    feed its approximate nearest neighbor index of random projection trees
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Implementation language for the vectorization and indexing pipeline (with
    C++)
  repo: https://github.com/bittremieux/ANN-SoLo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:331cba0f3506c5926e4a24d5ebeedd892b4556c9fb63876ac7c6b628220ca3dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spectral-vector-binning

## Summary

Convert peak-list spectra (query and library alike) into sparse, unit-length vectors by aggressive noise filtering, keeping only the 50 most intense peaks, and binning peaks into 1 Da mass bins with intensities summed per bin. This fixed-width sparse representation is the input format that makes approximate nearest neighbor (ANN) indexing of spectral libraries possible.

## When to use

Use when spectra must be transformed into fixed-dimensional numeric vectors (e.g. to build an ANN index or compute coarse cosine similarities) prior to spectral library candidate selection or similarity search.

## When NOT to use

- Input spectra are already processed and you only need to verify preprocessing, not re-vectorize
- You need fine-grained fragment-level matching rather than coarse candidate retrieval — the 1 Da binned vectors are too coarse for final SSM scoring, which requires the shifted dot product on un-binned peaks
- The library is small enough that exact brute-force comparison of all spectra is tractable, removing the need for an ANN vector representation

## Inputs

- Query spectra in peak-list form (e.g. from mzML files, e.g. the iPRG2012 or HEK293 data sets)
- Spectral library spectra (e.g. NIST human HCD library v2016/09/12, MassIVE-KB v2017/11/27, or Selevsek TripleTOF yeast library)

## Outputs

- Sparse unit-length vectors with one dimension per 1 Da mass bin, one per spectrum
- Per-spectrum candidate lists with coarse vector similarities feeding ann_candidates.csv

## How to apply

Preprocess query and library spectra identically so their vector spaces are comparable: remove precursor peaks and noise peaks below 1% of the base-peak intensity, keep only the 50 most intense peaks, and discard any spectrum with fewer than 10 peaks remaining or a mass range below 250 Da. Rank-transform the retained peak intensities, then vectorize each spectrum into a sparse vector over 1 Da mass bins, summing intensities of all peaks falling in a bin, and normalize the vector to unit length so dot products equal cosine similarity. This identical treatment of library and query spectra is what allows an ensemble of random projection trees over the binned library vectors to retrieve candidates for query vectors with high recall; coarse vector similarities computed on these binned vectors are only used for candidate selection, not final scoring (final SSM scoring uses the shifted dot product).

## Related tools

- **ANN-SoLo** (Implements the spectrum preprocessing and 1 Da binning/vectorization that feed its approximate nearest neighbor index of random projection trees) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Implementation language for the vectorization and indexing pipeline (with C++)) — https://github.com/bittremieux/ANN-SoLo

## Examples

```
ann_solo search queries.mzml library.spectra --precursor_tolerance 3Da --fdr 0.01 --internals --cache --num_candidates 100
```

## Evaluation signals

- Every output spectrum vector is normalized to unit length (L2 norm = 1), so dot products are interpretable as cosine similarities
- No retained spectrum contains fewer than 10 peaks or a mass range below 250 Da after filtering
- No spectrum retains more than 50 peaks, and no peak below 1% of the base-peak intensity survives filtering
- Bin width is exactly 1 Da; per-bin values equal the summed (rank-transformed) intensities of peaks in that bin
- Library and query spectra pass through the identical preprocessing chain, verified by checking that query vectors over the same bins use the same normalization as library vectors

## Limitations

- 1 Da binning is intentionally coarse; it supports only approximate candidate retrieval and cannot resolve fragment-level differences used in final scoring
- ANN retrieval is approximate, so candidate sets are not guaranteed to contain the exact nearest neighbors; correctness relies on returning a sufficiently high number of candidates
- Spectra that fall below the minimum peak count (10) or mass range (250 Da) thresholds are discarded entirely and cannot be searched
- Non-metric similarity measures (e.g. the shifted dot product) cannot be indexed directly with this vector scheme; retrieving candidates by shifted dot product directly is untested future work

## Evidence

- [methods] noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] restrict to 50 most intense peaks, discard sparse spectra: "the spectrum is further restricted to its 50 most intense peaks"
- [methods] any spectrum that contains fewer than 10 peaks remaining or with a mass range less than 250 Da is discarded: "any spectrum that contains fewer than 10 peaks remaining or with a mass range less than 250 Da is discarded"
- [methods] identical preprocessing of query and library spectra: "Prior to spectral library searching both the query spectra and library spectra are similarly preprocessed"
- [readme] ANN index selects a limited set of most similar library spectra: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra to compare to an unknown query"
- [intro] final matching uses the shifted dot product, not the binned vectors: "the shifted dot product is used to accurately match modified query spectra to their unmodified library counterpart by taking into account peak shifts caused by a modification"
