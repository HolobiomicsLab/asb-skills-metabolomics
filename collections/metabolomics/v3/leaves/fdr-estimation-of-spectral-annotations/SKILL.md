---
name: fdr-estimation-of-spectral-annotations
description: Use when filtering GC-MS deconvoluted spectral matches (with per-match
  cosine similarity and balance score annotations) and you need to choose joint thresholds
  that minimize the FDR of the top annotation.
when_to_use_negative:
- Input is raw GC-MS data that has not yet been auto-deconvolved by MSHub (no cosine/balance
  score annotations exist to filter on)
- The analysis goal is deconvolution itself (non-negative matrix factorization of
  fragmentation patterns) rather than post-deconvolution annotation filtering
- Matches lack the balance score annotation, since neither the joint threshold nor
  the conservative >80% filter can be evaluated
edam_operation: http://edamontology.org/operation_3211
tools:
- name: MSHub
  role: Produces the auto-deconvoluted spectral matches with cosine similarity and
    balance score annotations that the FDR filters operate on
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform hosting the GC-MS workflows for storing, processing, annotating,
    and comparing the spectral matches used in the FDR benchmark
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:bd2cafb56ceadf519df45d71e7d30485331a33d3124e6bf4e1879f062aeb4538
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3211
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fdr-estimation-of-spectral-annotations

## Summary

Estimates the false discovery rate of top-match spectral annotations from MSHub/GNPS GC-MS auto-deconvolution output by jointly applying cosine similarity and balance score thresholds (cosine >0.9 with balance >60%; conservative balance >80%), as benchmarked in the Figure 1m,n test dataset.

## When to use

Use when filtering GC-MS deconvoluted spectral matches (with per-match cosine similarity and balance score annotations) and you need to choose joint thresholds that minimize the FDR of the top annotation.

## When NOT to use

- Input is raw GC-MS data that has not yet been auto-deconvolved by MSHub (no cosine/balance score annotations exist to filter on)
- The analysis goal is deconvolution itself (non-negative matrix factorization of fragmentation patterns) rather than post-deconvolution annotation filtering
- Matches lack the balance score annotation, since neither the joint threshold nor the conservative >80% filter can be evaluated

## Inputs

- MSHub/GNPS deconvolution output with spectral matches annotated with cosine similarity and balance score (Figure 1m-o test dataset)

## Outputs

- Per-filter FDR values of the top match (fdr_filter_comparison.csv)
- Selected threshold regime (combined cosine >0.9 / balance >60% vs balance >80%) with confirmed FDR ordering

## How to apply

Load the MSHub deconvolution output for the test dataset, where each spectral match carries a cosine similarity and a balance score (the latter quantifying reproducibility of fragmentation patterns across all samples). Apply the joint filter of cosine >0.9 with balance score >60% and compute the FDR of the top match; then apply the balance-score-only filter of >80% and compute the FDR of the top match again. Compare the two regimes: the combined cosine >0.9 / balance >60% threshold achieves the lowest FDR of the top match, while the more conservative balance score >80% essentially guarantees the lowest observed FDR even for poor cosine (match) scores. Write per-filter FDR results to fdr_filter_comparison.csv so the threshold ordering can be audited.

## Related tools

- **MSHub** (Produces the auto-deconvoluted spectral matches with cosine similarity and balance score annotations that the FDR filters operate on) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Platform hosting the GC-MS workflows for storing, processing, annotating, and comparing the spectral matches used in the FDR benchmark) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
filtered = matches[(matches.cosine > 0.9) & (matches.balance_score > 0.60)]; filtered.to_csv('fdr_filter_comparison.csv')
```

## Evaluation signals

- Reproduced FDR ordering must match the reported relationship: lowest FDR of top match at the combined cosine >0.9 / balance >60% threshold
- Lowest observed FDR must be guaranteed under the balance score >80% filter, even for poor cosine scores
- fdr_filter_comparison.csv must contain per-filter FDR results for both the joint and balance-score-only regimes
- Balance score values must reflect reproducibility of fragmentation patterns across all samples in the dataset

## Limitations

- Threshold values are benchmarked on the specific Figure 1m,n test dataset; they may not transfer to other GC-MS datasets without re-evaluating FDR
- The conservative balance score >80% filter guarantees the lowest observed FDR but is not stated to outperform the combined cosine >0.9 / balance >60% threshold at the top match

## Evidence

- [readme] companion repository: [evidence span withheld — anchor status: not_found]
