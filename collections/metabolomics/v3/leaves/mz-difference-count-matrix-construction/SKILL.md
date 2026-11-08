---
name: mz-difference-count-matrix-construction
description: Use when aligning two tandem MS (MS/MS) spectra whose fragment ion m/z
  values do not coincide exactly and whose precursor neutral losses differ, requiring
  a pair-specific similarity measure between fragment ions.
when_to_use_negative:
- Spectra share no informative common m/z differences and no precursor neutral-loss
  structure — the count matrix becomes a poor basis for aligning dissimilar compounds
- You need compound identification cosine-style scores; SIMILE should be complemented
  with cosine-based scoring rather than replacing it
- The alignment target is small-molecule adduct states beyond protonation/deprotonation;
  SIMILE is currently limited to protonation/deprotonation analysis
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: simile (sml)
  role: Python library whose similarity_matrix() function builds the count matrix,
    transition matrix, Laplacian pseudoinverse, and returns (S, spec_ids); V2 also
    accepts precursor-based neutral loss difference counts
  repo: https://github.com/biorack/simile
- name: numpy
  role: Used to store and persist the similarity matrix and spec_ids arrays (.npy
    format)
- name: scipy
  role: Linear algebra backend for constructing and inverting the normalized graph
    Laplacian
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:8735819b0825e10570bcd5ae0163ff6a7cb6a9985394ee6d731fd9479c7150e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - numpy
  - scipy
  - simile (sml)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# m/z difference count matrix construction

## Summary

Construction of an (m+n)×(m+n) all-by-all m/z difference count matrix over the union of fragment ion peaks from two fragmentation spectra, row-normalized into transition probabilities, and converted into the pseudoinverse of the graph Laplacian to yield the SIMILE fragment ion similarity matrix S.

## When to use

Use when aligning two tandem MS (MS/MS) spectra whose fragment ion m/z values do not coincide exactly and whose precursor neutral losses differ, requiring a pair-specific similarity measure between fragment ions.

## When NOT to use

- Spectra share no informative common m/z differences and no precursor neutral-loss structure — the count matrix becomes a poor basis for aligning dissimilar compounds
- You need compound identification cosine-style scores; SIMILE should be complemented with cosine-based scoring rather than replacing it
- The alignment target is small-molecule adduct states beyond protonation/deprotonation; SIMILE is currently limited to protonation/deprotonation analysis

## Inputs

- Two parsed tandem MS (MS/MS) spectra as ordered fragment ion m/z lists (mzs)
- Precursor m/z values for both spectra (pmzs)
- m/z tolerance (e.g. a fragment-matching tolerance in Da)
- Optionally, precursor-based neutral loss difference counts

## Outputs

- Pair-specific fragment ion similarity matrix S of shape (m+n)×(m+n), with finite numeric entries (e.g. similarity_matrix.npy)
- spec_ids array marking which rows/columns of S belong to each input spectrum (e.g. spec_ids.npy)

## How to apply

Parse two spectra (e.g. from MassBank MGF entries or the NIST Tandem Mass Spectral Library) to extract ordered fragment ion m/z lists (mzs) and precursor m/z values (pmzs). Build an (m+n)×(m+n) matrix whose entries count pairwise m/z differences between all fragment ions of the two spectra within a user-supplied m/z tolerance; this count matrix must be pair-specific because a global substitution matrix over the infinite set of possible m/z values would be infeasible. Row-normalize the difference frequencies to transition probabilities, giving a 'shortest path' notion of fragment-ion relatedness. Convert the transition matrix into its normalized graph Laplacian and compute the Moore–Penrose pseudoinverse, yielding an 'average commute time' fragment ion similarity matrix S that satisfies the two expected fragmentation properties: fragment ions are similar if their mass difference is common, and if their ancestor/descendent fragment ions are similar. Optionally include precursor-based neutral loss difference counts alongside the m/z difference counts (a V2 feature of the simile package). Record spec_ids marking which rows/columns of S belong to each input spectrum, then save S and spec_ids (e.g. similarity_matrix.npy, spec_ids.npy).

## Related tools

- **simile (sml)** (Python library whose similarity_matrix() function builds the count matrix, transition matrix, Laplacian pseudoinverse, and returns (S, spec_ids); V2 also accepts precursor-based neutral loss difference counts) — https://github.com/biorack/simile
- **numpy** (Used to store and persist the similarity matrix and spec_ids arrays (.npy format))
- **scipy** (Linear algebra backend for constructing and inverting the normalized graph Laplacian)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)
```

## Evaluation signals

- Verify similarity_matrix.npy exists and has shape (m+n)×(m+n) matching the union of the two peak lists
- Verify all entries of S are finite numeric values (no NaN/inf after Laplacian pseudoinverse)
- Verify spec_ids.npy has the same length as S and correctly partitions rows/columns between the two input spectra
- Check that the similarity_matrix() call returns both S and spec_ids (the documented API contract)
- Optionally downstream-check that the resulting S supports a non-empty max weight matching via sml.multiple_match

## Limitations

- SIMILE is currently limited to analysis of protonation/deprotonation; comprehensively addressing additional adducts is left to future research
- Parameters used to compare alignment algorithms were not optimized; further optimization is left as future work
- Parameter optimization entails unquantified trade-offs (e.g. improving specificity vs. number of hits)
- A global substitution matrix for fragment ions would be infinite due to the infinite number of possible m/z values, so the count matrix is necessarily pair-specific and must be rebuilt for every new spectrum pair
- A fragment ion similarity matrix alone does not yield compound identification; significant SIMILE alignment of fragmentation spectra does not necessitate compound structure similarity

## Evidence

- [methods] Pair-specific fragment ion similarity matrices parameterized by all-by-all m/z difference counts between fragment ions: [evidence span withheld — anchor status: not_found]
- [readme] Fragment ions are similar if the difference in mass between them is common.: "Fragment ions are similar if the difference in mass between them is common."
- [readme] S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance): "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [methods] MUST use vocabulary specific to this paper: [evidence span withheld — anchor status: not_found]
