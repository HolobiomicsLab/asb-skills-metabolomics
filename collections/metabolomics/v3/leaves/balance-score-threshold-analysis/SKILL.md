---
name: balance-score-threshold-analysis
description: Decision support for choosing spectral-match filter thresholds (cosine
  similarity and balance score) on MSHub/GNPS GC-MS deconvolution output when the
  goal is to minimize the FDR of top-match annotations.
when_to_use_negative:
- Input is raw GC-MS data that has not yet been deconvoluted by MSHub — balance scores
  do not exist before auto-deconvolution via non-negative matrix factorization.
- The analysis goal is LC-MS/MS molecular networking without GC-MS spectral deconvolution
  — balance-score filtering is defined for deconvoluted GC-MS fragmentation patterns.
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: MSHub
  role: Performs auto-deconvolution of GC-MS data via unsupervised non-negative matrix
    factorization and computes the balance score used as the filtering threshold.
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform hosting the GC-MS deconvolution and molecular networking workflows
    whose annotated output is filtered by balance score and cosine similarity.
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:add6b98fb53753456856cc258ee56c10e925ff6fda2ce0b464420bcf4e79c991
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# balance-score-threshold-analysis

## Summary

Applies the MSHub/GNPS 'balance score' — a deconvolution-confidence metric quantifying reproducibility of fragmentation patterns across samples — jointly with the cosine match score to filter GC-MS annotations and minimize the false discovery rate of top-match annotations.

## When to use

Decision support for choosing spectral-match filter thresholds (cosine similarity and balance score) on MSHub/GNPS GC-MS deconvolution output when the goal is to minimize the FDR of top-match annotations.

## When NOT to use

- Input is raw GC-MS data that has not yet been deconvoluted by MSHub — balance scores do not exist before auto-deconvolution via non-negative matrix factorization.
- The analysis goal is LC-MS/MS molecular networking without GC-MS spectral deconvolution — balance-score filtering is defined for deconvoluted GC-MS fragmentation patterns.

## Inputs

- MSHub deconvolution output (feature/quantification table with per-compound fragmentation patterns and abundance across samples)
- Spectral matches annotated with cosine similarity and balance score (Figure 1m-n test dataset)

## Outputs

- fdr_filter_comparison.csv (per-filter FDR results for the combined cosine >0.9 / balance >60% filter vs. the balance >80%-only filter)
- filtered_annotations.csv (retained compounds annotated with balance scores meeting the applied threshold)

## How to apply

Load the MSHub deconvolution feature/quantification table with per-compound fragmentation and abundance across samples, and compute the balance score quantifying the reproducibility of each deconvoluted compound's fragmentation pattern across all samples (missing or chimeric peaks produce low scores). Then compare two filtering regimes on the test dataset: (1) a joint filter of cosine >0.9 combined with balance score >60%, and (2) a conservative balance-score-only filter of >80%. Compute the FDR of the top match under each regime and confirm the reported ordering: the lowest FDR of the top match occurs at the combined cosine >0.9 / balance >60% threshold, while balance score >80% essentially ensures the lowest observed FDR even for poor cosine (match) scores. Write the per-filter FDR results to fdr_filter_comparison.csv and retain only compounds whose balance score meets the applied threshold in filtered_annotations.csv.

## Related tools

- **MSHub** (Performs auto-deconvolution of GC-MS data via unsupervised non-negative matrix factorization and computes the balance score used as the filtering threshold.) — github.com/bittremieux/GNPS_GC
- **GNPS** (Platform hosting the GC-MS deconvolution and molecular networking workflows whose annotated output is filtered by balance score and cosine similarity.) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
df = pd.read_csv('mshub_deconvolution_table.csv'); kept = df[(df.cosine > 0.9) & (df.balance_score > 0.6)]; kept.to_csv('filtered_annotations.csv', index=False)
```

## Evaluation signals

- Reproduced FDR ordering must match the reported relationship: lowest FDR of the top match at combined cosine >0.9 / balance >60%, with lowest observed FDR guaranteed at balance >80%; otherwise flag for expert_review.
- Every retained compound in filtered_annotations.csv carries a balance-score field meeting the applied threshold.
- filtered_annotations.csv columns match the input feature table schema plus the added balance-score column.
- fdr_filter_comparison.csv exists and contains FDR values for both the joint cosine/balance filter and the balance-score-only filter.
- Low balance scores correspond to deconvoluted compounds with missing or chimeric peaks in their fragmentation patterns.

## Limitations

- Threshold values (cosine >0.9, balance >60%, balance >80%) were benchmarked on the specific Figure 1m-n test dataset and may not transfer directly to other datasets.
- The balance score depends on reproducibility of fragmentation patterns across all samples; compounds present in few samples or with chimeric spectra may be spuriously down-scored.

## Evidence

- [abstract] balance score definition: "We introduce a “balance score” that quantifies the reproducibility of fragmentation patterns across all samples."
- [abstract] deconvolution method: "MSHub/GNPS performs auto-deconvolution of compound fragmentation patterns via unsupervised non-negative matrix factorization"
- [abstract] scalability strategy: "using a Fast Fourier Transform-based strategy to overcome scalability limitations"
- [readme] workflow hosting: [evidence span withheld — anchor status: not_found]
