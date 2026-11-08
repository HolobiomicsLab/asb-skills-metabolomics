---
name: random-projection-tree-index-construction
description: Use when performing open modification spectral library searching against
  large libraries (e.g. NIST human HCD or MassIVE-KB), where exhaustive shifted-dot-product
  comparison of every query against all library spectra is too slow and a limited
  candidate set suffices.
when_to_use_negative:
- When an exact search is required — the index is deliberately approximate; the paper
  tolerates this only because a large number of candidates is returned, so do not
  use it when guaranteed retrieval of all nearest neighbors is mandatory
- For small libraries where exhaustive brute-force comparison is already fast enough
  — the ANN index overhead is not justified
- When searching for unmodified peptides in the first cascade level — the paper uses
  a cascade search where unmodified peptides are identified before the open search
  that needs ANN candidate selection
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: ANN-SoLo
  role: Implements the random projection tree ensemble index construction and query
    traversal used to select candidate library spectra during the open search
  repo: https://github.com/bittremieux/ANN-SoLo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:8d77ee4067df0fc30c6e2b48f3335c7aec6c9fe40e1ad0e25817360e9c5c10aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# random-projection-tree-index-construction

## Summary

Constructs an ensemble of random projection trees over sparse 1-Da-binned spectral library vectors (ANN-SoLo) so that a limited, high-recall candidate set of library spectra can be retrieved per query during open modification searching instead of brute-force comparison.

## When to use

Use when performing open modification spectral library searching against large libraries (e.g. NIST human HCD or MassIVE-KB), where exhaustive shifted-dot-product comparison of every query against all library spectra is too slow and a limited candidate set suffices.

## When NOT to use

- When an exact search is required — the index is deliberately approximate; the paper tolerates this only because a large number of candidates is returned, so do not use it when guaranteed retrieval of all nearest neighbors is mandatory
- For small libraries where exhaustive brute-force comparison is already fast enough — the ANN index overhead is not justified
- When searching for unmodified peptides in the first cascade level — the paper uses a cascade search where unmodified peptides are identified before the open search that needs ANN candidate selection

## Inputs

- Preprocessed spectral library spectra (e.g. NIST human HCD library v2016/09/12 or MassIVE-KB v2017/11/27) as sparse unit-length vectors over 1 Da mass bins
- Preprocessed query spectra (centroided MS2, e.g. from mzML files of the iPRG2012 or HEK293 data sets), vectorized identically to the library
- ANN index hyperparameters: number of trees in the ensemble (100–1000) and number of tree nodes to inspect per query (~20000–400000)

## Outputs

- ann_candidates.csv recording, per query spectrum, the retrieved candidate library spectrum identifiers and their coarse vector similarities
- Candidate set per query forwarded to the precursor mass filter and shifted-dot-product ranking step

## How to apply

First preprocess library and query spectra identically: remove precursor and noise peaks below 1% of the base-peak intensity, restrict to the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range below 250 Da, and rank-transform peak intensities. Vectorize each library spectrum into a sparse vector using 1 Da mass bins with intensities summed per bin, then normalize to unit length so dot products equal cosine similarity. Build an ensemble of random projection trees that recursively partition the library vectors using random split hyperplanes constructed from two randomly sampled points; the paper's iPRG2012 settings use roughly 100–1000 trees. This index is built once in an initialization phase; at query time, traverse the trees to inspect approximately 20000–400000 nodes per query and combine the reached subspaces across trees into the candidate set. The rationale is that approximate retrieval is acceptable because a sufficiently large number of candidates is returned, which is then followed by precursor mass filtering and shifted-dot-product candidate ranking.

## Related tools

- **ANN-SoLo** (Implements the random projection tree ensemble index construction and query traversal used to select candidate library spectra during the open search) — https://github.com/bittremieux/ANN-SoLo

## Evaluation signals

- ann_candidates.csv exists and is well-formed, containing candidate library spectrum identifiers and coarse similarity scores for each query spectrum
- Number of trees in the ensemble falls in the paper's 100–1000 range (iPRG2012 settings)
- Nodes inspected per query fall in the ~20000–400000 range reported for the paper's settings
- Expert review confirms high recall of candidates relative to an exact brute-force nearest-neighbor check on a sample of queries
- Spectral vectors are normalized to unit length so coarse similarities are comparable as cosine similarities on 1 Da-binned vectors

## Limitations

- Retrieval is approximate; the paper tolerates this only because a sufficiently high number of candidates are returned, so rare true matches may be missed
- The paper did not investigate whether alternative ANN indexing techniques (HNSW, LSH, product quantization) are suitable for spectral library searching; non-metric space indexing for direct shifted-dot-product retrieval remains untested
- GPU acceleration for candidate selection and SSM evaluation was not yet implemented in this version
- Python 3.6 to 3.9 is required by ANN-SoLo per the README (Python 3.10 and newer not supported)

## Evidence

- [intro] ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum.: "During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum."
- [methods] The ANN index is governed by two major hyperparameters: the number of trees in the index ensemble and the number of tree nodes to inspect.: [evidence span withheld — anchor status: not_found]
- [methods] noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] the spectrum is further restricted to its 50 most intense peaks: "the spectrum is further restricted to its 50 most intense peaks"
- [methods] any spectrum that contains fewer than 10 peaks remaining or with a mass range less than 250 Da is discarded: "any spectrum that contains fewer than 10 peaks remaining or with a mass range less than 250 Da is discarded"
- [methods] a candidate selection step during which a subset of spectra in the spectral library are selected as candidate matches: "a candidate selection step during which a subset of spectra in the spectral library are selected as candidate matches"
- [readme] ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra to compare to an unknown query"
