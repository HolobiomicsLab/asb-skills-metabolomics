---
name: fragmentation-pattern-reproducibility-quantification
description: Use when processing GC-MS data deconvoluted with MSHub/GNPS and you need
  to score deconvolution confidence per compound and discard annotations whose fragmentation
  patterns are irreproducible across samples.
when_to_use_negative:
- The data are not GC-MS deconvolution output from MSHub/GNPS (e.g. LC-MS/MS feature
  tables or already-annotated MS2 molecular networking inputs)
- A single-sample dataset, since the balance score is defined as reproducibility of
  fragmentation patterns ACROSS samples and cannot be computed
- The feature table already carries a computed balance-score column and the task is
  only thresholding, not score computation
tools:
- name: MSHub
  role: Performs auto-deconvolution of GC-MS compound fragmentation patterns via unsupervised
    non-negative matrix factorization and provides the scoring from which the balance
    score is computed
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Community workflow platform in which the GC-MS auto-deconvolution, annotation,
    and molecular networking steps (including the balance-score filtering) are executed
  repo: github.com/bittremieux/GNPS_GC
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:60705201f834f8642fced318ad3a9cd356544b5dddf7a1fe1571c54591e0650a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fragmentation-pattern-reproducibility-quantification

## Summary

Computes and applies the 'balance score' introduced in the MSHub/GNPS GC-MS auto-deconvolution workflow, a metric that quantifies the reproducibility of each deconvoluted compound's fragmentation pattern across all samples, and uses it (jointly with the cosine match score) to filter annotation results.

## When to use

Use when processing GC-MS data deconvoluted with MSHub/GNPS and you need to score deconvolution confidence per compound and discard annotations whose fragmentation patterns are irreproducible across samples.

## When NOT to use

- The data are not GC-MS deconvolution output from MSHub/GNPS (e.g. LC-MS/MS feature tables or already-annotated MS2 molecular networking inputs)
- A single-sample dataset, since the balance score is defined as reproducibility of fragmentation patterns ACROSS samples and cannot be computed
- The feature table already carries a computed balance-score column and the task is only thresholding, not score computation

## Inputs

- MSHub/GNPS deconvolution feature/quantification table for GC-MS data (per-compound fragmentation pattern and abundance across all samples, CSV/TSV)
- User-specified balance-score threshold
- Per-compound cosine match scores from spectral annotation

## Outputs

- Feature table annotated with a per-compound balance score column
- filtered_annotations.csv containing only compounds meeting the balance-score threshold

## How to apply

Load the MSHub deconvolution feature/quantification table (per-compound fragmentation and abundance across all samples) with pandas. For each deconvoluted compound, compute the balance score as implemented in the MSHub/GNPS scoring, where the score quantifies the reproducibility of the compound's fragmentation pattern across samples — deconvoluted patterns containing missing or chimeric peaks produce low scores, while clean patterns score high. Annotate every compound row with its computed balance score. Apply the user-specified balance-score threshold jointly with the cosine match score to retain only high-confidence deconvolutions, discarding irreproducible deconvolution artifacts. Write the retained, score-annotated compounds to filtered_annotations.csv.

## Related tools

- **MSHub** (Performs auto-deconvolution of GC-MS compound fragmentation patterns via unsupervised non-negative matrix factorization and provides the scoring from which the balance score is computed) — github.com/bittremieux/GNPS_GC
- **GNPS** (Community workflow platform in which the GC-MS auto-deconvolution, annotation, and molecular networking steps (including the balance-score filtering) are executed) — github.com/bittremieux/GNPS_GC

## Examples

```
import pandas as pd; df = pd.read_csv('mshub_feature_table.csv'); df[df['balance_score'] >= threshold].to_csv('filtered_annotations.csv', index=False)
```

## Evaluation signals

- filtered_annotations.csv exists and every retained compound row carries a balance-score field whose value meets the applied threshold
- Columns of filtered_annotations.csv match the input feature-table schema plus the balance-score column
- Every discarded row's balance score is below the applied threshold (removed compounds correspond to irreproducible deconvolution artifacts)
- Balance scores discriminate as expected: compounds with missing or chimeric peaks score low, high-confidence deconvolutions score high

## Limitations

- The balance score is threshold-dependent; the appropriate cutoff must be chosen per analysis and applied jointly with the cosine match score, not alone
- Low balance scores arise from missing or chimeric peaks in the deconvoluted pattern, so borderline compounds may reflect coelution artifacts rather than absence of the compound
- Scalability of the deconvolution required a Fast Fourier Transform-based strategy, so very large GC-MS datasets may hit computational limits in the companion implementation (GNPS_GC repo is archived)

## Evidence
