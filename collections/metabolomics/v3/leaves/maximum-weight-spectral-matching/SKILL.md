---
name: maximum-weight-spectral-matching
description: Use when comparing pairs or sets of MS/MS tandem spectra — e.g. from
  MassBank or molecular networking datasets — where compounds have multiple delocalized
  structural differences and cosine-based scores fail, and significance estimation
  (p-values) is needed.
when_to_use_negative:
- When the goal is straightforward compound identification via cosine or modified-cosine
  similarity and no significance testing is needed — the article recommends complementing
  SIMILE with cosine-based scoring rather than replacing it
- When fragment spectra come from adduct types outside protonation/deprotonation (e.g.
  [M+Na]+) — the article explicitly notes 'SIMILE is currently limited to analysis
  of protonation/deprotonation, which is certainly a limitation.'
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: SIMILE
  role: Python library providing similarity_matrix, multiple_match, sym_compare, z_test,
    and matching_ions_report for max-weight spectral matching with significance estimation
  repo: https://github.com/biorack/simile
- name: MS-DIAL
  role: Computational pipeline through which SIMILE was applied to molecular networking
    datasets
- name: pandas
  role: DataFrame schema used by sml.matching_ions_report to summarize matched mass
    deltas and scores
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:4e099b3ec925de41e7cf34bd53aacaa46f51c79cd02b45ba23f3e6fc96258cea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - MS-DIAL
  - SIMILE
  - pandas
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# maximum-weight-spectral-matching

## Summary

SIMILE's maximum-weight matching step aligns fragment ions between tandem mass spectra using a pair-specific similarity matrix, then scores each alignment's significance via a permuted intra/inter-spectral null distribution (z-test), enabling statistically grounded spectral comparisons beyond cosine similarity.

## When to use

Use when comparing pairs or sets of MS/MS tandem spectra — e.g. from MassBank or molecular networking datasets — where compounds have multiple delocalized structural differences and cosine-based scores fail, and significance estimation (p-values) is needed.

## When NOT to use

- When the goal is straightforward compound identification via cosine or modified-cosine similarity and no significance testing is needed — the article recommends complementing SIMILE with cosine-based scoring rather than replacing it
- When fragment spectra come from adduct types outside protonation/deprotonation (e.g. [M+Na]+) — the article explicitly notes 'SIMILE is currently limited to analysis of protonation/deprotonation, which is certainly a limitation.'

## Inputs

- Peak m/z lists (mzs) for each tandem mass spectrum
- Precursor m/z values (pmzs) for each spectrum
- m/z tolerance parameter for similarity matrix construction
- Spectrum identifiers (spec_ids) linking matrix entries to spectra

## Outputs

- Fragment-ion similarity matrix S with spectrum identifiers
- Max-weight matching matrix M
- Symmetric pro/con comparison matrix C (+1 pro / −1 con)
- Per-spectrum SIMILE alignment scores (spec_scores)
- Per-pair p-values (pval) and permuted null distribution (null_dist)
- matching_ions_report DataFrame with mass deltas, scores, and metadata

## How to apply

Build the fragment-ion similarity matrix S with sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance), where mzs are peak lists and pmzs are precursor masses (the README notes V2 also supports precursor-based neutral loss difference counts). Solve for the max-weight matching M with sml.multiple_match(S, spec_ids), which replaced the original monotonic alignment with improved performance. Construct the pro/con symmetric comparison matrix C via sml.sym_compare(M, spec_ids), where symmetric matches are +1 (pro) and asymmetric matches are −1 (con). Estimate significance with sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5), which permutes intra- and inter-spectral fragment similarity scores to build a null distribution and yields per-spectrum scores, p-values, and the null distribution itself. Summarize matched mass deltas, scores, and metadata with sml.matching_ions_report(S, M, C, mzs, pmzs). Rank candidate alignments by p-value, and complement SIMILE with cosine-based scoring for compound identification since significant alignment does not strictly imply structural similarity.

## Related tools

- **SIMILE** (Python library providing similarity_matrix, multiple_match, sym_compare, z_test, and matching_ions_report for max-weight spectral matching with significance estimation) — https://github.com/biorack/simile
- **MS-DIAL** (Computational pipeline through which SIMILE was applied to molecular networking datasets)
- **pandas** (DataFrame schema used by sml.matching_ions_report to summarize matched mass deltas and scores)

## Examples

```
import simile as sml; S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance); M = sml.multiple_match(S, spec_ids); C = sml.sym_compare(M, spec_ids); spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)
```

## Evaluation signals

- z_test must return finite spec_scores and p-values bounded in [0,1]; null_dist must be populated when return_dist=True and log_size=5
- Every entry of the similarity matrix S should correspond to a valid spec_ids pairing; no duplicate spectrum identifiers in the spec_ids output
- C matrix values must be constrained to {+1, −1} (symmetric pro vs. asymmetric con matches)
- matching_ions_report output must be a pandas DataFrame with mass deltas and scores present for each matched ion pair; low-p-value pairs should report non-empty matched mass deltas
- Cross-check: pairs with high cosine similarity should also show significant SIMILE alignment (though significant SIMILE alignments may exceed cosine-detected pairs)

## Limitations

- SIMILE is currently limited to analysis of protonation/deprotonation adduct spectra; other adduct types are left to future work
- Parameters used to compare alignment algorithms were not optimized; further optimization may trade off specificity vs. number of hits
- Significant SIMILE alignment does not necessitate compound structure similarity; some molecule classes may be more appropriate for cosine-based scoring
- Without statistical significance, intermediate similarity scores are to some degree uninterpretable for prioritizing metabolites
- Python is pinned to 3.7 currently due to non-SIMILE bugs; conda environment-base.yml supplies minimum dependencies (numpy, scipy, pandas)

## Evidence

- [methods] max-weight matching method: [evidence span withheld — anchor status: not_found]
- [methods] z_test null distribution: "spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [readme] readme usage: "# Calculate significance of max weight matching between fragment ions
# for all combination of spectra
spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [readme] null distribution intuition: "yield a null distibution generated by permuting intra and inter spectral fragment similarity scores to yield p-values."
- [discussion] complement cosine: "We recommend complementing SIMILE with the use of cosine-based scoring for compound identification."
