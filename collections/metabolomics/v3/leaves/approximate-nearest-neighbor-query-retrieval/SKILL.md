---
name: approximate-nearest-neighbor-query-retrieval
description: Use when an open modification spectral library search needs to preselect
  a limited, approximate set of most-similar library spectra per query before accurate
  spectrum-spectrum match scoring.
when_to_use_negative:
- The unmodified-peptide first level of the cascade search, where a standard library
  search rather than ANN candidate selection is used
- An exactness-critical nearest-neighbor task where the approximation of tree-based
  retrieval cannot be tolerated (this stage relies on returning many approximate candidates)
- Spectra that do not survive preprocessing (fewer than 10 peaks, mass range below
  250 Da) and therefore cannot be meaningfully vectorized and indexed
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ANN-SoLo
  role: Implements the ANN index (random projection tree ensemble) used to select
    candidate library spectra for each query during the open search
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Python
  role: Language in which ANN-SoLo's candidate selection and retrieval scripts are
    implemented
- name: Faiss
  role: General-purpose dense-vector similarity search library surfaced in the repository
    context; the paper's index itself is built from random projection trees, and other
    ANN techniques were left as future work
  repo: https://github.com/facebookresearch/faiss
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:451332e5cc67d304002b2917de3ea6722646a1611252e4bc45883df7fcc0fbb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/ANN-SoLo
  tools:
  - ANN-SoLo
  - Faiss
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# approximate-nearest-neighbor-query-retrieval

## Summary

Reconstructs ANN-SoLo's approximate nearest neighbor candidate-selection stage: library mass spectra are vectorized and indexed with an ensemble of random projection trees so that each query spectrum retrieves only a limited set of most-similar library spectra for downstream shifted dot product scoring.

## When to use

Use when an open modification spectral library search needs to preselect a limited, approximate set of most-similar library spectra per query before accurate spectrum-spectrum match scoring.

## When NOT to use

- The unmodified-peptide first level of the cascade search, where a standard library search rather than ANN candidate selection is used
- An exactness-critical nearest-neighbor task where the approximation of tree-based retrieval cannot be tolerated (this stage relies on returning many approximate candidates)
- Spectra that do not survive preprocessing (fewer than 10 peaks, mass range below 250 Da) and therefore cannot be meaningfully vectorized and indexed

## Inputs

- Preprocessed query MS/MS spectra (precursor and low-intensity peaks removed, top-50 peaks retained, rank-transformed intensities)
- Spectral library spectra preprocessed identically (e.g. NIST human HCD library 2016/09/12, MassIVE-KB 2017/11/27, TripleTOF yeast library)
- ANN index hyperparameters: number of trees (100-1000) and number of tree nodes inspected per query (~20000-400000)

## Outputs

- ann_candidates.csv listing, per query spectrum, retrieved candidate library spectra identifiers and their coarse vector similarities
- Candidate set fed to the candidate ranking step for shifted dot product SSM scoring

## How to apply

Preprocess query and library spectra identically: remove precursor and noise peaks below 1% of the base-peak intensity, keep only the 50 most intense peaks, discard spectra with fewer than 10 peaks or a mass range below 250 Da, and rank-transform peak intensities. Vectorize each library spectrum into a sparse 1 Da-binned vector with intensities summed per bin and normalize it to unit length so similarity is a dot product. Build an ensemble of random projection trees over these library vectors, recursively partitioning the space with random split hyperplanes built from two randomly sampled points (the paper's iPRG2012 settings use 100-1000 trees). For each query vector, traverse every tree to its data subspace and combine subspaces across trees (about 20000-400000 nodes inspected per query) to assemble the candidate set. Approximate retrieval is acceptable here because the candidate selection step returns a sufficiently large set of candidates that the subsequent candidate ranking (SSM scoring) stage recovers true matches.

## Related tools

- **ANN-SoLo** (Implements the ANN index (random projection tree ensemble) used to select candidate library spectra for each query during the open search) — https://github.com/bittremieux/ANN-SoLo
- **Python** (Language in which ANN-SoLo's candidate selection and retrieval scripts are implemented)
- **Faiss** (General-purpose dense-vector similarity search library surfaced in the repository context; the paper's index itself is built from random projection trees, and other ANN techniques were left as future work) — https://github.com/facebookresearch/faiss

## Evaluation signals

- ann_candidates.csv exists and is well-formed, containing per-query candidate library spectra identifiers and coarse similarities
- On a sample of queries, ANN retrieval recovers a high-recall candidate set relative to an exact brute-force nearest-neighbor check
- Index construction uses a tree count within the paper's 100-1000 tree range and queries inspect roughly 20000-400000 tree nodes
- Query and library spectra underwent identical preprocessing (1% base-peak intensity cutoff, top-50 peaks, >=10 peaks, >=250 Da mass range)
- Retrieved candidates are only a limited subset of the library, consistent with the candidate-selection purpose before SSM ranking

## Limitations

- Approximate retrieval may miss the true nearest neighbors; the design relies on returning a sufficiently high number of candidates so ranking recovers matches
- Alternative ANN techniques (HNSW, LSH, product quantization) were not investigated for spectral library searching and remain future work
- GPU acceleration of candidate selection was not implemented in the 2018 version

## Evidence

- [intro] ANN index purpose: "During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum"
- [methods] Identical preprocessing of queries and library: "Prior to spectral library searching both the query spectra and library spectra are similarly preprocessed"
- [methods] Noise-peak removal threshold: "noise peaks with an intensity below 1 % of the intensity of the most intense peak are removed"
- [methods] Candidate selection definition: "a candidate selection step during which a subset of spectra in the spectral library are selected as candidate matches"
- [readme] README description of ANN indexing role: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra"
