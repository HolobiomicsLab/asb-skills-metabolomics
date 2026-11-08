---
name: metabolite-model-feature-ranking
description: Use when you have trained one model per metabolite across multiple omics
  feature sets and need to identify which features are globally important or consistent
  regulators across the metabolome.
when_to_use_negative:
- Input is already a single aggregated feature-importance table (this skill aggregates
  raw per-metabolite model importances).
- Only one omics feature set is available and no cross-omics consistency comparison
  is desired.
- The trained models are from a non-cancer cell line or non-CCLE data source; the
  article notes such models are not directly transferrable due to differences between
  in vitro and in vivo conditions, cell line panels, and limited feature overlap.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_2259
tools:
- name: scikit-learn
  role: Provides the Random Forest Regressor used to generate the per-metabolite feature
    importances that are subsequently aggregated for global ranking.
  repo: https://github.com/sriram-lab/Metab8D
- name: XGBoost
  role: Benchmarked alternative regression algorithm alongside random forests in the
    ML_function.ipynb workflow; feature importances can be extracted from these models
    as well.
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Language of the Recon8D/Metab8D ML workflow scripts that generate and aggregate
    per-metabolite feature importances.
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:4160790d3f17f8d1633e4b89a44428eda7531636ee09bd805e1ac2287df1ee77
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - XGBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabolite-model-feature-ranking

## Summary

Aggregates per-metabolite random forest feature importances from the Metab8D workflow to identify globally important multiomic predictors of metabolomic variation, such as YAP1 signaling, and reports cross-omics consistency counts per regulator.

## When to use

Use when you have trained one model per metabolite across multiple omics feature sets and need to identify which features are globally important or consistent regulators across the metabolome.

## When NOT to use

- Input is already a single aggregated feature-importance table (this skill aggregates raw per-metabolite model importances).
- Only one omics feature set is available and no cross-omics consistency comparison is desired.
- The trained models are from a non-cancer cell line or non-CCLE data source; the article notes such models are not directly transferrable due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap.

## Inputs

- Per-metabolite random forest feature importances from 225 metabolite models (Metab8D_network.xlsx: top 20 features per metabolite per feature set, with confidence scores 0–8)
- RF_results: Pearson's correlations and P values for all metabolite models from each of nine feature sets (for significance via Bonferroni-corrected P value 0.05/225)

## Outputs

- top_regulators.csv: aggregated global feature-importance table with Z-scored per-omics ranks and cross-omics consistency counts (e.g. YAP1: 35/2/5/132 in transcriptomics/proteomics/methylation/phosphoproteomics)
- Ranked list of top global regulators (e.g. YAP1, GPX8, SCD, miR-142, EPHA2, ERBB2) with number of metabolite models in which each appears as a top feature

## How to apply

Load the per-metabolite feature importances produced by the ML_function.ipynb workflow (Metab8D_network.xlsx contains the top 20 features per metabolite per feature set with confidence scores 0–8 based on how many of 8 control experiments each feature appeared in). For each omics class — coding transcripts, splicing, miRNA, lncRNA, proteomics, phosphoproteomics, DNA methylation, histone PTMs, and CNV — average the feature ranks across all 225 metabolite models, then Z-score scale per-class average ranks to obtain a normalized global importance score per feature. Rank features across all classes to produce the global top-regulator table (e.g. YAP1 Z≈69, GPX8 Z≈61, SCD Z≈59), and count how many metabolite models in which each top feature appears within the top 20 (e.g. YAP1: 35 transcriptomics, 2 proteomics, 5 methylation, 132 phosphoproteomics). Cross-check consistency of regulators (GPX8, miR-142, EPHA2, ERBB2) across omics classes. Judge correctness by verifying the aggregated top-feature counts match the per-model top-20 lists in Metab8D_network.xlsx and that the top regulators are stable across omics classes.

## Related tools

- **scikit-learn** (Provides the Random Forest Regressor used to generate the per-metabolite feature importances that are subsequently aggregated for global ranking.) — https://github.com/sriram-lab/Metab8D
- **XGBoost** (Benchmarked alternative regression algorithm alongside random forests in the ML_function.ipynb workflow; feature importances can be extracted from these models as well.) — https://github.com/sriram-lab/Metab8D
- **Python** (Language of the Recon8D/Metab8D ML workflow scripts that generate and aggregate per-metabolite feature importances.) — https://github.com/sriram-lab/Metab8D

## Examples

```
python ML_function.ipynb --feature-set histone_ptm --data example_datasets/histone_ptm_zero_imputed.csv --n-top-features 20
```

## Evaluation signals

- Verify that the aggregated top-feature counts per omics class for a given regulator (e.g. YAP1: 35 transcriptomics, 2 proteomics, 5 methylation, 132 phosphoproteomics) match the per-model top-20 lists in Metab8D_network.xlsx.
- Confirm the top three global regulators reproduce YAP1 (Z≈69), GPX8 (Z≈61), and SCD (Z≈59).
- Check that Z-score scaling is applied per omics class (not globally) so cross-class ranks are comparable.
- Confirm the input feature sets exclude genomic mutations (binary structure) as stated in the README, leaving nine feature sets × 225 metabolites = 2,025 trained models.
- Sanity-check that no top-regulator feature appears as a top feature in fewer classes than its confidence score in Metab8D_network.xlsx would allow.

## Limitations

- Random forest feature importance reflects association ('guilt by association') rather than causation; top predictors are descriptive, not mechanistic.
- Models are trained on CCLE cancer cell lines in culture and may not reflect in vivo metabolism; they are not directly transferrable to other data sources.
- The metabolite panel is only 225 metabolites, a fraction of those in a cell across compartments, so the global ranking misses metabolites outside the panel.
- Only 90 phosphoproteins were measured with a targeted approach, so phosphoproteomic feature importance is not representative of the full phosphoproteome.
- Genomic mutations are excluded from random forest model results due to their binary structure.

## Evidence

- [discussion] guilt by association caveat: [evidence span withheld — anchor status: not_found]
