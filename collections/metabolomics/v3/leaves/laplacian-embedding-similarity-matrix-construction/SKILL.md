---
name: laplacian-embedding-similarity-matrix-construction
description: Use when input is a collection of MS/MS spectra (fragment m/z lists and
  precursor m/z values) and you need a pair-specific fragment-ion similarity matrix
  that tolerates multiple structural differences before spectral alignment or significance
  testing.
when_to_use_negative:
- Spectra whose adducts are other than protonation/deprotonation — the paper states
  SIMILE 'is currently limited to analysis of protonation/deprotonation, which is
  certainly a limitation'.
- Compound classes the authors flag as potentially better served by cosine-based scoring,
  where a modified-cosine similarity is the more appropriate primary measure.
- Cases where you only need plain peak-matching similarity without the fragment-similarity
  graph embedding (i.e., cosine-based spectral similarity scores).
edam_topics:
- http://edamontology.org/topic_3522
- http://edamontology.org/topic_0121
tools:
- name: SIMILE
  role: Python library whose sml.similarity_matrix function builds the mass-difference
    transition matrix and its Laplacian pseudo-inverse similarity matrix S with spec_ids
  repo: https://mybinder.org/v2/gh/biorack/simile/HEAD
- name: numpy
  role: Python dependency used to represent the similarity and transition matrices
    numerically
- name: scipy
  role: Python dependency supporting the matrix computations behind the Laplacian
    embedding
- name: pandas
  role: Python dependency used for the tabular reporting built from S in later steps
    (matching_ions_report)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:3ca3190bfefca2809b4dfa6bd57f6888d0761085e69f766258ffbfd2b3e0a418
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - SIMILE
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3522
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# laplacian-embedding-similarity-matrix-construction

## Summary

Constructs a pair-specific fragmentation similarity matrix for tandem mass spectra using SIMILE, in which row-normalized mass-difference (and optional precursor neutral-loss) frequencies define a transition matrix that is converted to the pseudo-inverse of its normalized Laplacian, yielding an 'average commute time' distance between fragment ions. This graph-theoretic embedding is what makes subsequent spectral alignment robust to multiple structural modifications and enables finite, context-sensitive fragment-ion similarities.

## When to use

Use when input is a collection of MS/MS spectra (fragment m/z lists and precursor m/z values) and you need a pair-specific fragment-ion similarity matrix that tolerates multiple structural differences before spectral alignment or significance testing.

## When NOT to use

- Spectra whose adducts are other than protonation/deprotonation — the paper states SIMILE 'is currently limited to analysis of protonation/deprotonation, which is certainly a limitation'.
- Compound classes the authors flag as potentially better served by cosine-based scoring, where a modified-cosine similarity is the more appropriate primary measure.
- Cases where you only need plain peak-matching similarity without the fragment-similarity graph embedding (i.e., cosine-based spectral similarity scores).

## Inputs

- mzs: list of fragment m/z arrays, one per tandem mass spectrum
- pmzs: precursor m/z values for each spectrum (optional but recommended, enables precursor-based neutral-loss difference counts)
- tolerance: m/z tolerance parameter for binning mass-difference counts (value not specified in the paper's code snippets; must be chosen by the practitioner)

## Outputs

- S: pair-specific fragmentation similarity matrix over fragment ions
- spec_ids: list of spectrum identifiers associated with S, needed by downstream sml.multiple_match, sml.sym_compare, and sml.z_test

## How to apply

Extract the fragment m/z lists (mzs) and precursor m/z values (pmzs) for each spectrum, then call sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance) to obtain the similarity matrix S and the spectrum identifier list spec_ids. Internally, mass-difference counts (optionally including precursor-based neutral-loss difference counts) are row-normalized into a transition matrix satisfying property (a), that fragment ions are similar if their mass difference is common; this transition matrix is then converted to the pseudo-inverse of its normalized Laplacian, satisfying property (b), that fragment ions are similar if their ancestor and descendant fragment ions are similar. The rationale is that a global substitution matrix over all fragment ions would be infinite, but restricting the graph to a single pair of spectra yields finite, context-sensitive similarities corresponding to an average commute time distance — a random walk on the transition matrix must wander between two ions and back, and ions sharing ancestors/descendants have shorter commute times. The tolerance parameter controls the m/z binning of difference counts and must be set explicitly (its value is not fixed in the article's snippets); decide it based on instrument mass accuracy. The output S feeds directly into sml.multiple_match and sml.sym_compare for matching and significance testing, so keep S and spec_ids together as a pair.

## Related tools

- **SIMILE** (Python library whose sml.similarity_matrix function builds the mass-difference transition matrix and its Laplacian pseudo-inverse similarity matrix S with spec_ids) — https://mybinder.org/v2/gh/biorack/simile/HEAD
- **numpy** (Python dependency used to represent the similarity and transition matrices numerically)
- **scipy** (Python dependency supporting the matrix computations behind the Laplacian embedding)
- **pandas** (Python dependency used for the tabular reporting built from S in later steps (matching_ions_report))

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)
```

## Evaluation signals

- sml.similarity_matrix returns exactly two objects, S and spec_ids, and spec_ids has one identifier per input spectrum — confirm both before proceeding.
- The matrix construction should reflect row-normalized mass-difference frequencies as transition probabilities; verify no global (cross-spectrum) substitution matrix is used — similarity must be pair-specific.
- Downstream invariants: S must be consumable by sml.multiple_match(S, spec_ids) without reshaping, and by sml.z_test with return_dist=True, log_size=5 producing spec_scores, pval, and null_dist.
- On the 20-flavonoid/20-indole GNPS reference-library benchmark, the full pipeline over S yields SIMILE vs cosine calls for all 780 pairs (40 choose 2), with SIMILE identifying 97 flavonoid pairs missed by GNPS cosine.
- Environment check: the conda environment from environment-base.yml pins python3 3.7 with numpy, scipy, and pandas per the README.

## Limitations

- The tolerance parameter value is not specified in the article's code snippets and must be supplied by the user.
- SIMILE is currently limited to protonation/deprotonation analysis; comprehensively addressing additional adducts is left to future research.
- Since this is the first application of scoring metabolite-fragmentation spectra using a similarity matrix, close attention must be applied; the authors recommend complementing it with cosine-based scoring.
- Intermediate similarity scores without statistical significance are uninterpretable for prioritizing metabolites — S alone must be paired with the z_test significance step.
- Likely, there will be classes of molecules that are more appropriate for cosine-based scoring than SIMILE-based scoring.

## Evidence

- [readme] Property (a) is satisfied by constructing a transition matrix with row-normalized mass difference frequencies as transition probabilites.: "Property (a) is satisfied by constructing a transition matrix with row-normalized mass difference frequencies as transition probabilites."
- [readme] Property (b) is satisfied by converting the transition matrix of (a) into the pseudo-inverse of its (normalized) laplacian.: "Property (b) is satisfied by converting the transition matrix of (a) into the pseudo-inverse of its (normalized) laplacian."
- [readme] This corresponds to an "average commute time" distance between fragment ions with the following intuition: "This corresponds to an "average commute time" distance between fragment ions with the following intuition"
- [methods] S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance): "# Generate fragmentation similarity matrix
S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [intro] if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive: "if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive"
- [readme] Precursor-based neutral loss difference counts can be used in addition to the original MZ difference counts: "Precursor-based neutral loss difference counts can be used in addition to the original MZ difference counts"
