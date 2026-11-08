---
name: z-score-normalization-of-rank-metrics
description: Use when aggregating per-model feature-importance ranks across multiple
  omics feature sets into one global regulator ranking, e.g. combining ranks from
  225 metabolite random-forest models over nine feature sets where raw ranks are not
  directly comparable across classes.
when_to_use_negative:
- Normalizing raw omics feature matrices before model training — the paper z-scores
  the feature data within each CV fold, not aggregated ranks
- Analyzing a single metabolite model where no cross-omics aggregation is needed —
  per-model Shapley/importance analysis suffices
- Applying the ranking to external/in vivo metabolomics data — the article states
  such models are not directly transferrable due to in vitro vs in vivo differences
  and limited feature overlap
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: scikit-learn
  role: Provides the Random Forest Regressor feature importances whose ranks are averaged
    and z-scored across metabolite models
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Language for the Metab8D ML workflow scripts that generate and aggregate the
    feature-importance ranks
  repo: https://github.com/sriram-lab/Metab8D
- name: Metab8D GitHub repository
  role: Source of ML_function.ipynb and Metab8D_network.xlsx (top 20 features per
    metabolite model with confidence scores) used as aggregation input
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:054a645fb33f8da42f369cd568802a9095fe8efc7c9a9384a9f8eff42a3e9e25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Metab8D GitHub repository
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# z-score-normalization-of-rank-metrics

## Summary

Scale per-omics-class average feature-importance ranks by z-scoring so that feature sets from different biomolecular classes (transcriptomics, proteomics, methylation, phosphoproteomics, etc.) become comparable in a single global ranking of metabolic regulators such as YAP1, GPX8, and SCD.

## When to use

Use when aggregating per-model feature-importance ranks across multiple omics feature sets into one global regulator ranking, e.g. combining ranks from 225 metabolite random-forest models over nine feature sets where raw ranks are not directly comparable across classes.

## When NOT to use

- Normalizing raw omics feature matrices before model training — the paper z-scores the feature data within each CV fold, not aggregated ranks
- Analyzing a single metabolite model where no cross-omics aggregation is needed — per-model Shapley/importance analysis suffices
- Applying the ranking to external/in vivo metabolomics data — the article states such models are not directly transferrable due to in vitro vs in vivo differences and limited feature overlap

## Inputs

- Per-metabolite random-forest feature importances for 225 metabolites across nine omics feature sets (Metab8D_network.xlsx / ML_function.ipynb outputs)
- Trained random forest models and CCLE omics data deposited on the Recon8D Synapse page
- RF_results Pearson's correlations and Bonferroni-corrected P values for identifying significant metabolite models

## Outputs

- top_regulators.csv — global top-regulator table with z-scored importance scores per feature and per-omics-class top-feature counts
- Cross-omics consistency summary (e.g. YAP1 Z≈69, GPX8 Z≈61, SCD Z≈59; YAP1 35/2/5/132 across transcriptomics/proteomics/methylation/phosphoproteomics)

## How to apply

Load the per-metabolite random-forest feature importances produced by ML_function.ipynb (top 20 features for 225 metabolites from nine feature sets, excluding genomic mutations, with confidence scores 0-8 in Metab8D_network.xlsx). For each omics class, average each feature's rank across all 225 metabolite models so the class-level importance is scale-free. Z-score scale the per-class average ranks (each class normalized to mean 0, SD 1) so classes with different rank distributions contribute comparably to the global score. Rank features across all classes to obtain the global top-regulator table, then count in how many metabolite models each top feature appears as a top feature per class (e.g. YAP1: 35 transcriptomics / 2 proteomics / 5 methylation / 132 phosphoproteomics). Write the aggregated table and summary to top_regulators.csv. The rationale is that z-scoring removes between-class scale differences, mirroring the paper's use of z-score normalization as its standard scaling step.

## Related tools

- **scikit-learn** (Provides the Random Forest Regressor feature importances whose ranks are averaged and z-scored across metabolite models) — https://github.com/sriram-lab/Metab8D
- **Python** (Language for the Metab8D ML workflow scripts that generate and aggregate the feature-importance ranks) — https://github.com/sriram-lab/Metab8D
- **Metab8D GitHub repository** (Source of ML_function.ipynb and Metab8D_network.xlsx (top 20 features per metabolite model with confidence scores) used as aggregation input) — https://github.com/sriram-lab/Metab8D

## Evaluation signals

- Z-scored per-class average ranks should have mean 0 and SD 1 within each omics class before global ranking
- Global top three regulators should reproduce YAP1 (Z≈69), GPX8 (Z≈61), and SCD (Z≈59)
- YAP1 top-feature counts should match 35/2/5/132 in transcriptomics/proteomics/methylation/phosphoproteomics
- Aggregated table should cover features from nine feature sets (225 metabolites × 9 models = 2,025), excluding genomic mutations
- Output file top_regulators.csv should be produced with both z-scores and per-class consistency counts

## Limitations

- Feature importance ranks from 'guilt by association' network inference are descriptive and not necessarily mechanistic
- Only 90 phosphoproteins were measured with a targeted approach, so the phosphoproteomics rank pool is not representative of the full phosphoprotein array
- The 225-metabolite panel is only a fraction of cellular metabolites, so the global ranking is bounded by the assayed metabolites
- Random-forest models were trained on CCLE cancer cell lines in culture and may not reflect metabolism in vivo

## Evidence
