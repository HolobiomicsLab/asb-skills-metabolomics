---
name: fragment-ion-similarity-scoring
description: Use when comparing two fragmentation spectra whose fragment m/z values
  do not match exactly and whose precursor ion neutral losses differ (e.g., structurally
  related compounds with multiple modifications), where cosine scoring fails.
when_to_use_negative:
- Spectra are from compounds differing by only one localized modification — cosine-based
  scoring is the article's recommended complement and may be more appropriate for
  some molecule classes.
- Analysis requires adducts other than [M+H]+/[M-H]- — SIMILE is currently limited
  to protonation/deprotonation analysis.
- Only a global, dataset-wide fragment substitution matrix is needed — this skill
  produces a pair-specific, context-sensitive matrix, not a reusable global one.
edam_operation: null
tools:
- name: SIMILE
  role: Python library providing sml.similarity_matrix, which builds the m/z difference
    count matrix, transition matrix, and Laplacian pseudoinverse to return S and spec_ids
  repo: https://github.com/biorack/simile
- name: numpy
  role: Dependency used for array storage of S and spec_ids (e.g., similarity_matrix.npy,
    spec_ids.npy) and downstream numeric checks
- name: scipy
  role: Numeric dependency for linear-algebra operations underlying the Laplacian
    pseudoinverse computation
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:baf08168f2ec7a23101e191d904527e4ce7f78e6563a61e6b98cfdb8cc480394
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - GNPS
  - SIMILE
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fragment-ion-similarity-scoring

## Summary

Computes a pair-specific fragment ion similarity matrix S from two tandem MS spectra by building an (m+n)×(m+n) pairwise m/z-difference count matrix, row-normalizing it into transition probabilities, and taking the Moore-Penrose pseudoinverse of the normalized graph Laplacian to get an average-commute-time similarity between all fragment ions.

## When to use

Use when comparing two fragmentation spectra whose fragment m/z values do not match exactly and whose precursor ion neutral losses differ (e.g., structurally related compounds with multiple modifications), where cosine scoring fails.

## When NOT to use

- Spectra are from compounds differing by only one localized modification — cosine-based scoring is the article's recommended complement and may be more appropriate for some molecule classes.
- Analysis requires adducts other than [M+H]+/[M-H]- — SIMILE is currently limited to protonation/deprotonation analysis.
- Only a global, dataset-wide fragment substitution matrix is needed — this skill produces a pair-specific, context-sensitive matrix, not a reusable global one.

## Inputs

- mzs: ordered lists of fragment ion m/z values for the two spectra (e.g., parsed from MassBank or GNPS MGF entries)
- pmzs: precursor m/z values for each input spectrum
- tolerance: m/z tolerance used when counting pairwise mass differences

## Outputs

- S: (m+n)×(m+n) fragment ion similarity matrix (pseudoinverse of normalized graph Laplacian; average-commute-time distances), saveable as similarity_matrix.npy
- spec_ids: array marking spectrum membership of each row/column of S, saveable as spec_ids.npy

## How to apply

Parse two tandem MS spectra (e.g., MassBank/GNPS MGF entries) with pyteomics to extract ordered fragment m/z lists (mzs) and precursor m/z values (pmzs). Call sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance): the tool counts all pairwise m/z differences within the user-supplied tolerance over the union of the two peak lists, row-normalizes the difference frequencies into transition probabilities encoding a shortest-path notion of fragment relatedness, then converts the transition matrix into the normalized graph Laplacian and computes its Moore-Penrose pseudoinverse, yielding an average-commute-time similarity matrix S of shape (m+n)×(m+n). The returned spec_ids vector marks which rows/columns belong to each input spectrum and must be carried forward into sml.multiple_match, sml.sym_compare, and sml.z_test. The pair-specific (rather than global) parameterization is essential because a global substitution matrix over all possible fragment m/z values would be infinite; restricting to one spectrum pair keeps the matrix finite and context-sensitive.

## Related tools

- **SIMILE** (Python library providing sml.similarity_matrix, which builds the m/z difference count matrix, transition matrix, and Laplacian pseudoinverse to return S and spec_ids) — https://github.com/biorack/simile
- **numpy** (Dependency used for array storage of S and spec_ids (e.g., similarity_matrix.npy, spec_ids.npy) and downstream numeric checks)
- **scipy** (Numeric dependency for linear-algebra operations underlying the Laplacian pseudoinverse computation)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)
```

## Evaluation signals

- similarity_matrix.npy exists and has shape (m+n)×(m+n), matching the size of the union of the two fragment peak lists.
- All entries of S are finite numeric values (no NaN/Inf); the paper reports no quantitative acceptance threshold, so dimensional/finite-value QC is the acceptance criterion.
- spec_ids.npy correctly partitions rows/columns of S into the two spectra, consistent with the input mzs lengths.
- Downstream consistency: sml.multiple_match(S, spec_ids) and sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5) accept S without shape errors, confirming spec_ids alignment.
- Property check on the theory: fragment ions sharing common mass differences should show higher similarity (shortest-path via row-normalized transition probabilities) and ions with shared ancestors/descendents should be closer (average commute time).

## Limitations

- The tolerance parameter value used in the article's code snippet is not specified; users must supply an m/z tolerance appropriate to their instrument resolution.
- SIMILE is limited to protonation/deprotonation analysis; other adducts are not addressed.
- Parameter choices were not optimized in the study, and optimization involves unquantified trade-offs (specificity vs. number of hits).
- Significant SIMILE alignment does not necessitate compound structural similarity, though significant alignments often correspond to moderate-to-high structural similarity.

## Evidence

- [methods] pair-specific fragment ion similarity via Laplacian pseudoinverse: "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [readme] transition matrix construction from mass difference frequencies: "Property (a) is satisfied by constructing a transition matrix with row-normalized mass difference frequencies as transition probabilites."
- [readme] average commute time distance rationale: "Property (b) is satisfied by converting the transition matrix of (a) into the pseudo-inverse of its (normalized) laplacian."
- [intro] pair-specific matrix rationale from intro: "if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive, and"
