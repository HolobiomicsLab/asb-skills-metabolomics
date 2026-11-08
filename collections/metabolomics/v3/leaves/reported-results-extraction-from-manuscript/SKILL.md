---
name: reported-results-extraction-from-manuscript
description: Use when a manuscript reports paired before/after optimization metrics
  across multiple biological studies (e.g. GSI, feature counts, annotations) and you
  need a machine-readable, arithmetic-consistency-checked results table.
when_to_use_negative:
- Raw LC-MS data (mzML/mzXML) rather than published metric values is the only input
  — use the BAGO optimization pipeline (getBPCData, findTopSignals, sepEfficiency)
  instead
- Only one condition or study is available, so no paired before/after comparison across
  studies can be assembled
- The manuscript reports only qualitative claims (e.g. 'improved performance') without
  absolute before/after counts needed for arithmetic verification
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Python
  role: Used with pandas to assemble the extracted per-study metric rows into a consolidated
    CSV table
  repo: https://github.com/HuanLab/bago
- name: BAGO
  role: Source of the validated studies and the GSI separation-efficiency metric whose
    reported results are being extracted
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:7885b719451cee378a1d53935a8bbaabd4ab9940f0afcd7d3a195217b8fc8260
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/huaxuyu/bago
  repo_url: https://github.com/HuanLab/bago
  tools:
  - BAGO
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reported-results-extraction-from-manuscript

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts before/after gradient-optimization metrics (GSI, feature counts, MS/MS spectra, annotated compounds) for all six BAGO validation studies from the manuscript text, re-derives the reported percentage gains, and assembles them into a single consolidated results table.

## When to use

Use when a manuscript reports paired before/after optimization metrics across multiple biological studies (e.g. GSI, feature counts, annotations) and you need a machine-readable, arithmetic-consistency-checked results table.

## When NOT to use

- Raw LC-MS data (mzML/mzXML) rather than published metric values is the only input — use the BAGO optimization pipeline (getBPCData, findTopSignals, sepEfficiency) instead
- Only one condition or study is available, so no paired before/after comparison across studies can be assembled
- The manuscript reports only qualitative claims (e.g. 'improved performance') without absolute before/after counts needed for arithmetic verification

## Inputs

- Manuscript results text describing six biological validation studies (mouse cecum HILIC, human urine, serum metabolomics, serum lipidomics, Drosophila HILIC, Drosophila reversed-phase)
- BAGO applications documentation page from https://bago.readthedocs.io/en/latest/applications.html
- Reported absolute before/after counts for features, MS/MS spectra, and annotated compounds

## Outputs

- bago_optimization_results.csv: consolidated table with per-study GSI (before/after), feature counts, unique MS/MS spectra counts, annotated compound counts, and recomputed percentage gains
- Arithmetic verification of every reported percentage gain against reported absolute counts

## How to apply

Read the article's results text and the BAGO applications documentation covering the six validated biological studies (mouse cecum HILIC, human urine, serum metabolomics, serum lipidomics, Drosophila HILIC and reversed-phase). For each study, extract before- and after-optimization values of the gradient separation index (GSI), metabolic/lipidic feature counts, unique MS/MS spectra counts, and annotated compound counts that meet quantification criteria. Recompute each reported percentage gain (e.g. +48.5% annotated compounds, +57 metabolites and +126 lipids in Drosophila, 11.0% more features in mouse cecum) from the reported absolute before/after counts and verify the arithmetic matches. Check that every extracted GSI value lies in [0, 1], consistent with its definition as a normalized omics-scale separation score. Finally, assemble the per-study rows into one table with Python (pandas) and save as a CSV (e.g. bago_optimization_results.csv).

## Related tools

- **Python** (Used with pandas to assemble the extracted per-study metric rows into a consolidated CSV table) — https://github.com/HuanLab/bago
- **BAGO** (Source of the validated studies and the GSI separation-efficiency metric whose reported results are being extracted) — https://github.com/HuanLab/bago

## Examples

```
import pandas as pd; df = pd.DataFrame(rows); df.to_csv('bago_optimization_results.csv', index=False)
```

## Evaluation signals

- Every extracted GSI value lies within [0, 1], consistent with its definition as a normalized separation score
- Each recomputed percentage gain matches the manuscript-reported percentage (e.g. 11.0% feature gain = (1894-1660)/1660)
- All six studies (mouse cecum HILIC, human urine, serum metabolomics, serum lipidomics, Drosophila HILIC, Drosophila reversed-phase) appear as distinct rows
- Before/after counts are paired for the same metric type (features, MS/MS spectra, annotated compounds) within each study row
- Final CSV loads cleanly with pandas and column labels match the metrics reported in the article

## Limitations

- Depends entirely on the manuscript and BAGO applications docs reporting absolute before/after counts; percentages alone cannot be cross-checked arithmetically
- Raw MS data are available only on demand, so extracted values cannot be re-measured from raw mzML/mzXML within this skill
- Metrics are specific to BAGO's gradient-separation context (GSI, feature counts, MS/MS spectra, annotated compounds); they will not generalize to other optimization frameworks without re-mapping the schema

## Evidence

- [full_text] mouse cecum GSI improvement and feature gain: [evidence span withheld — anchor status: not_found]
