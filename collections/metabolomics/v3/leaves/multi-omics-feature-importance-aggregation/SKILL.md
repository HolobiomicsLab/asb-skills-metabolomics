---
name: multi-omics-feature-importance-aggregation
description: Use when per-metabolite model feature importances exist across multiple
  omics feature sets (e.g., 9 feature sets x 225 metabolites) and the goal is a global,
  cross-omics top-regulator ranking.
when_to_use_negative:
- 'When the goal is mechanistic/causal regulation: the ''guilt by association'' aggregation
  is descriptive and not necessarily mechanistic.'
- When applying to a single metabolite model or a single omics class — the aggregation
  is only meaningful across all 225 metabolite models and multiple feature sets.
- 'When the trained models come from non-CCLE sources: the models are not directly
  transferrable due to differences between in vitro and in vivo conditions and limited
  feature overlap.'
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: scikit-learn Random Forest Regressor
  role: Trains the per-metabolite random forests whose feature importances feed this
    aggregation
- name: Python
  role: Implements the rank-averaging, Z-score scaling, and per-omics counting that
    produce top_regulators.csv
  repo: sriram-lab/Metab8D
- name: Metab8D (ML_function.ipynb)
  role: Generates the top-20 feature lists and confidence scores (Metab8D_network.xlsx)
    that are the raw input to aggregation
  repo: sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:1434179b8354b24cbeb884eb25867a49eb8c8290041994772e3b4d60d7583e81
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Metab8D (ML_function.ipynb)
  - Python
  - scikit-learn
  - scikit-learn Random Forest Regressor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# multi-omics-feature-importance-aggregation

## Summary

Aggregates per-metabolite random-forest feature importances across the 225 metabolite models and all omics feature sets (transcriptomics, proteomics, methylation, phosphoproteomics, CNV, etc.) into a normalized global top-regulator ranking, revealing cross-omics consistent regulators such as YAP1.

## When to use

Use when per-metabolite model feature importances exist across multiple omics feature sets (e.g., 9 feature sets x 225 metabolites) and the goal is a global, cross-omics top-regulator ranking.

## When NOT to use

- When the goal is mechanistic/causal regulation: the 'guilt by association' aggregation is descriptive and not necessarily mechanistic.
- When applying to a single metabolite model or a single omics class — the aggregation is only meaningful across all 225 metabolite models and multiple feature sets.
- When the trained models come from non-CCLE sources: the models are not directly transferrable due to differences between in vitro and in vivo conditions and limited feature overlap.

## Inputs

- Metab8D_network.xlsx: top 20 features for 2,025 trained metabolite models (nine feature sets x 225 metabolites) with confidence scores 0-8
- RF_results: Pearson's correlations and Bonferroni-corrected P values for all metabolite models from each of nine feature sets
- Feature importances (Shapley analysis) for select predicted metabolites from ML_function.ipynb outputs

## Outputs

- top_regulators.csv: global top-regulator table with per-feature averaged and Z-scored importance ranks across omics classes
- Per-omics count table of how many metabolite models each top feature ranks first (e.g., YAP1: 35/2/5/132 in transcriptomics/proteomics/methylation/phosphoproteomics)
- Cross-omics consistency summary for key regulators (GPX8, miR-142, EPHA2, ERBB2)

## How to apply

Load the top-20 features and confidence scores (0-8) from Metab8D_network.xlsx, which holds all 2,025 trained metabolite models (nine feature sets for 225 metabolites, genomic mutations excluded due to their binary structure). For each omics class, average each feature's rank across all 225 per-metabolite models, then Z-score scale the per-class average ranks to obtain a normalized global importance score per feature. Rank features across all classes to reproduce the global top-regulator table (expect YAP1 Z≈69, GPX8 Z≈61, SCD Z≈59). Separately, count in how many metabolite models each top feature ranks first per omics class (e.g., YAP1: 35 transcriptomics / 2 proteomics / 5 methylation / 132 phosphoproteomics) to assess cross-omics consistency, and optionally cross-check secondary regulators such as GPX8, miR-142, EPHA2, and ERBB2 before writing the aggregated table.

## Related tools

- **scikit-learn Random Forest Regressor** (Trains the per-metabolite random forests whose feature importances feed this aggregation)
- **Python** (Implements the rank-averaging, Z-score scaling, and per-omics counting that produce top_regulators.csv) — sriram-lab/Metab8D
- **Metab8D (ML_function.ipynb)** (Generates the top-20 feature lists and confidence scores (Metab8D_network.xlsx) that are the raw input to aggregation) — sriram-lab/Metab8D

## Examples

```
avg = df.groupby(['feature_set','feature']).feature_rank.mean(); z = (avg - avg.mean()) / avg.std(); z.sort_values(ascending=False).to_csv('top_regulators.csv')
```

## Evaluation signals

- Row-count invariant: aggregated table must cover 2,025 model entries (nine feature sets x 225 metabolites), with the genomic mutations feature set excluded.
- Expected ranking: YAP1 is the top global regulator (Z≈69), followed by GPX8 (Z≈61) and SCD (Z≈59).
- Expected per-omics top-feature counts for YAP1: 35 (transcriptomics), 2 (proteomics), 5 (methylation), 132 (phosphoproteomics).
- All confidence scores must fall in the 0-8 range, reflecting the number of controls (out of 8 experiments) in which each feature appeared in the top 20.
- Secondary regulators GPX8, miR-142, EPHA2, and ERBB2 should show consistency across omics classes in the aggregated table.

## Limitations

- The 90 phosphoproteins were measured with a targeted approach and are not representative of the entire cellular phosphoprotein array, biasing phosphoproteomics-based importance counts.
- The metabolite panel of 225 metabolites is only a fraction of the metabolites that exist within a cell across compartments, so the global ranking covers a limited metabolome.
- Network associations from systems-level data are descriptive ('guilt by association') and findings are not necessarily mechanistic; experimentally validating proposed interactions would require an immense effort.

## Evidence
