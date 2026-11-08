---
name: protein-sequence-alignment-analogy-application
description: Use when aligning pairs of fragmentation (MS/MS) spectra that contain
  multiple delocalized structural differences, where cosine-based similarity fails
  and a biologically-meaningful alignment with p-value estimation is required.
when_to_use_negative:
- The task is compound identification where cosine-based scoring is already appropriate
  — the article recommends complementing SIMILE with cosine-based scoring, and notes
  some molecule classes may be more appropriate for cosine-based scoring.
- The alignment does not require statistical significance — intermediate similarity
  scores without p-values are to some degree uninterpretable for prioritizing metabolites.
- The analysis involves adducts other than protonation/deprotonation — SIMILE is currently
  limited to analysis of protonation/deprotonation.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0080
tools:
- name: SIMILE
  role: Python library implementing the pair-specific fragment ion similarity matrix
    and maximum weight matching that realize the protein-alignment analogy for MS/MS
    spectra
  repo: https://github.com/biorack/simile
- name: BLAST
  role: cited protein-sequence alignment algorithm whose statistical-significance
    property SIMILE transfers to tandem mass spectra
- name: GNPS
  role: molecular networking tool implementing an alignment approach that SIMILE's
    analogy-based method improves upon
- name: NIST Hybrid Search
  role: existing alignment approach contrasted with SIMILE's pair-specific similarity
    framework
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-30118-9
    title: SIMILE enables alignment of tandem mass spectra with statistical significance
  source_content_hash: sha256:91069f44a1eeea8237af00ad5368b093a66c629bee09c2c62d75a959edd0b361
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biorack/simile
  tools:
  - BLAST
  - GNPS
  - NIST Hybrid Search
  - SIMILE
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# protein-sequence-alignment-analogy-application

## Summary

Applies the protein-sequence alignment framework (Needleman–Wunsch, Smith–Waterman, BLAST) by analogy to tandem mass spectrometry, substituting a pair-specific fragment ion similarity matrix for a global substitution matrix so that MS/MS spectra can be aligned with gaps, substitutions, and statistical significance.

## When to use

Use when aligning pairs of fragmentation (MS/MS) spectra that contain multiple delocalized structural differences, where cosine-based similarity fails and a biologically-meaningful alignment with p-value estimation is required.

## When NOT to use

- The task is compound identification where cosine-based scoring is already appropriate — the article recommends complementing SIMILE with cosine-based scoring, and notes some molecule classes may be more appropriate for cosine-based scoring.
- The alignment does not require statistical significance — intermediate similarity scores without p-values are to some degree uninterpretable for prioritizing metabolites.
- The analysis involves adducts other than protonation/deprotonation — SIMILE is currently limited to analysis of protonation/deprotonation.

## Inputs

- Fragment ion m/z arrays (mzs) from tandem mass spectra
- Precursor m/z values (pmzs) for the spectra being compared
- m/z tolerance for difference-count binning (tolerance parameter)

## Outputs

- Fragment ion similarity matrix S and spectral identifier array spec_ids (from sml.similarity_matrix)
- Max weight matching matrix M with same indexing as S (from sml.multiple_match)
- Pro/con comparison matrix C marking symmetric (+1) vs asymmetric (−1) matches (from sml.sym_compare)
- Spectral alignment scores and p-values with null_dist (from sml.z_test)
- Matching ions report dataframe with mass deltas and scores (from sml.matching_ions_report)

## How to apply

Build a pair-specific fragment ion similarity matrix rather than a global one: because the number of possible fragment m/z values is effectively infinite, a global substitution matrix for fragment ions would be infeasible, so SIMILE parameterizes spectral graph-theoretic similarity on the all-by-all m/z difference counts within a single pair of spectra, drawing on precursor-based neutral loss difference counts as well as MZ difference counts (V2). Analogy to protein alignment: substitutions are mass-delta-shifted fragment ions and gaps are unmatched ions, replacing the monotonic alignment originally used by SIMILE with maximum weight matching (sml.multiple_match) for improved performance. The same conceptual machinery used by Needleman–Wunsch, Smith–Waterman, and BLAST — yielding alignments with statistical significance — is transferred by computing symmetric matches as pro (+1) and asymmetric matches as con (−1) via sml.sym_compare, then estimating significance with sml.z_test against a null distribution built from permuted intra- and inter-spectral fragment similarity scores (log_size=5). Rationale: significant alignment scores correspond to compound structural similarity, so this framing allows ranking candidate alignments by p-value where intermediate cosine scores would be uninterpretable.

## Related tools

- **SIMILE** (Python library implementing the pair-specific fragment ion similarity matrix and maximum weight matching that realize the protein-alignment analogy for MS/MS spectra) — https://github.com/biorack/simile
- **BLAST** (cited protein-sequence alignment algorithm whose statistical-significance property SIMILE transfers to tandem mass spectra)
- **GNPS** (molecular networking tool implementing an alignment approach that SIMILE's analogy-based method improves upon)
- **NIST Hybrid Search** (existing alignment approach contrasted with SIMILE's pair-specific similarity framework)

## Examples

```
S, spec_ids = sml.similarity_matrix(mzs, pmzs=pmzs, tolerance=tolerance); M = sml.multiple_match(S, spec_ids); C = sml.sym_compare(M, spec_ids); spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)
```

## Evaluation signals

- Matching matrix M must have the same dimensions as the similarity matrix S and its matched ion pairs must be consistent with spec_ids.
- Symmetric matches in C should be 1 (pro) and asymmetric matches −1 (con), per the sml.sym_compare contract in the README.
- Significant alignment scores should correspond to compound structural similarity; alignments between compounds with multiple delocalized structural differences should be detected where cosine-based similarity fails.
- The z_test must return a null_dist generated by permuting intra- and inter-spectral fragment similarity scores, with log_size=5 as in the documented example.
- The predominant mass deltas in the report (e.g. 30.983 for [+O2 −H]) should be chemically interpretable relative to known modifications.

## Limitations

- Parameters used to compare alignment algorithms were not optimized; further optimization is left as future work and such optimizations almost certainly result in trade-offs in performance (e.g. improving specificity vs. number of hits).
- SIMILE is currently limited to analysis of protonation/deprotonation; comprehensively addressing additional adducts is left to future research.
- Since this is the first application of scoring metabolite-fragmentation spectra using a similarity matrix, close attention must be applied; the authors recommend complementing SIMILE with well-established cosine-based scoring.
- Without statistical significance, intermediate similarity scores are to some degree uninterpretable for prioritizing metabolites.
- There are improvements to the SIMILE algorithm that can likely boost its reliability and interpretability.

## Evidence

- [intro] protein-alignment analogy framing with pair-specific similarity matrix: "Here, we introduce Significant Interrelation of MS/MS Ions via Laplacian Embedding (SIMILE), an approach that leverages methods used for protein-sequence alignment"
- [intro] pair-specific similarity matrix rationale: "if restricted to a single pair of fragmentation spectra, a spectral graph-theoretic framework parameterized by their all-by-all m/z difference counts can generate finite, context sensitive, and"
- [readme] maximum weight matching replaces monotonic alignment: "Maximum weight matching is used instead of original monotonic alignment method with improved performance"
- [readme] concrete invocation showing the analogy applied end-to-end: "spec_scores, pval, null_dist = sml.z_test(S, M, C, spec_ids, return_dist=True, log_size=5)"
- [discussion] alignment yields more associations than cosine-based scoring: "our alignment method, SIMILE, yields different and typically more associations than cosine-based scoring algorithms"
