---
name: cross-omics-regulator-consistency-check
description: Use when per-metabolite model feature importances are available for multiple
  omics feature sets and you need to identify globally influential regulators, e.g.
  confirming YAP1 as a top predictor across 4 omic layers.
when_to_use_negative:
- Only one omics feature set is available — no cross-omics consistency can be assessed.
- The goal is single-metabolite feature importance (use Shapley analysis per metabolite
  instead), not global metabolome-wide regulator ranking.
- Input is raw omics data rather than trained-model importances; models must first
  be trained and their feature importances extracted (see ML_function.ipynb).
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3339
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: RandomForestRegressor provides the per-metabolite feature importances that
    are aggregated across omics classes for the consistency check
- name: Python
  role: Scripting language for averaging ranks, Z-score scaling, and counting cross-class
    consistency; script available in the Recon8D GitHub repository
  repo: https://github.com/sriram-lab/Metab8D
- name: machine learning (random forest regression)
  role: The trained random forest models (one per metabolite per feature set) supply
    the feature-importance matrix that feeds the global ranking
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:a18a2004f166126d5dae0b352d210afe21edb059e60a2bb264ccba3069cac086
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3339
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cross-omics-regulator-consistency-check

## Summary

Aggregates per-metabolite random-forest feature importances across the eight biomolecular feature classes (transcriptomics, proteomics, methylation, phosphoproteomics, miRNA, lncRNA, CNV, histone PTMs) from the 225 CCLE metabolite models in Metab8D to rank global metabolome regulators and count how many models each feature tops in each omics class.

## When to use

Use when per-metabolite model feature importances are available for multiple omics feature sets and you need to identify globally influential regulators, e.g. confirming YAP1 as a top predictor across 4 omic layers.

## When NOT to use

- Only one omics feature set is available — no cross-omics consistency can be assessed.
- The goal is single-metabolite feature importance (use Shapley analysis per metabolite instead), not global metabolome-wide regulator ranking.
- Input is raw omics data rather than trained-model importances; models must first be trained and their feature importances extracted (see ML_function.ipynb).

## Inputs

- Metab8D_network.xlsx (top 20 features for 225 RF metabolite models from nine feature sets with confidence scores 0–8)
- Per-metabolite random-forest feature importances from ML_function.ipynb outputs (225 metabolites × 9 feature sets)
- RF_results: Pearson's correlations and P values for all metabolite models from each of nine feature sets (for filtering significant models)

## Outputs

- Global regulator ranking table with Z-scaled cross-omics importance scores (top_regulators.csv)
- Per-regulator cross-omics consistency counts (e.g. number of metabolite models per feature class where the feature ranks in the top 20)

## How to apply

Load the top-20 feature and confidence-score output (Metab8D_network.xlsx) or per-model importances from ML_function.ipynb for all 225 metabolites across the nine feature sets. For each omics class, average the per-metabolite feature ranks, then Z-score scale the class-wise averages so importances are comparable across feature sets; rank features across classes to recover the global regulator table (YAP1 Z≈69, GPX8 Z≈61, SCD Z≈59). For each top regulator, count in how many metabolite models it appears as a top feature within each feature class (e.g. YAP1: 35 transcriptomics, 2 proteomics, 5 methylation, 132 phosphoproteomics), and flag features that are consistent across omics layers as high-priority mechanistic candidates. Cross-check any identified regulator against the Recon3D reaction-mapping scripts (recon_mapping) to connect it to metabolically relevant genes.

## Related tools

- **scikit-learn** (RandomForestRegressor provides the per-metabolite feature importances that are aggregated across omics classes for the consistency check)
- **Python** (Scripting language for averaging ranks, Z-score scaling, and counting cross-class consistency; script available in the Recon8D GitHub repository) — https://github.com/sriram-lab/Metab8D
- **machine learning (random forest regression)** (The trained random forest models (one per metabolite per feature set) supply the feature-importance matrix that feeds the global ranking)

## Examples

```
python aggregate_regulators.py --network Metab8D_network.xlsx --rf_results RF_results --out top_regulators.csv
```

## Evaluation signals

- YAP1 should emerge as the top global regulator across 4 omic layers with Z≈69, followed by GPX8 (Z≈61) and SCD (Z≈59).
- YAP1 cross-omics counts should reproduce 35 (transcriptomics), 2 (proteomics), 5 (DNA methylation), and 132 (phosphoproteomics) metabolite models where it ranks as a top feature.
- Every feature class average should be Z-score scaled before cross-class comparison; unscaled ranks would bias the global ranking toward omics classes with more features.
- Models with Pearson's correlation below the Bonferroni-corrected P value threshold (0.05/225) in RF_results should be excluded or down-weighted before aggregation.
- Consistency counts should fall between 0 and 225 for each feature class, and regulators flagged as consistent should appear in ≥ 2 distinct omics classes.

## Limitations

- Feature importances come from a 'guilt by association' strategy and are descriptive, not necessarily mechanistic — cross-omics consistency does not prove causation.
- Only 90 phosphoproteins were measured (targeted approach), so phosphoproteomics importances do not represent the full cellular phosphoprotein array.
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo; models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions and limited feature overlap.

## Evidence

- [methods] Performance of each model was assessed using Pearson's correlation between predicted and true metabolite concentrations: [evidence span withheld — anchor status: not_found]
