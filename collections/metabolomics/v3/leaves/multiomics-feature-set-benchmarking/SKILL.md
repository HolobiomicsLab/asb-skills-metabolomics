---
name: multiomics-feature-set-benchmarking
description: Use when a study has matched multiomic feature tables and a metabolite
  abundance matrix for the same samples (e.g. CCLE cell lines), and the goal is to
  determine which omics class most accurately predicts metabolomic variation.
when_to_use_negative:
- When the omics input has no matched metabolite abundance measurements for the same
  samples (models are not directly transferrable to other sources due to differences
  between in vitro and in vivo conditions, cell line panels, and limited feature overlap).
- 'When the objective is causal mechanism discovery rather than predictive feature
  benchmarking: the ''guilt by association'' strategy may not identify causative mechanisms
  and network associations from systems-level data are descriptive.'
- When only a small set of omics classes is available and the comparison across classes
  is not the goal — the benchmarking loses its discriminating purpose.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3337
tools:
- name: scikit-learn
  role: RandomForestRegressor used to train and cross-validate per-metabolite models
    for each feature set
- name: XGBoost
  role: Benchmarking algorithm compared against random forest during algorithm selection
- name: Python
  role: Scripting language for the ML workflow (ML_function.ipynb)
  repo: https://github.com/sriram-lab/Metab8D
- name: machine learning (random forest regression)
  role: Primary ML algorithm for metabolomic prediction, chosen because it outperformed
    other methods on cross-validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:dcce1ebe4540b318b371e57eb71a9e0eb63b638102909e42c00df0710a4411f8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - XGBoost
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# multiomics-feature-set-benchmarking

## Summary

Benchmark omics feature sets from eight biomolecular classes (CNV, methylation, histone PTMs, coding/non-coding transcriptomics, miRNA/lncRNA, proteomics, phosphoproteomics) for their ability to predict metabolite abundance across matched cancer cell lines using per-metabolite random forest regression with 5-fold cross-validation and Pearson's correlation as the accuracy metric.

## When to use

Use when a study has matched multiomic feature tables and a metabolite abundance matrix for the same samples (e.g. CCLE cell lines), and the goal is to determine which omics class most accurately predicts metabolomic variation.

## When NOT to use

- When the omics input has no matched metabolite abundance measurements for the same samples (models are not directly transferrable to other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap).
- When the objective is causal mechanism discovery rather than predictive feature benchmarking: the 'guilt by association' strategy may not identify causative mechanisms and network associations from systems-level data are descriptive.
- When only a small set of omics classes is available and the comparison across classes is not the goal — the benchmarking loses its discriminating purpose.

## Inputs

- Ten multiomic feature sets from eight biomolecular classes: genomic CNV, genomic mutations (excluded from RF results), epigenomic DNA methylation, epigenomic histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics (CCLE-derived)
- Matched 225-metabolite abundance matrix for the same cancer cell lines

## Outputs

- RF_results.csv: Pearson's correlations and P values for all metabolite models from each of nine feature sets
- Metab8D_network.xlsx: top 20 features per metabolite model per feature set with confidence scores
- Cross-validated significance calls per metabolite-feature-set pair (Bonferroni-corrected, 0.05/225)

## How to apply

Load each of the ten feature sets from eight biomolecular classes (excluding binary mutation data from random forest results) and the matched 225-metabolite abundance matrix from the CCLE cell line panel. Preprocess each feature set by replacing missing values (NaN) with zeros and removing features with zero variance. For each feature set and each of the 225 metabolites, train a scikit-learn RandomForestRegressor with 5-fold cross-validation, applying Z-score normalization within each CV fold, and assess performance using Pearson's correlation between predicted and true metabolite concentrations. Determine statistically significant predictions using a Bonferroni-corrected P value (0.05/225). Compile cross-validated correlations, P values, and significance calls for all 225 metabolites x 9 feature sets into RF_results.csv, then compare feature sets to identify which omics class consistently yields the most accurate metabolomic predictions.

## Related tools

- **scikit-learn** (RandomForestRegressor used to train and cross-validate per-metabolite models for each feature set)
- **XGBoost** (Benchmarking algorithm compared against random forest during algorithm selection)
- **Python** (Scripting language for the ML workflow (ML_function.ipynb)) — https://github.com/sriram-lab/Metab8D
- **machine learning (random forest regression)** (Primary ML algorithm for metabolomic prediction, chosen because it outperformed other methods on cross-validation)

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rfr = RandomForestRegressor(n_estimators=1000); rfr.fit(X_train, y_train)
```

## Evaluation signals

- All 225 metabolites x 9 feature sets produce a finite Pearson's correlation and P value in RF_results.csv.
- Bonferroni-corrected significance threshold equals 0.05/225 for every metabolite-feature-set pair.
- Zero-variance features are removed before model training for every feature set.
- Transcriptomics (coding and non-coding RNAs) consistently outperforms other omics classes in cross-validated correlation.
- Feature importances for top 20 features per metabolite are recoverable with confidence scores (0 through 8) based on the number of control experiments.

## Limitations

- Mutations were excluded from random forest model results due to their binary structure.
- Models trained on CCLE cell line data are not directly transferrable to predicting metabolomics from other sources (in vitro vs. in vivo differences, cell line panel differences, limited feature overlap).
- The metabolite panel of 225 metabolites is only a fraction of metabolites that exist within a cell across compartments.
- Only 90 phosphoproteins were measured with a targeted approach, not representative of the entire cellular phosphoprotein array.
- Differences in sample quantities between omics classes can influence accuracies; omics with too few samples (cell lines) may not perform as well.

## Evidence

- [methods] Performance of each model was assessed using Pearson's correlation between predicted and true metabolite concentrations: [evidence span withheld — anchor status: not_found]
- [methods] Data preprocessing involved replacing missing values (NaN) with zeros, removing features with 0 variance: [evidence span withheld — anchor status: not_found]
