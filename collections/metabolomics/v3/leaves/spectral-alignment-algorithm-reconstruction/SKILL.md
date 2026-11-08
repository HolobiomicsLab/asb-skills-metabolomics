---
name: spectral-alignment-algorithm-reconstruction
description: Use when fragment ion matches must be computed between spectra from a
  pair-specific similarity matrix and you need the maximum weight matching (matching
  matrix M) prior to significance testing.
when_to_use_negative:
- You need statistical significance or p-values for alignments — use the downstream
  sml.z_test step instead; M alone is an uninterpretable intermediate similarity score
  without significance estimation.
- Only cosine-based spectral similarity between intact spectra is needed — SIMILE
  complements rather than replaces cosine-based scoring for compound identification.
- Analyzing adducts beyond protonation/deprotonation — SIMILE is currently limited
  to protonation/deprotonation analysis.
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIMILE
  role: Python library whose sml.multiple_match function computes the maximum weight
    matching over the similarity matrix S using spec_ids to separate intra- from inter-spectral
    comparisons
  repo: https://mybinder.org/v2/gh/biorack/simile/HEAD
- name: BLAST
  role: Cited as one of the protein-sequence alignment algorithms whose substitution/gap
    alignment concept SIMILE's matching adapts to fragment ion spectra
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:8a4cc927c543a7cb6ab56ec1315abb0c7b98fce79e647343041a217972575005
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - SIMILE
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# spectral-alignment-algorithm-reconstruction

## Summary

Reconstruct SIMILE's spectral alignment pipeline by deriving a maximum weight matching over a pair-specific fragment ion similarity matrix, replacing the original monotonic alignment approach used in protein-sequence-style alignment of tandem MS/MS spectra.

## When to use

Use when fragment ion matches must be computed between spectra from a pair-specific similarity matrix and you need the maximum weight matching (matching matrix M) prior to significance testing.

## When NOT to use

- You need statistical significance or p-values for alignments — use the downstream sml.z_test step instead; M alone is an uninterpretable intermediate similarity score without significance estimation.
- Only cosine-based spectral similarity between intact spectra is needed — SIMILE complements rather than replaces cosine-based scoring for compound identification.
- Analyzing adducts beyond protonation/deprotonation — SIMILE is currently limited to protonation/deprotonation analysis.

## Inputs

- Fragment ion similarity matrix S (numpy array from sml.similarity_matrix, built from m/z difference counts with tolerance parameter)
- Spectral identifier array spec_ids assigning fragment ions to their source spectra
- Peak m/z values (mzs) and precursor m/z values (pmzs) that generated S

## Outputs

- Matching matrix M of max weight matches with the same dimensions/indexing as S
- matching_matrix.npy (saved matching matrix)
- Matched fragment ion pairs usable downstream by sml.sym_compare and sml.z_test

## How to apply

Load the fragment ion similarity matrix S and the spectral identifier array spec_ids produced by sml.similarity_matrix, which parameterizes similarity via all-by-all m/z difference counts (and, in V2, precursor-based neutral loss difference counts). Formulate the matching problem as a maximum weight matching over S, where candidate edges are similarity entries between fragment ions belonging to different spectra distinguished by spec_ids — analogous to the substitution/gap framework of protein-sequence alignment algorithms like Needleman–Wunsch and Smith–Waterman. Run a maximum-weight-matching algorithm (SIMILE V2 uses maximum weight matching instead of the original monotonic alignment method, with improved performance) to select mutually consistent, highest-scoring fragment ion pairings. Assemble the pairings into the matching matrix M with the same indexing as S and spec_ids and save it as matching_matrix.npy; note that multiple matching is also available in V2 for fragment-centric analyses.

## Related tools

- **SIMILE** (Python library whose sml.multiple_match function computes the maximum weight matching over the similarity matrix S using spec_ids to separate intra- from inter-spectral comparisons) — https://mybinder.org/v2/gh/biorack/simile/HEAD
- **BLAST** (Cited as one of the protein-sequence alignment algorithms whose substitution/gap alignment concept SIMILE's matching adapts to fragment ion spectra)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance); M = sml.multiple_match(S, spec_ids)
```

## Evaluation signals

- M has the same dimensions as S (matching matrix indexes into the similarity matrix)
- Matched ion pairs are consistent with spec_ids — matches occur between different spectra, not within one spectrum
- The script runs end-to-end after similarity_matrix without errors and saves matching_matrix.npy
- Pairings selected are mutually consistent (maximum weight matching) rather than an arbitrary monotonic alignment
- Downstream sym_compare matrix C yields symmetric matches as 1 (pro) and asymmetric matches as -1 (con), confirming M structure

## Limitations

- Qualitative acceptance only — the paper reports no quantitative threshold for validating M; verification is that dimensions match S and pairings are consistent with spec_ids
- SIMILE is limited to protonation/deprotonation analysis; addressing additional adducts is left to future research
- Parameters used to compare alignment algorithms were not optimized; further optimization could trade off specificity vs. number of hits
- Without downstream statistical significance, intermediate similarity scores are to some degree uninterpretable
- Python is pinned to 3.7 due to non-SIMILE bugs

## Evidence

- [readme] V2 replaces monotonic alignment with maximum weight matching for improved performance: "Maximum weight matching is used instead of original monotonic alignment method with improved performance"
- [methods] The max weight matching step is invoked via sml.multiple_match on the similarity matrix: "M = sml.multiple_match(S, spec_ids)"
- [methods] The similarity matrix is generated from m/z values with a tolerance parameter before matching: "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [other] The matching step computes fragment ion matches using maximum weight matching on the similarity matrix: [evidence span withheld — anchor status: not_found]
- [intro] SIMILE leverages protein-sequence alignment methods for pairwise alignment of fragmentation spectra: "leverages methods used for protein-sequence alignment to enable robust pairwise alignment of fragmentation spectra with p-value estimation"
