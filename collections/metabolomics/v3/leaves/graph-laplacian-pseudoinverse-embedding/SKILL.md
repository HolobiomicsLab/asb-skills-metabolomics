---
name: graph-laplacian-pseudoinverse-embedding
description: Use when two tandem MS spectra (e.g., MassBank MGF entries) must be aligned
  via a pair-specific fragment similarity built from all pairwise m/z differences
  within a tolerance, especially when compounds differ by multiple structural changes.
when_to_use_negative:
- The spectra involve adducts other than protonation/deprotonation — the paper states
  SIMILE 'is currently limited to analysis of protonation/deprotonation'
- The goal is standard compound identification where established cosine-based scoring
  suffices — the authors recommend complementing SIMILE with cosine-based scoring,
  and note some molecule classes may be more appropriate for cosine scoring
- A single global fragment-ion substitution matrix across many spectra pairs is required
  — the paper motivates the pair-specific approach precisely because a global matrix
  would be infinite due to the infinite number of possible m/z values
tools:
- name: SIMILE
  role: Python library whose sml.similarity_matrix function implements the m/z difference
    counting, transition-matrix construction, and Laplacian pseudoinverse embedding
  repo: https://mybinder.org/v2/gh/biorack/simile/HEAD
- name: numpy
  role: Saving the similarity matrix S and spec_ids as .npy files; core dependency
    of the SIMILE environment
- name: scipy
  role: Core dependency of the SIMILE environment used for the linear-algebra steps
    (pseudoinverse/Laplacian computation)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:d201b9a8df10413813d9260b881e73bccd0e8433cd622e25e59839e18b73877e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SIMILE
  - numpy
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# graph-laplacian-pseudoinverse-embedding

## Summary

Generates a pair-specific fragment-ion similarity matrix (S) from two tandem MS fragmentation spectra by building an all-by-all m/z difference count matrix, row-normalizing it into transition probabilities, and taking the pseudoinverse of its normalized graph Laplacian. This yields an 'average commute time' fragment similarity that handles multiple structural differences without requiring identical m/z values or shared precursor neutral losses.

## When to use

Use when two tandem MS spectra (e.g., MassBank MGF entries) must be aligned via a pair-specific fragment similarity built from all pairwise m/z differences within a tolerance, especially when compounds differ by multiple structural changes.

## When NOT to use

- The spectra involve adducts other than protonation/deprotonation — the paper states SIMILE 'is currently limited to analysis of protonation/deprotonation'
- The goal is standard compound identification where established cosine-based scoring suffices — the authors recommend complementing SIMILE with cosine-based scoring, and note some molecule classes may be more appropriate for cosine scoring
- A single global fragment-ion substitution matrix across many spectra pairs is required — the paper motivates the pair-specific approach precisely because a global matrix would be infinite due to the infinite number of possible m/z values

## Inputs

- Ordered fragment ion m/z lists (mzs) for two tandem MS spectra, e.g., parsed from MassBank MGF records via pyteomics/ms2 parsing
- Precursor m/z values (pmzs) for the two spectra
- Scalar m/z tolerance (tolerance) for counting pairwise mass differences

## Outputs

- Fragment-ion similarity matrix S of shape (m+n) x (m+n) over the union of the two peak lists, containing finite numeric entries (saved as similarity_matrix.npy)
- spec_ids array marking which rows/columns of S belong to each input spectrum (saved as spec_ids.npy)

## How to apply

Parse two tandem MS records with pyteomics/ms2-like parsing to extract ordered fragment m/z lists (mzs) and precursor m/z values (pmzs). Build an (m+n) x (m+n) m/z difference count matrix over the union of fragment ions, counting pairwise mass differences within the user-supplied m/z tolerance (optionally augmented with precursor-based neutral loss difference counts in SIMILE V2). Row-normalize the mass difference frequencies into transition probabilities — a 'shortest path' notion of fragment relatedness satisfying the property that ions with common mass differences are similar. Convert the transition matrix into the normalized graph Laplacian and compute its Moore-Penrose pseudoinverse, giving the 'average commute time' similarity so that ions sharing ancestors/descendants score higher. Record spec_ids marking which rows/columns belong to each spectrum, and save S and spec_ids as .npy files with numpy. The tolerance parameter is required but its value is not specified in the paper's snippet; choose it from the m/z accuracy of the source library.

## Related tools

- **SIMILE** (Python library whose sml.similarity_matrix function implements the m/z difference counting, transition-matrix construction, and Laplacian pseudoinverse embedding) — https://mybinder.org/v2/gh/biorack/simile/HEAD
- **numpy** (Saving the similarity matrix S and spec_ids as .npy files; core dependency of the SIMILE environment)
- **scipy** (Core dependency of the SIMILE environment used for the linear-algebra steps (pseudoinverse/Laplacian computation))

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)
```

## Evaluation signals

- similarity_matrix.npy exists and its shape is (m+n) x (m+n), matching the union of the two input peak lists
- All entries of S are finite numeric values (no NaN/inf); the paper reports no quantitative acceptance threshold, so dimensional/finite-value QC is the acceptance criterion
- spec_ids.npy has length m+n and partitions rows/columns correctly between the two input spectra
- The matrix is derived from row-normalized mass difference frequencies, so each row of the underlying transition matrix sums to 1 before Laplacian conversion
- Downstream sanity check: feeding S into sml.multiple_match and sml.z_test should yield a valid matching M and a finite p-value/null_dist

## Limitations

- SIMILE is limited to protonation/deprotonation analysis; comprehensively addressing additional adducts is left to future research
- Intermediate similarity scores without statistical significance are uninterpretable for prioritizing metabolites — this matrix step alone does not provide p-values (see sml.z_test)
- The paper notes close attention is required since this is the first application of scoring metabolite-fragmentation spectra using a similarity matrix; it should be complemented with cosine-based scoring
- The tolerance parameter value used in the published snippet is not specified, so reproducibility depends on choosing an m/z tolerance consistent with the source library

## Evidence

- [readme] row-normalized mass difference frequencies as transition probabilities: "Property (a) is satisfied by constructing a transition matrix with row-normalized mass difference
frequencies as transition probabilites."
- [readme] pseudoinverse of normalized laplacian gives average commute time distance: "Property (b) is satisfied by converting the transition matrix of (a) into the pseudo-inverse of its (normalized) laplacian. This corresponds to an "average commute time" distance"
- [methods] similarity_matrix call: "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [intro] pair-specific finite substitution motivation: "if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive"
