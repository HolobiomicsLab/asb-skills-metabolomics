---
name: annotation-table-format-handling
description: Use when working with an MSHub deconvolution feature/quantification table
  of GC-MS compounds and you must attach a balance score to each compound row and
  filter the annotation set at a threshold before downstream use.
when_to_use_negative:
- The input is raw GC-MS vendor or mzML-type spectra that still require spectral deconvolution
  — use MSHub auto-deconvolution first; this skill operates on the already-deconvoluted
  compound table.
- The goal is molecular-network construction or visualization (e.g. Cytoscape/network
  mapping via GNPS or pyMolNetEnhancer) rather than balance-score-based filtering
  of the annotation table.
- The table already carries balance scores and has been threshold-filtered, so re-annotating
  would duplicate fields.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSHub
  role: Produces the auto-deconvoluted GC-MS compound table and the balance score
    that quantifies reproducibility of fragmentation patterns across samples
  repo: https://github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Platform whose workflows store, process, share, annotate, and perform molecular
    networking of the GC-MS data; supplies cosine-match annotations filtered jointly
    with the balance score
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:5dc1587a3f536c2021306066163b5b0ab66a1583a6865b7d431f6afca5c4d4ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bittremieux/GNPS_GC
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# annotation-table-format-handling

## Summary

Handling of GC-MS annotation tables produced by MSHub/GNPS deconvolution: annotating each deconvoluted compound with its balance score (reproducibility of its fragmentation pattern across samples) and filtering the table at a balance-score threshold to retain reliable annotations.

## When to use

Use when working with an MSHub deconvolution feature/quantification table of GC-MS compounds and you must attach a balance score to each compound row and filter the annotation set at a threshold before downstream use.

## When NOT to use

- The input is raw GC-MS vendor or mzML-type spectra that still require spectral deconvolution — use MSHub auto-deconvolution first; this skill operates on the already-deconvoluted compound table.
- The goal is molecular-network construction or visualization (e.g. Cytoscape/network mapping via GNPS or pyMolNetEnhancer) rather than balance-score-based filtering of the annotation table.
- The table already carries balance scores and has been threshold-filtered, so re-annotating would duplicate fields.

## Inputs

- MSHub/GNPS deconvolution feature/quantification table (per-compound fragmentation and abundance across samples, CSV)
- Applied balance-score threshold and cosine-similarity match threshold

## Outputs

- filtered_annotations.csv containing only compounds whose balance score meets the threshold, with input feature-table columns plus a balance-score field per compound

## How to apply

Load the MSHub deconvolution feature/quantification table (per-compound fragmentation and abundance across samples) with pandas. For each deconvoluted compound, use the balance score from the MSHub/GNPS scoring — introduced in the article as the metric that 'quantifies the reproducibility of fragmentation patterns across all samples'; missing or chimeric deconvolution peaks produce low scores. Annotate each compound row with this balance-score field, then filter the table to retain only compounds whose balance score meets the applied threshold, jointly with the cosine match score used for spectral matching, discarding irreproducible deconvolution artifacts. Write the retained, score-annotated compounds to filtered_annotations.csv, preserving the input feature-table columns plus the added balance-score column.

## Related tools

- **MSHub** (Produces the auto-deconvoluted GC-MS compound table and the balance score that quantifies reproducibility of fragmentation patterns across samples) — https://github.com/bittremieux/GNPS_GC
- **GNPS** (Platform whose workflows store, process, share, annotate, and perform molecular networking of the GC-MS data; supplies cosine-match annotations filtered jointly with the balance score) — http://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
import pandas as pd; df = pd.read_csv('deconvolution_table.csv'); df = df[df['balance_score'] >= threshold]; df.to_csv('filtered_annotations.csv', index=False)
```

## Evaluation signals

- filtered_annotations.csv exists and parses as a table
- Every retained compound row carries a balance-score field whose value meets the applied threshold
- Output columns equal the input feature-table schema plus the added balance-score column
- No retained compound has a low balance score indicative of missing or chimeric deconvolution peaks
- Rows discarded correspond to irreproducible deconvolution artifacts (fragmentation patterns not reproducible across samples)

## Limitations

- The article specifies the balance score concept but no fixed numeric cutoff is given in the provided context; the threshold must be chosen by the practitioner.
- Low balance scores arise from missing or chimeric peaks in the non-negative-matrix-factorization deconvolution, so genuine compounds with poor across-sample reproducibility may be discarded.
- The deconvolution relied on a Fast Fourier Transform-based strategy to overcome scalability limitations, which constrains the scale of input data processed.

## Evidence

- [abstract] balance score quantifies reproducibility across samples: "We introduce a “balance score” that quantifies the reproducibility of fragmentation patterns across all samples."
- [abstract] MSHub auto-deconvolution via NMF: "MSHub/GNPS performs auto-deconvolution of compound fragmentation patterns via unsupervised non-negative matrix factorization and quantifies the reproducibility of fragmentation patterns across"
- [abstract] GNPS workflows for GC-MS annotation and networking: "workflows to enable the community to store, process, share, annotate, compare, and perform molecular networking of GC-MS data within the Global Natural Product Social (GNPS) Molecular Networking"
- [readme] GNPS_GC companion repository: [evidence span withheld — anchor status: not_found]
