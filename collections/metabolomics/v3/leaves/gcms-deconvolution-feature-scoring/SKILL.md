---
name: gcms-deconvolution-feature-scoring
description: Use when a MSHub/GNPS GC-MS deconvolution feature/quantification table
  must be scored for deconvolution confidence and filtered to discard chimeric or
  irreproducible compound fragmentation patterns before annotation.
when_to_use_negative:
- Input is already a filtered annotation table with no per-sample fragmentation-pattern
  information from which reproducibility can be quantified.
- Data are LC-MS/MS feature-based molecular networking outputs (e.g. MZmine/GNPS feature
  tables for pyMolNetEnhancer-style motif mapping) rather than GC-MS deconvolution
  results.
- The goal is molecular networking or chemical-class mapping (GNPS molecular networking,
  MS2LDA motif annotation) rather than deconvolution-confidence scoring.
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: MSHub
  role: Machine learning approach that auto-deconvolves GC-MS data and yields the
    feature/quantification table whose fragmentation patterns are scored for reproducibility
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Workflow platform within which GC-MS deconvolution, annotation, and molecular
    networking are executed and from which result tables are retrieved for filtering
  repo: https://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:5ffd2ace650f7cf790e90cbd64fed166b53e890db22a22c6641e3cead33ea312
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# gcms-deconvolution-feature-scoring

## Summary

Scores deconvoluted GC-MS compound features from MSHub/GNPS using the 'balance score', a metric quantifying the reproducibility of fragmentation patterns across samples, and uses it jointly with the cosine match score to retain only high-confidence deconvolutions when filtering annotations.

## When to use

Use when a MSHub/GNPS GC-MS deconvolution feature/quantification table must be scored for deconvolution confidence and filtered to discard chimeric or irreproducible compound fragmentation patterns before annotation.

## When NOT to use

- Input is already a filtered annotation table with no per-sample fragmentation-pattern information from which reproducibility can be quantified.
- Data are LC-MS/MS feature-based molecular networking outputs (e.g. MZmine/GNPS feature tables for pyMolNetEnhancer-style motif mapping) rather than GC-MS deconvolution results.
- The goal is molecular networking or chemical-class mapping (GNPS molecular networking, MS2LDA motif annotation) rather than deconvolution-confidence scoring.

## Inputs

- MSHub/GNPS GC-MS deconvolution feature/quantification table (per-compound fragmentation and abundance across samples)
- User-specified balance-score threshold
- Cosine similarity match scores per compound annotation

## Outputs

- filtered_annotations.csv containing retained compounds annotated with their balance scores
- Per-compound balance-score values quantifying fragmentation-pattern reproducibility across samples

## How to apply

Load the MSHub deconvolution feature/quantification table (per-compound fragmentation and abundance across samples) with pandas. MSHub/GNPS performs auto-deconvolution of compound fragmentation patterns via unsupervised non-negative matrix factorization (with a Fast Fourier Transform-based strategy to overcome scalability limitations), and a 'balance score' quantifies the reproducibility of fragmentation patterns across all samples — missing or chimeric peaks produce low scores, so the balance score acts as a deconvolution-confidence metric. Annotate each deconvoluted compound row with its computed balance score, then filter the annotation set jointly on the balance score threshold and the cosine similarity match score, retaining only compounds passing both. Write the retained, score-annotated compounds to filtered_annotations.csv and validate that every retained row carries a balance-score value meeting the threshold and that columns match the input schema plus the balance-score column.

## Related tools

- **MSHub** (Machine learning approach that auto-deconvolves GC-MS data and yields the feature/quantification table whose fragmentation patterns are scored for reproducibility) — github.com/bittremieux/GNPS_GC
- **GNPS** (Workflow platform within which GC-MS deconvolution, annotation, and molecular networking are executed and from which result tables are retrieved for filtering) — https://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
import pandas as pd; df = pd.read_csv('mshub_quant_table.csv'); df['balance_score'] = df.groupby('compound')['fragmentation'].apply(compute_balance_score); df[df.balance_score >= threshold].to_csv('filtered_annotations.csv')
```

## Evaluation signals

- filtered_annotations.csv exists and is non-empty after filtering.
- Every retained compound row carries a balance-score field whose value meets the applied threshold.
- Output table columns match the input feature-table schema plus the added balance-score column.
- Compounds with missing or chimeric peaks (low balance scores) are absent from the filtered output.
- Retained annotations also satisfy the cosine similarity match-score criterion applied jointly with the balance-score threshold.

## Limitations

- Missing or chimeric peaks in the deconvoluted spectral pattern produce low balance scores, so genuinely present but poorly deconvoluted compounds may be discarded by the threshold.
- Deconvolution scalability required a Fast Fourier Transform-based strategy; very large datasets may still be constrained by this step.
- No changelog was found for the companion repository, so implementation details of the scoring must be traced to the manuscript rather than repo version history.

## Evidence
