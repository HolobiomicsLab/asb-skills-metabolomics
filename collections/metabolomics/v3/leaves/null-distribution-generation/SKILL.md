---
name: null-distribution-generation
description: Use when pairwise fragmentation-spectrum alignment scores need statistical
  significance — e.g. when ranking SIMILE max-weight matchings or interpreting intermediate
  similarity scores between MS/MS spectra pairs.
when_to_use_negative:
- When only cosine-based similarity is needed and no significance estimate is required
  — SIMILE complements, rather than replaces, cosine-based scoring such as modified
  cosine.
- When analyzing adducts other than protonation/deprotonation — SIMILE is currently
  limited to analysis of protonation/deprotonation.
- When fragment-ion similarities would be invalid to exchange under the permutation
  null (e.g. spectra generated under different adduct assumptions).
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SIMILE
  role: sml.z_test computes the null distribution, z-scores, and p-values for max-weight
    matchings of fragment-ion similarity matrices
  repo: https://github.com/biorack/simile
- name: MS-DIAL
  role: upstream processing pipeline that produced the fragmentation spectra aligned
    by SIMILE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:d4fcdc7d4b9876e6df7f86d7977749454e4234cd0872bfb5ad2344b3988dc9d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - MS-DIAL
  - SIMILE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# null-distribution-generation

## Summary

Generates an empirical null distribution of SIMILE spectral-alignment scores by permuting intra- and inter-spectral fragment-ion similarity scores under the exchangeability null, so that max-weight matching scores can be converted into z-scores and p-values for pairwise tandem-MS spectra comparisons.

## When to use

Use when pairwise fragmentation-spectrum alignment scores need statistical significance — e.g. when ranking SIMILE max-weight matchings or interpreting intermediate similarity scores between MS/MS spectra pairs.

## When NOT to use

- When only cosine-based similarity is needed and no significance estimate is required — SIMILE complements, rather than replaces, cosine-based scoring such as modified cosine.
- When analyzing adducts other than protonation/deprotonation — SIMILE is currently limited to analysis of protonation/deprotonation.
- When fragment-ion similarities would be invalid to exchange under the permutation null (e.g. spectra generated under different adduct assumptions).

## Inputs

- Fragment-ion similarity matrix S from sml.similarity_matrix
- spec_ids array identifying spectra in S
- Max-weight matching matrix M from sml.multiple_match
- Pro/con comparison matrix C from sml.sym_compare (symmetric=1, asymmetric=-1)

## Outputs

- spec_scores: max-weight matching scores for all spectrum combinations
- pval: p-values for each max-weight matching against the null distribution
- null_dist: empirical null distribution (10^5 samples at log_size=5)

## How to apply

Run sml.z_test on the three SIMILE upstream artifacts: the fragment-ion similarity matrix S and spec_ids from sml.similarity_matrix, the max-weight matching matrix M from sml.multiple_match, and the pro/con comparison matrix C from sml.sym_compare (symmetric matches = 1, asymmetric = -1). Pass spec_ids so the exchangeability null can permute fragment-ion similarity scores within and between spectra for all combinations of spectra, comparing pro (+1) and con (-1) matches. Set return_dist=True and log_size=5 to sample 10^5 null permutations and have the null distribution returned alongside the scores. The function returns spec_scores, pval, and null_dist, which should be serialized together (e.g. z_test_results.npz). The null is built on the intuition that matched fragment ions are likely 'siblings' — more similar to each other than to their parents or children — so the permutation leverages intraspectral comparisons to add confidence to interspectral comparisons.

## Related tools

- **SIMILE** (sml.z_test computes the null distribution, z-scores, and p-values for max-weight matchings of fragment-ion similarity matrices) — https://github.com/biorack/simile
- **MS-DIAL** (upstream processing pipeline that produced the fragmentation spectra aligned by SIMILE)

## Examples

```
import simile as sml; spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)
```

## Evaluation signals

- Return values match the call signature: spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)
- null_dist array has 10^5 entries consistent with log_size=5
- pval values are within [0,1] and correspond monotonically to z-score extremity
- Serialized npz contains exactly the three named fields spec_scores, pval, null_dist
- Significant alignment scores (low pval) track moderate-to-high compound structural similarity on validation pairs

## Limitations

- Parameters used to compare alignment algorithms were not optimized; further optimization could improve performance but entails trade-offs (e.g. specificity vs. number of hits).
- SIMILE is limited to analysis of protonation/deprotonation adducts.
- Since this is the first application of scoring metabolite-fragmentation spectra using a similarity matrix, results should be complemented with well-established cosine-based scoring; some molecule classes may be better suited to cosine-based scoring.
- Without statistical significance, intermediate similarity scores are to some degree uninterpretable — significance estimation is the motivation for this null-distribution step.
- Current research explores using asymmetry of the SIMILE max weight matrix as an alternative way to generate a null distribution.

## Evidence

- [readme] Permuting intra and inter spectral fragment similarity scores to yield p-values: "permuting intra and inter spectral fragment similarity scores to yield p-values"
- [methods] sml.z_test call signature with return_dist and log_size parameters: "spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [readme] Null distribution leverages intraspectral comparisons for interspectral confidence: "a null distibution generated by permuting intra and inter spectral fragment similarity scores to yield p-values"
- [discussion] Intermediate similarity scores without significance are uninterpretable: "without statistical significance, intermediate similarity scores are to some degree uninterpretable"
