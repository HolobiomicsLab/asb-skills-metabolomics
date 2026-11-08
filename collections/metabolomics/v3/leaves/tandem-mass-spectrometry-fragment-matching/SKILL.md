---
name: tandem-mass-spectrometry-fragment-matching
description: Use when pairwise alignment of tandem MS/MS fragmentation spectra (m/z
  arrays with precursor m/z) must be scored with statistical significance (z-test
  p-values) for structurally related metabolites, e.g. in molecular networking.
when_to_use_negative:
- Input spectra are not protonation/deprotonation tandem MS/MS — SIMILE is currently
  limited to protonation/deprotonation analysis and does not handle additional adducts.
- Goal is compound identification where cosine-based scoring alone is appropriate
  and expected — the article recommends complementing SIMILE with cosine-based scoring
  rather than replacing it.
- Only a single spectrum's internal fragment structure is needed without pairwise
  alignment across spectra — the z-test null is built from intra- and inter-spectral
  permutations across spectrum combinations.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIMILE
  role: Python library providing similarity_matrix, multiple_match, sym_compare, z_test,
    and matching_ions_report for interrelating fragmentation spectra with significance
    estimation
  repo: https://github.com/biorack/simile
- name: MS-DIAL
  role: Computational pipeline for molecular networking datasets in which SIMILE alignments
    were applied
- name: GNPS
  role: Reference molecular-networking framework whose cosine-based alignment SIMILE
    complements
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:f683ce3203016b86ef12d1c8289af21914fe777e908c9c0bd6509afbfde15758
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - GNPS
  - MS-DIAL
  - SIMILE
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# tandem-mass-spectrometry-fragment-matching

## Summary

Interrelate fragmentation spectra of small molecules using SIMILE's fragment-ion similarity matrix, max-weight matching, and symmetric comparison to discover statistically significant spectral alignments robust to multiple structural differences, beyond cosine-based scoring.

## When to use

Use when pairwise alignment of tandem MS/MS fragmentation spectra (m/z arrays with precursor m/z) must be scored with statistical significance (z-test p-values) for structurally related metabolites, e.g. in molecular networking.

## When NOT to use

- Input spectra are not protonation/deprotonation tandem MS/MS — SIMILE is currently limited to protonation/deprotonation analysis and does not handle additional adducts.
- Goal is compound identification where cosine-based scoring alone is appropriate and expected — the article recommends complementing SIMILE with cosine-based scoring rather than replacing it.
- Only a single spectrum's internal fragment structure is needed without pairwise alignment across spectra — the z-test null is built from intra- and inter-spectral permutations across spectrum combinations.

## Inputs

- Fragment-ion m/z arrays (mzs) from tandem MS/MS spectra
- Precursor m/z values (pmzs) for each spectrum
- m/z matching tolerance (tolerance parameter, value not specified in article)
- SIMILE artifacts: similarity matrix S, spec_ids, max-weight matching M, symmetric comparison matrix C

## Outputs

- spec_scores: z-scores of max-weight matchings for all pairs of spectra
- pval: p-values of max-weight matchings against the permutation null distribution
- null_dist: empirical null distribution of matching scores (10^5 permutations, log_size=5)
- M: max-weight matching matrix over fragment-ion similarity
- C: pro/con symmetric-comparison matrix (+1 symmetric, -1 asymmetric)
- df from matching_ions_report: mass deltas, scores, and metadata per comparison

## How to apply

Build a pair-specific fragment-ion similarity matrix S with sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance), which constructs a row-normalized mass-difference transition matrix and its normalized-Laplacian pseudo-inverse (average-commute-time distance) to capture common mass deltas and ancestor/descendant relationships between fragment ions. Compute the maximum-weight matching M with sml.multiple_match(S, spec_ids), then generate the pro/con comparison matrix C with sml.sym_compare(M, spec_ids), where symmetric matches are 1 and asymmetric matches are -1. Call sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5) to permute intra- and inter-spectral fragment similarity scores across all combinations of spectra, producing an empirical null distribution (10^5 samples) against which z-scores and p-values of the max-weight matchings are computed. Finally summarize mass deltas and scores with sml.matching_ions_report(S, M, C, mzs, pmzs). SIMILE should be complemented with cosine-based scoring (e.g. modified cosine) rather than used alone, since some molecule classes may score better by cosine.

## Related tools

- **SIMILE** (Python library providing similarity_matrix, multiple_match, sym_compare, z_test, and matching_ions_report for interrelating fragmentation spectra with significance estimation) — https://github.com/biorack/simile
- **MS-DIAL** (Computational pipeline for molecular networking datasets in which SIMILE alignments were applied)
- **GNPS** (Reference molecular-networking framework whose cosine-based alignment SIMILE complements)

## Examples

```
spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)
```

## Evaluation signals

- sml.z_test returns spec_scores, pval, and null_dist; save all three (e.g. .npz) and check that null_dist contains 10^5 samples (log_size=5).
- C from sml.sym_compare must contain only +1 (symmetric/pro) and -1 (asymmetric/con) entries for matched fragment pairs.
- The matching_ions_report dataframe should contain mass deltas and scores with metadata; verify the predominant m/z delta equals expected structural shifts (e.g. 30.983 corresponding to [+O2 −H]).
- P-values should be bounded [0,1] and computed from the empirical null distribution, not from a parametric assumption.
- Sanity-check that SIMILE recovers alignments not found by cosine-based similarity scores, per the article's comparative finding.

## Limitations

- SIMILE is currently limited to protonation/deprotonation; comprehensive adduct handling is left to future research.
- Parameters used to compare alignment algorithms were not optimized, and optimization entails trade-offs (e.g. improving specificity vs. number of hits).
- Significant SIMILE alignment does not necessitate compound structure similarity, and some molecule classes may be more appropriate for cosine-based scoring.
- Intermediate similarity scores without statistical significance are uninterpretable for prioritizing metabolites; hence SIMILE should be complemented with cosine-based scoring.

## Evidence

- [methods] Generate fragmentation similarity matrix (S, spec_ids) via sml.similarity_matrix: "S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance)"
- [methods] Calculate significance of max weight matching via sml.z_test with null_dist return: "spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [abstract] SIMILE provides statistical significance estimates for spectral alignments: [evidence span withheld — anchor status: not_found]
- [readme] Null distribution generated by permuting fragment similarity scores: "a null distibution generated by permuting intra and inter spectral fragment similarity scores to yield p-values"
- [discussion] SIMILE yields different and typically more associations than cosine-based scoring algorithms: "our alignment method, SIMILE, yields different and typically more associations than cosine-based scoring algorithms"
- [discussion] SIMILE is currently limited to protonation/deprotonation analysis: "SIMILE is currently limited to analysis of protonation/deprotonation, which is certainly a limitation."
