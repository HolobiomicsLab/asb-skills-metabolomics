---
name: ann-recall-versus-speed-tuning
description: Use when configuring or re-running the ANN candidate-selection stage
  of an ANN-SoLo open search on large spectral libraries (e.g., MassIVE-KB or NIST
  human HCD), where you must trade recall of candidate library spectra against query
  traversal cost.
when_to_use_negative:
- You are running the standard (closed) search level of the cascade, where candidate
  selection is done by a precursor mass filter rather than the ANN index
- The library is small enough that exact brute-force nearest-neighbor retrieval over
  all library spectra is affordable — approximate indexing adds complexity without
  need
- You need exact nearest-neighbor ordering rather than a high-recall candidate subset;
  the ANN stage deliberately trades precision of ranking for speed
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3520
tools:
- name: ANN-SoLo
  role: Implements the ANN index (random projection trees) used for candidate selection
    in the open search; both tuned hyperparameters are ANN-SoLo search parameters
  repo: https://github.com/bittremieux/ANN-SoLo
- name: Faiss
  role: General-purpose similarity-search library surfaced alongside the skill; relevant
    when evaluating alternative ANN indexing techniques (HNSW, LSH, product quantization)
    that the paper leaves as future work
  repo: facebookresearch/faiss
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.8b00359
    title: Fast Open Modification Spectral Library Searching through Approximate Nearest
      Neighbor Indexing
  source_content_hash: sha256:df9b0b3ce13348cc6737ebd69d34dde5dcdc43b42bb109a7d187b0dde4b0dcbf
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# ann-recall-versus-speed-tuning

## Summary

Tunes the two hyperparameters governing ANN-SoLo's approximate nearest neighbor candidate-selection index for open modification spectral library searching: the number of random projection trees built over the library, and the number of tree nodes inspected per query. The goal is a high-recall candidate set at controlled runtime, since the coarse ANN stage only needs to preserve the true best matches, not rank them exactly.

## When to use

Use when configuring or re-running the ANN candidate-selection stage of an ANN-SoLo open search on large spectral libraries (e.g., MassIVE-KB or NIST human HCD), where you must trade recall of candidate library spectra against query traversal cost.

## When NOT to use

- You are running the standard (closed) search level of the cascade, where candidate selection is done by a precursor mass filter rather than the ANN index
- The library is small enough that exact brute-force nearest-neighbor retrieval over all library spectra is affordable — approximate indexing adds complexity without need
- You need exact nearest-neighbor ordering rather than a high-recall candidate subset; the ANN stage deliberately trades precision of ranking for speed

## Inputs

- Preprocessed query spectra (precursor and low-intensity peaks removed, top-50 most intense peaks retained, intensity rank-transformed)
- Preprocessed spectral library vectors (sparse 1 Da mass-bin vectors normalized to unit length), e.g., the NIST human HCD library (version 2016/09/12) or MassIVE-KB library (version 2017/11/27)

## Outputs

- ann_candidates.csv: per query spectrum, the retrieved candidate library spectra identifiers and their coarse vector similarities
- Tuned index configuration (number of trees, number of tree nodes inspected per query)

## How to apply

First preprocess library and query spectra identically — remove noise peaks below 1% of the most intense peak, restrict to the 50 most intense peaks, discard spectra with fewer than 10 peaks or mass range <250 Da, and rank-transform intensities — then vectorize each spectrum into a sparse 1 Da-binned, unit-length-normalized vector. Build the index by constructing an ensemble of random projection trees that recursively partition the library vectors using random split hyperplanes from two randomly sampled points; use roughly 100–1000 trees (paper's iPRG2012 settings). For each query vector, traverse every tree in the ensemble, inspecting approximately 20000–400000 nodes per query, and union the reached data subspaces across trees to assemble the candidate set. Rationale: more trees and more inspected nodes raise recall of the true best library spectra but linearly increase runtime; because candidate selection is followed by a candidate ranking step (shifted dot product SSM scoring), approximate retrieval is tolerated as long as a sufficiently high number of candidates are returned. Sweep the two hyperparameters and check recall against exact brute-force nearest-neighbor retrieval on a query sample before committing to a setting.

## Related tools

- **ANN-SoLo** (Implements the ANN index (random projection trees) used for candidate selection in the open search; both tuned hyperparameters are ANN-SoLo search parameters) — https://github.com/bittremieux/ANN-SoLo
- **Faiss** (General-purpose similarity-search library surfaced alongside the skill; relevant when evaluating alternative ANN indexing techniques (HNSW, LSH, product quantization) that the paper leaves as future work) — facebookresearch/faiss

## Evaluation signals

- ann_candidates.csv exists and is well-formed: one row per query spectrum with candidate library identifiers and coarse similarities
- On a sample of queries, ANN retrieval recovers a high-recall candidate set relative to an exact brute-force nearest-neighbor check, consistent with the paper's rationale that a sufficiently high number of candidates are returned
- Number of trees is within the paper's iPRG2012 settings range (~100–1000) and nodes inspected per query fall in the ~20000–400000 range
- Retrieval recall does not collapse as inspected-node count is lowered for speed; downstream SSM identifications after candidate ranking remain stable

## Limitations

- The candidate set is approximate: overly aggressive settings (few trees / few inspected nodes) can miss true best matches before the shifted dot product ranking stage
- Suitability of alternative ANN techniques (HNSW, LSH, product quantization) for spectral library searching was not investigated — remains future work
- Non-metric space indexing to retrieve candidates directly on the shifted dot product similarity is only mentioned as a potential, untested approach
- GPU acceleration for candidate selection and SSM evaluation is not yet implemented in ANN-SoLo

## Evidence

- [other] ANN index hyperparameters and their roles: [evidence span withheld — anchor status: not_found]
- [intro] ANN index purpose in the open search: "During this open search ANN-SoLo uses an approximate nearest neighbor (ANN) index to efficiently find a limited set of the most similar library spectra for each query spectrum."
- [methods] Candidate selection as a defined stage: "a candidate selection step during which a subset of spectra in the spectral library are selected as candidate matches"
- [readme] Why limited candidate sets suffice: "ANN-SoLo uses approximate nearest neighbor indexing to speed up open modification searching by selecting only a limited number of the most relevant library spectra to compare to an unknown query"
- [discussion] Alternative ANN techniques untested: "An investigation into whether these techniques are suitable to speed up spectral library searching remains as future work."
