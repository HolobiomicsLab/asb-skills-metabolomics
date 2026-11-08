---
name: prediction-error-metric-aggregation
description: Use when a benchmark produces per-split error metrics (e.g. MRE and RMSE
  over 101 random 70:30 splits) and you need robust median-based rankings of multiple
  prediction tools per adduct class.
when_to_use_negative:
- When the goal is annotation filtering with a ΔCCS threshold (e.g. 2.8% with 10 ppm
  mass error) rather than cross-tool benchmark ranking.
- When you need a single-split train/test evaluation instead of aggregation over many
  random splits — per-split values, not medians, are the raw output here.
- When training sets cover adducts outside the compendium scope (e.g. dimers like
  [2M+H]+/[2M-H]-), where the underlying predictions themselves are unsupported.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3474
tools:
- name: CCSP 2.0
  role: Predictor under evaluation producing CCS values via linear-kernel SVR models
    on Mordred descriptors; its median MRE/RMSE anchor the ranking
  repo: https://github.com/facundof2016/CCSP2.0
- name: CCSbase
  role: External machine-learning CCS predictor benchmarked on identical splits; contributes
    per-split MRE/RMSE to the aggregation
- name: AllCCS
  role: External web-server CCS predictor benchmarked on identical splits; contributes
    per-split MRE/RMSE to the aggregation
- name: DeepCCS
  role: External command-line CCS predictor benchmarked on identical splits; contributes
    per-split MRE/RMSE to the aggregation
- name: Python
  role: Implementation environment for computing and aggregating per-split metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2022.08.09.503345
    title: 'CCS Predictor 2.0: An Open-Source Jupyter Notebook Tool for Filtering
      Out False Positives in Metabolomics'
  source_content_hash: sha256:977e060e259ad67151264edcbece55fddbed18fa660fc29e9127c7c7a27fcf49
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: ''
  repo_url: https://github.com/facundof2016/CCSP2.0
  tools:
  - AllCCS
  - CCSP 2.0
  - CCSbase
  - DeepCCS
  - Mordred
  - Python
  - RDKit
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# prediction-error-metric-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-split regression error metrics (MRE, RMSE) computed over many random train/test splits into robust medians per method per adduct, then ranks competing predictors (e.g. CCSP 2.0 vs CCSbase vs AllCCS vs DeepCCS).

## When to use

Use when a benchmark produces per-split error metrics (e.g. MRE and RMSE over 101 random 70:30 splits) and you need robust median-based rankings of multiple prediction tools per adduct class.

## When NOT to use

- When the goal is annotation filtering with a ΔCCS threshold (e.g. 2.8% with 10 ppm mass error) rather than cross-tool benchmark ranking.
- When you need a single-split train/test evaluation instead of aggregation over many random splits — per-split values, not medians, are the raw output here.
- When training sets cover adducts outside the compendium scope (e.g. dimers like [2M+H]+/[2M-H]-), where the underlying predictions themselves are unsupported.

## Inputs

- 101 per-split MRE values per tool per adduct (computed against McLean Compendium experimental CCS values)
- 101 per-split RMSE values (A^2) per tool per adduct
- Adduct-labeled cleaned data set sizes (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions)

## Outputs

- Median MRE (%) per tool per adduct
- Median RMSE (A^2) per tool per adduct
- Ranking of tools (e.g. [M-H]- MRE: CCSP 2.0 1.251%, CCSbase 1.427%, AllCCS 1.855%, DeepCCS 1.871%)

## How to apply

For each adduct data set (567 [M-H]-, 518 [M+H]+, 461 [M+Na]+ ions from the McLean Unified CCS Compendium), compute MRE and RMSE for every tool on the identical 101 random 70:30 calibration/test splits, ensuring the same splits are shared across all tools so rankings reflect model quality rather than split choice. Compute MRE as the relative error against the experimentally-measured McLean Compendium CCS value for each withheld test ion, and RMSE in angstroms squared (A^2). Aggregate the 101 per-split values by taking the median (not the mean) per tool per adduct, since median MRE is robust to outlier splits. Rank the tools on median MRE; for example, for [M-H]- the expected ordering is CCSP 2.0 (1.251%), CCSbase (1.427%), AllCCS (1.855%), DeepCCS (1.871%). Cross-check the ranking against median RMSE, which can reorder tools (e.g. DeepCCS RMSE 6.068 A^2 beat CCSbase's 7.359 A^2 despite a worse median MRE).

## Related tools

- **CCSP 2.0** (Predictor under evaluation producing CCS values via linear-kernel SVR models on Mordred descriptors; its median MRE/RMSE anchor the ranking) — https://github.com/facundof2016/CCSP2.0
- **CCSbase** (External machine-learning CCS predictor benchmarked on identical splits; contributes per-split MRE/RMSE to the aggregation)
- **AllCCS** (External web-server CCS predictor benchmarked on identical splits; contributes per-split MRE/RMSE to the aggregation)
- **DeepCCS** (External command-line CCS predictor benchmarked on identical splits; contributes per-split MRE/RMSE to the aggregation)
- **Python** (Implementation environment for computing and aggregating per-split metrics)

## Examples

```
import pandas as pd; med = df.groupby(['tool','adduct'])[['MRE','RMSE']].median().reset_index().sort_values(['adduct','MRE'])  # df holds 101 per-split MRE/RMSE rows per tool per adduct
```

## Evaluation signals

- Reproduced [M-H]- median MRE ranking must be CCSP 2.0 (1.251%), CCSbase (1.427%), AllCCS (1.855%), DeepCCS (1.871%) within sampling variability.
- Reproduced [M-H]- median RMSE values must match 4.876, 7.359, 9.919, 6.068 A^2 for CCSP 2.0, CCSbase, AllCCS, DeepCCS respectively.
- CCSP 2.0 median metrics must fall near published values: MRE 1.251%, 1.729%, 1.874% and RMSE 4.876, 5.755, 6.556 A^2 for [M-H]-, [M+H]+, [M+Na]+.
- All four tools must be evaluated on the identical 101 splits per adduct — verify split indices match across tools before aggregating.
- Median (not mean) aggregation must be used; spot-check that a single anomalous split does not shift the reported median MRE.

## Limitations

- Rankings by median MRE can disagree with rankings by median RMSE (e.g. DeepCCS vs CCSbase on [M-H]-), so a single metric is insufficient.
- Results are specific to the McLean Unified CCS Compendium training sets; predictions for dimers, less common adducts, or in-source fragments are not supported.
- Aggregated medians carry sampling variability across the 101 random splits; exact reproduction is expected only within that variability.

## Evidence
