---
name: spectral-graph-theory-transition-matrices
description: Use when two fragmentation spectra need a pair-specific fragment-ion
  similarity that tolerates non-identical m/z values and does not rely on precursor
  ion neutral losses, e.g. inputs to spectral alignment or modified-cosine alternatives.
when_to_use_negative:
- The analysis requires precursor-ion neutral-loss features; the original skill uses
  MZ difference counts only (precursor-based neutral loss difference counts are a
  separate SIMILE V2 capability)
- A global substitution matrix over all possible m/z values is desired — the paper
  argues this would be infinite; this skill is restricted to a single pair of spectra
- Downstream analysis expects cosine similarity scores rather than a Laplacian-embedding
  similarity matrix
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIMILE
  role: Provides sml.similarity_matrix, which builds the mass-difference transition
    matrix, graph Laplacian, and pseudoinverse to emit S and spec_ids
  repo: https://github.com/biorack/simile
- name: numpy
  role: Numerical backend for the difference count matrix, Laplacian pseudoinverse
    computation, and saving S/spec_ids as .npy files
- name: scipy
  role: Python dependency of the SIMILE environment used for the spectral graph computations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:dcaa79c72c57496e4924aab88b5bd37760b069762c77044702ba55b1a52cbf60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - SIMILE
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-graph-theory-transition-matrices

## Summary

Builds a pair-specific fragment-ion similarity matrix from the all-by-all m/z difference counts of two tandem MS spectra by converting row-normalized mass-difference transition probabilities into the pseudoinverse of their graph Laplacian (average-commute-time distance), as implemented in SIMILE's sml.similarity_matrix.

## When to use

Use when two fragmentation spectra need a pair-specific fragment-ion similarity that tolerates non-identical m/z values and does not rely on precursor ion neutral losses, e.g. inputs to spectral alignment or modified-cosine alternatives.

## When NOT to use

- The analysis requires precursor-ion neutral-loss features; the original skill uses MZ difference counts only (precursor-based neutral loss difference counts are a separate SIMILE V2 capability)
- A global substitution matrix over all possible m/z values is desired — the paper argues this would be infinite; this skill is restricted to a single pair of spectra
- Downstream analysis expects cosine similarity scores rather than a Laplacian-embedding similarity matrix

## Inputs

- Ordered fragment ion m/z lists (mzs) for two tandem MS spectra (e.g. from pyteomics/ms2-like parsing of MassBank MGF entries)
- Precursor m/z values (pmzs) for both spectra
- User-supplied m/z tolerance for counting pairwise mass differences (value not specified in the article snippet)

## Outputs

- Similarity matrix S of shape (m+n) x (m+n) covering the union of fragment ions from both spectra
- spec_ids array marking which rows/columns of S belong to each input spectrum
- Persisted artifacts: similarity_matrix.npy and spec_ids.npy (numpy files)

## How to apply

Parse two tandem MS records (e.g. MassBank MGF entries) to extract ordered fragment m/z lists (mzs) and precursor m/z values (pmzs). Build an (m+n) x (m+n) m/z difference count matrix over the union of fragment ions, counting pairwise mass differences within a user-supplied m/z tolerance. Row-normalize the mass-difference frequencies into transition probabilities, which encodes a shortest-path notion of fragment-ion relatedness (fragments are similar if their mass difference is common). Convert the transition matrix into its normalized graph Laplacian and compute the Moore-Penrose pseudoinverse, yielding an average-commute-time similarity matrix S in which ions sharing ancestors/descendents are scored as more similar. Call sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance) to obtain S and spec_ids, saving them as similarity_matrix.npy and spec_ids.npy via numpy.

## Related tools

- **SIMILE** (Provides sml.similarity_matrix, which builds the mass-difference transition matrix, graph Laplacian, and pseudoinverse to emit S and spec_ids) — https://github.com/biorack/simile
- **numpy** (Numerical backend for the difference count matrix, Laplacian pseudoinverse computation, and saving S/spec_ids as .npy files)
- **scipy** (Python dependency of the SIMILE environment used for the spectral graph computations)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)
```

## Evaluation signals

- similarity_matrix.npy exists and its shape is (m+n) x (m+n), matching the union of the two peak lists
- All entries of S are finite numeric values (no NaN/inf)
- spec_ids.npy partitions the rows/columns of S correctly between the two input spectra
- S should reflect high similarity for fragment pairs whose m/z difference is a common mass delta within the supplied tolerance
- The output feeds forward cleanly: sml.multiple_match(S, spec_ids) must accept S and spec_ids without dimensional errors

## Limitations

- The paper reports no quantitative acceptance threshold for this step; only dimensional/finite-value QC is available as the acceptance criterion
- The tolerance parameter value used in the article snippet is not specified
- SIMILE (as published) is limited to protonation/deprotonation analysis; additional adducts are left to future research
- This similarity matrix alone lacks statistical significance — intermediate similarity scores are uninterpretable without the downstream z_test step

## Evidence

- [methods] Generate fragmentation similarity matrix (S, spec_ids) via sml.similarity_matrix: "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [readme] Transition matrix rationale (shortest path): "This is satisfied by constructing a transition matrix with row-normalized mass difference frequencies as transition probabilites."
- [readme] Laplacian pseudoinverse rationale (average commute time): "This is satisfied by converting the transition matrix of (a) into the pseudo-inverse of its (normalized) laplacian."
- [intro] Pair-specific vs global substitution matrix motivation: "if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive, and"
- [discussion] Limitation: protonation/deprotonation only: "SIMILE is currently limited to analysis of protonation/deprotonation, which is certainly a limitation."
