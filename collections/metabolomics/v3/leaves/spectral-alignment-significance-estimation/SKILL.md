---
name: spectral-alignment-significance-estimation
description: Use when fragmentation-spectra alignments (e.g. from a max-weight matching
  over a SIMILE similarity matrix S and spec_ids) must be ranked by p-value or judged
  significant, rather than treated as uninterpretable intermediate similarity scores.
when_to_use_negative:
- The analysis goal is compound identification scoring rather than alignment significance
  — the paper recommends complementing SIMILE with well-established cosine-based (e.g.
  modified cosine) scoring in that case
- Spectra were acquired as adducts other than protonation/deprotonation, since SIMILE
  is currently limited to analysis of protonation/deprotonation
- The input is already a significance result (e.g. a prior z_test output); re-running
  z_test on derived scores would double-count the null
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIMILE
  role: Provides the sml.z_test function that permutes fragment-ion similarity scores
    to build the null distribution and returns spec_scores, pval, and null_dist
  repo: https://mybinder.org/v2/gh/biorack/simile/HEAD
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:77a23e5676d284a78ac76968d5ec40bd849e7bc6f296c6f3cdea503ad7f38552
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SIMILE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# spectral-alignment-significance-estimation

## Summary

Compute statistical significance (z-scores and p-values) for pairwise tandem mass-spectrum alignments using the SIMILE sml.z_test routine, which builds an empirical null distribution by permuting intra- and inter-spectral fragment-ion similarity scores from the max-weight matching.

## When to use

Use when fragmentation-spectra alignments (e.g. from a max-weight matching over a SIMILE similarity matrix S and spec_ids) must be ranked by p-value or judged significant, rather than treated as uninterpretable intermediate similarity scores.

## When NOT to use

- The analysis goal is compound identification scoring rather than alignment significance — the paper recommends complementing SIMILE with well-established cosine-based (e.g. modified cosine) scoring in that case
- Spectra were acquired as adducts other than protonation/deprotonation, since SIMILE is currently limited to analysis of protonation/deprotonation
- The input is already a significance result (e.g. a prior z_test output); re-running z_test on derived scores would double-count the null

## Inputs

- Fragment-ion similarity matrix S (from sml.similarity_matrix)
- spec_ids array mapping matrix entries to spectra
- Max-weight matching matrix M (from sml.multiple_match)
- Pro/con symmetric-comparison matrix C (from sml.sym_compare; pro=+1, con=-1)

## Outputs

- spec_scores: alignment significance scores for all spectrum combinations
- pval: p-values for max-weight matchings against the null distribution
- null_dist: empirical null distribution of matching scores (10^5 samples with log_size=5)
- Serialized results file (e.g. z_test_results.npz) containing spec_scores, pval, and null_dist

## How to apply

First generate the upstream SIMILE artifacts: the fragment-ion similarity matrix S and spec_ids via sml.similarity_matrix, the max-weight matching M via sml.multiple_match, and the symmetric-comparison matrix C via sml.sym_compare (symmetric matches scored +1 'pro', asymmetric matches -1 'con'). Then call sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5), which permutes fragment-ion similarity scores within and between spectra for all spectrum combinations — per the paper, the null distribution is generated 'by permuting the pair-specific similarity matrix interrelating two fragmentation spectra randomly many times, with restrictions imposed according to the null hypothesis' — using log_size=5 to yield 10^5 null samples. The function returns spec_scores, pval, and null_dist; save all three (e.g. to z_test_results.npz) so significance estimates and the empirical null are reproducible downstream. Rank candidate alignments by their p-values, and interpret significant alignment as evidence of moderate-to-high structural similarity — but not a guarantee of it.

## Related tools

- **SIMILE** (Provides the sml.z_test function that permutes fragment-ion similarity scores to build the null distribution and returns spec_scores, pval, and null_dist) — https://mybinder.org/v2/gh/biorack/simile/HEAD

## Examples

```
spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)
```

## Evaluation signals

- The z_test_results.npz output contains exactly the three named fields spec_scores, pval, and null_dist
- pval values are valid p-values (bounded in [0, 1]) and spec_scores/pval arrays align with all spectrum combinations implied by spec_ids
- null_dist has 10^5 samples, consistent with the log_size=5 setting
- Ranking candidate alignments by p-value yields a different (typically larger) set of associations than cosine-based similarity scoring would
- The z_test call signature matches the documented pattern 'spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)'

## Limitations

- Significant SIMILE alignment of fragmentation spectra does not necessitate compound structure similarity
- Without statistical significance, intermediate similarity scores are to some degree uninterpretable, so this step is needed before prioritizing metabolites
- SIMILE is a first-of-its-kind application of scoring metabolite-fragmentation spectra using a similarity matrix; close attention must be applied
- SIMILE is currently limited to analysis of protonation/deprotonation, which is certainly a limitation
- Current research in multiple comparison is exploring using the asymmetry of the SIMILE max weight matrix as an alternative way to generate a null distribution

## Evidence

- [supplementary] The alignment-score null distribution used to calculate significance is generated by permuting the pair-specific similarity matrix interrelating two fragmentation spectra randomly many times, with restrictions imposed according to the null hypothesis.: [evidence span withheld — anchor status: not_found]
- [methods] Calculate significance of max weight matching via sml.z_test with null_dist return: "spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [readme] Null distribution construction via permutation of intra/inter spectral similarity scores: "we can ask how likely it is that the fragment ions matched up between fragmentation spectra by SIMILE are siblings"
- [abstract] SIMILE provides statistical significance estimates for spectral alignments: [evidence span withheld — anchor status: not_found]
- [readme] Alternative null-generation via max-weight matrix asymmetry: "Current research in multiple comparison is exploring using the asymmetry of the SIMILE max weight matrix as an alternative way to generate a null distribution"
