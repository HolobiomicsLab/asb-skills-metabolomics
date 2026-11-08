---
name: fragment-ion-similarity-matrix-construction
description: Use when you have tandem-MS peak m/z lists (e.g. from MassBank or NIST
  Tandem Mass Spectral Library) plus precursor m/z values for one or more spectra
  and need a quantitative similarity measure between every pair of fragment ions before
  running spectral alignment or significance testing.
when_to_use_negative:
- Input is already an aligned spectral similarity/matching matrix — this skill builds
  the similarity matrix from raw peak lists, not from pre-existing alignments
- Only a global (pair-independent) fragment-ion substitution matrix is needed; SIMILE
  deliberately avoids this because a global substitution matrix over infinite possible
  m/z values would be infeasible
- Need de novo structure elucidation rather than spectrum-pair alignment; SIMILE provides
  significance of spectral relationships, not chemical structure assignment
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: SIMILE
  role: Provides sml.similarity_matrix, the function that computes the fragment-ion
    similarity matrix and spectrum-identifier array from peak and precursor m/z lists
  repo: https://github.com/biorack/simile
- name: NumPy
  role: Dependency used for array handling of the m/z lists and the resulting similarity
    matrix
- name: SciPy
  role: Dependency used for sparse/linear-algebra operations underlying the Laplacian
    pseudo-inverse computation
provenance:
  source_task_ids:
  - task_004
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:7efd0c28fb1639e764e4f3510b7e24dd92e14818adc6f0d246ad6e599122836a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - NumPy
  - SIMILE
  - SciPy
  - pandas
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fragment-ion-similarity-matrix-construction

## Summary

Constructs a pair-specific fragment-ion similarity matrix S (plus a spectrum-identifier vector spec_ids) from peak m/z lists and precursor m/z values using SIMILE's sml.similarity_matrix, the first step of the SIMILE spectral-alignment pipeline. The matrix scores fragment-ion relatedness by mass-delta frequencies via a Laplacian commute-time distance, and its dimensions and indexing feed directly into max-weight matching and z-test significance estimation.

## When to use

Use when you have tandem-MS peak m/z lists (e.g. from MassBank or NIST Tandem Mass Spectral Library) plus precursor m/z values for one or more spectra and need a quantitative similarity measure between every pair of fragment ions before running spectral alignment or significance testing.

## When NOT to use

- Input is already an aligned spectral similarity/matching matrix — this skill builds the similarity matrix from raw peak lists, not from pre-existing alignments
- Only a global (pair-independent) fragment-ion substitution matrix is needed; SIMILE deliberately avoids this because a global substitution matrix over infinite possible m/z values would be infeasible
- Need de novo structure elucidation rather than spectrum-pair alignment; SIMILE provides significance of spectral relationships, not chemical structure assignment

## Inputs

- List of peak m/z arrays (one per tandem-MS spectrum, e.g. from MassBank or NIST library records)
- Corresponding precursor m/z values (pmzs)
- Mass tolerance parameter for counting m/z differences

## Outputs

- Similarity matrix S (all-by-all fragment-ion similarity scores)
- Spectrum-identifier array spec_ids mapping each ion row/column to its source spectrum

## How to apply

Call sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance) where mzs is a list of peak m/z arrays (one per spectrum) and pmzs the corresponding precursor m/z values; the tolerance parameter (value not specified in the snippet, left configurable) controls the m/z window for counting mass deltas between ions. Internally the function builds a transition matrix with row-normalized mass-difference frequencies as transition probabilities (a 'shortest path' distance), then converts it into the pseudo-inverse of its normalized Laplacian to obtain an 'average commute time' distance that captures ancestor/descendant ion relationships (property b in the README theory section). The function returns the similarity matrix S and the spectrum-identifier array spec_ids, which labels which spectrum each ion belongs to so that downstream matching (sml.multiple_match) and significance testing (sml.z_test) can distinguish intra- vs inter-spectral comparisons. Validate that S has shape (total_ions, total_ions) and that spec_ids partition S's rows consistently with the input spectra.

## Related tools

- **SIMILE** (Provides sml.similarity_matrix, the function that computes the fragment-ion similarity matrix and spectrum-identifier array from peak and precursor m/z lists) — https://github.com/biorack/simile
- **NumPy** (Dependency used for array handling of the m/z lists and the resulting similarity matrix)
- **SciPy** (Dependency used for sparse/linear-algebra operations underlying the Laplacian pseudo-inverse computation)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)
```

## Evaluation signals

- S is square with dimension equal to the total number of fragment ions across all input spectra
- spec_ids is a 1-D array whose length equals S's dimension and whose values partition rows into their source spectra
- Diagonal of S corresponds to intra-spectral ion comparisons (same spec_id) — these later feed the null distribution
- Off-diagonal S entries should be higher when the m/z difference between two ions is a common mass delta (per the mass-difference-frequency transition matrix)
- Downstream sml.multiple_match on S must produce a matching matrix M with the same dimensions as S, confirming S is well-formed for matching

## Limitations

- The tolerance parameter value is not specified in the methods snippet and must be chosen by the practitioner
- SIMILE is currently limited to protonation/deprotonation analysis — sodium adducts and other adduct types are left to future work
- Intermediate similarity scores without statistical significance (from downstream z_test) are uninterpretable for prioritizing metabolites
- The paper notes some molecule classes may be more appropriate for cosine-based scoring than SIMILE-based scoring
- Conda environment is pinned to python 3.7 due to non-SIMILE bugs

## Evidence

- [methods] Generate fragmentation similarity matrix S via sml.similarity_matrix: "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [readme] README theory: mass-difference transition matrix: "Property (a) is satisfied by constructing a transition matrix with row-normalized mass difference frequencies as transition probabilites."
- [readme] README theory: Laplacian commute-time distance: "Property (b) is satisfied by converting the transition matrix of (a) into the pseudo-inverse of its (normalized) laplacian."
- [readme] README usage signature: "# Generate fragmentation similarity matrix
S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [intro] Paper intro: pair-specific matrix rationale: "if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive, and"
