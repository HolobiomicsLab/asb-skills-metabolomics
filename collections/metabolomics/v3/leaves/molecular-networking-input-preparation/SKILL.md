---
name: molecular-networking-input-preparation
description: Use when MSHub deconvolution output (per-compound fragmentation/abundance
  table) must be annotated with balance scores and filtered before being used as GNPS
  molecular networking input.
when_to_use_negative:
- The table is already the balance-score-filtered output (filtered_annotations.csv);
  re-filtering would compound the thresholds.
- The GC-MS data are still raw and have not been auto-deconvoluted by MSHub, so no
  per-compound balance score exists yet.
- The analysis is LC-MS/MS feature-based molecular networking (e.g., MZmine/GNPS feature
  tables) rather than MSHub GC-MS deconvolution output.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MSHub
  role: Produces the deconvoluted per-compound feature/quantification table and the
    balance score (fragmentation-pattern reproducibility across samples) that this
    preparation/filtering step operates on
  repo: github.com/bittremieux/GNPS_GC
- name: GNPS
  role: Consumes the filtered, balance-score-annotated compound table as the input
    to molecular networking of GC-MS data
  repo: http://github.com/CCMS-UCSD/GNPS_Workflows
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-0700-3
    title: Auto-deconvolution and molecular networking of gas chromatography–mass
      spectrometry data
  source_content_hash: sha256:54b0d96e3b76c85f6de4430930b519031fee99bea7dd1d99899ed9e84f98268f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS
  - MSHub
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# molecular-networking-input-preparation

## Summary

Prepares MSHub/GNPS deconvoluted GC-MS compound tables for molecular networking by annotating each deconvoluted compound with its balance score — a metric of fragmentation-pattern reproducibility across all samples — and filtering annotations at a balance-score threshold applied jointly with the cosine match score.

## When to use

Use when MSHub deconvolution output (per-compound fragmentation/abundance table) must be annotated with balance scores and filtered before being used as GNPS molecular networking input.

## When NOT to use

- The table is already the balance-score-filtered output (filtered_annotations.csv); re-filtering would compound the thresholds.
- The GC-MS data are still raw and have not been auto-deconvoluted by MSHub, so no per-compound balance score exists yet.
- The analysis is LC-MS/MS feature-based molecular networking (e.g., MZmine/GNPS feature tables) rather than MSHub GC-MS deconvolution output.

## Inputs

- MSHub deconvolution feature/quantification table: per-compound fragmentation pattern and abundance across GC-MS samples
- Cosine match scores of deconvoluted compound spectra against annotation matches
- Analyst-specified balance-score threshold (and joint cosine threshold)

## Outputs

- filtered_annotations.csv: threshold-passing compounds with the balance-score column appended to the input feature-table schema
- Score-annotated compound table ready to serve as input to the GNPS molecular networking workflow

## How to apply

Load the MSHub deconvolution feature/quantification table — one row per deconvoluted compound, with its fragmentation pattern and abundance across GC-MS samples — into pandas. Compute or attach the balance score that MSHub derives for each compound during unsupervised non-negative matrix factorization deconvolution: it quantifies how reproducible the fragmentation pattern is across all samples, so missing or chimeric peaks in the deconvoluted pattern depress the score. Filter the annotation set to retain only compounds whose balance score meets the specified threshold, applying the cosine match score jointly as the second spectral-confidence criterion; no numeric defaults are published, so set and record both thresholds explicitly. Write the retained, score-annotated compounds to filtered_annotations.csv, preserving the input feature-table column schema and appending the balance-score column. Treat this filtered table as the prepared input node/annotation table for molecular networking of GC-MS data within GNPS.

## Related tools

- **MSHub** (Produces the deconvoluted per-compound feature/quantification table and the balance score (fragmentation-pattern reproducibility across samples) that this preparation/filtering step operates on) — github.com/bittremieux/GNPS_GC
- **GNPS** (Consumes the filtered, balance-score-annotated compound table as the input to molecular networking of GC-MS data) — http://github.com/CCMS-UCSD/GNPS_Workflows

## Examples

```
import pandas as pd; df = pd.read_csv("mshub_quant_table.csv"); df = df[df["balance_score"] >= BALANCE_THRESHOLD]; df.to_csv("filtered_annotations.csv", index=False)
```

## Evaluation signals

- filtered_annotations.csv exists and is non-empty after filtering
- Every retained compound row carries a balance-score field whose value meets the applied threshold
- Column schema of filtered_annotations.csv matches the input feature-table schema plus the added balance-score column
- Compounds with low balance scores (missing or chimeric peaks in the deconvoluted pattern) are absent from the retained set
- Retained annotations also satisfy the jointly applied cosine match-score criterion

## Limitations

- The balance score is a deconvolution-confidence metric: compounds with missing or chimeric peaks score low and may be discarded even if genuinely present in the samples.
- No default balance-score or cosine-similarity threshold values are published in the manuscript text or READMEs; thresholds must be chosen by the analyst.
- The bittremieux/GNPS_GC companion README is minimal (citation only) and the GNPS_Workflows README does not document the GC deconvolution scoring parameters, so parameter provenance rests on the manuscript.

## Evidence

- [readme] GNPS hosts the workflows into which the prepared GC-MS compound table is loaded for molecular networking.: [evidence span withheld — anchor status: external_doc]
