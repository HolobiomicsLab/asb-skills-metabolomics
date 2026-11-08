---
name: percentage-gain-consistency-verification
description: Use when consolidating before/after optimization metrics (GSI, feature
  counts, MS/MS spectra, annotated compounds) across multiple studies and you must
  verify each reported percentage gain is arithmetically consistent with the absolute
  counts.
when_to_use_negative:
- Input is already a validated feature table with recomputed gains — skip re-derivation
  and only spot-check
- The source text reports only percentage gains with no absolute before/after counts,
  making arithmetic cross-checking impossible
- You are optimizing a gradient with the BAGO software itself rather than verifying
  published optimization results
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Used to recompute percentage gains, assert GSI ∈ [0,1], and assemble the consolidated
    results table
  repo: https://github.com/HuanLab/bago
- name: BAGO
  role: The Bayesian LC gradient optimization framework whose six validated biological
    studies supply the before/after metrics being verified
  repo: https://github.com/HuanLab/bago
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2023.09.08.556930
    title: Bayesian optimization of separation gradients to maximize the performance
      of untargeted LC-MS
  source_content_hash: sha256:8e1218d1592340c3557fb7e05460aa51be396d5d6f58e0b8d845a80abc73b46b
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# percentage-gain-consistency-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A cross-study result-verification skill for the BAGO LC gradient optimization paper: recompute the reported percentage gains (e.g. +48.5% annotated compounds, +57 metabolites and +126 lipids in Drosophila, +11.0% features in mouse cecum HILIC) from their absolute before/after counts, validate all GSI values lie in [0, 1], and assemble the six validated studies into one pandas table.

## When to use

Use when consolidating before/after optimization metrics (GSI, feature counts, MS/MS spectra, annotated compounds) across multiple studies and you must verify each reported percentage gain is arithmetically consistent with the absolute counts.

## When NOT to use

- Input is already a validated feature table with recomputed gains — skip re-derivation and only spot-check
- The source text reports only percentage gains with no absolute before/after counts, making arithmetic cross-checking impossible
- You are optimizing a gradient with the BAGO software itself rather than verifying published optimization results

## Inputs

- Article results text and BAGO applications documentation reporting before/after metrics for the six validation studies
- Per-study absolute counts: metabolic/lipidic features, high-quality features, unique MS/MS spectra, annotated compounds, and GSI values
- Reported percentage gains (e.g. up to 48.5% more annotated compounds meeting quantification criteria)

## Outputs

- bago_optimization_results.csv — one row per validated study with before/after GSI, feature counts, MS/MS spectra, annotated compound counts, and recomputed percentage gains
- Per-study verification verdicts: whether each recomputed gain matches the reported value and whether each GSI lies in [0, 1]

## How to apply

Read the article results text and BAGO applications documentation covering the six validation studies (mouse cecum HILIC, human urine, serum metabolomics, serum lipidomics, Drosophila HILIC and reversed-phase). For each study, extract the before- and after-optimization values of the gradient separation index (GSI), metabolic/lipidic feature counts, unique MS/MS spectra counts, and annotated compound counts. Recompute each reported percentage gain from the absolute counts (gain% = (after − before) / before × 100) — e.g. mouse cecum HILIC features 1660 → 1894 (+11.0%), high-quality features 1000 → 1260 (+26.0%), GSI 0.0345 → 0.171 — and flag any mismatch with the reported value. Check that every extracted GSI falls in [0, 1], consistent with its definition as a normalized separation score. Assemble the per-study rows into a single table with Python (pandas) and save it as bago_optimization_results.csv.

## Related tools

- **Python** (Used to recompute percentage gains, assert GSI ∈ [0,1], and assemble the consolidated results table) — https://github.com/HuanLab/bago
- **BAGO** (The Bayesian LC gradient optimization framework whose six validated biological studies supply the before/after metrics being verified) — https://github.com/HuanLab/bago

## Examples

```
import pandas as pd; df = pd.DataFrame(rows); assert df['gsi'].between(0,1).all(); df.to_csv('bago_optimization_results.csv', index=False)
```

## Evaluation signals

- Recomputed percentage gains match the article-reported values (e.g. (1894−1660)/1660 ≈ +14.1% must be reconciled against the reported +11.0%; (1260−1000)/1000 = +26.0% must match exactly)
- All extracted GSI values satisfy 0 ≤ GSI ≤ 1 (e.g. mouse cecum HILIC GSI 0.0345 → 0.171)
- Final table contains one row per validated study — six studies covering both HILIC and reversed-phase columns and different sample matrices
- Saved CSV (bago_optimization_results.csv) reproduces the article-reported reference values when re-read with pandas
- Drosophila deltas equal the reported +57 metabolites and +126 lipids when recomputed from absolute counts

## Limitations

- Percentage gains cannot be verified when only the percentages are reported without the underlying absolute before/after counts
- Some reported gains (e.g. mouse cecum HILIC +11.0% from 1660→1894) may not reconcile exactly with naive recomputation, requiring reconciliation against the article's stated basis before acceptance
- Raw MS data underlying the published metrics are only available on demand, so counts cannot be independently re-derived from spectra

## Evidence
