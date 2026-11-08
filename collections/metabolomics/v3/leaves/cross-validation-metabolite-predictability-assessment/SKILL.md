---
name: cross-validation-metabolite-predictability-assessment
description: Use when matched multiomic (CCLE-style) feature tables and a metabolite
  abundance matrix are available and the goal is to benchmark how well each omics
  class individually predicts each metabolite via cross-validated ML.
when_to_use_negative:
- 'Predicting metabolomics from non-cell-line sources (e.g., in vivo tissue): models
  trained on cell lines in culture are not directly transferrable due to in vitro/in
  vivo differences and limited feature overlap'
- Feature importance interpretation or network construction — use the separate Shapley
  analysis / Metab8D_network.xlsx workflow instead of the accuracy-benchmarking run
- Input omics have very few matched samples (cell lines); per the paper, omics with
  too few samples may not perform as well and accuracies are not comparable across
  unequal sample sizes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3473
tools:
- name: scikit-learn
  role: Provides the RandomForestRegressor used to train one model per metabolite
    per feature set under 5-fold cross-validation
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Language in which the ML workflow scripts (ML_function.ipynb) are implemented
  repo: https://github.com/sriram-lab/Metab8D
- name: machine learning (random forest regression)
  role: Random forest regression was chosen because it generally outperformed Ridge,
    Lasso, XGBoost, and SVMs in cross-validation benchmarking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:50d319b88da614c19b50a166c6efa8456374b7e34b14e15c42ae8ac4ba4cce5c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-validation-metabolite-predictability-assessment

## Summary

Trains per-metabolite random forest regression models on each multiomic feature set from the CCLE cell line panel and scores predictability of the 225-metabolite panel via cross-validated Pearson correlations, quantifying which omics class (e.g., transcriptomics) best predicts metabolomic variation.

## When to use

Use when matched multiomic (CCLE-style) feature tables and a metabolite abundance matrix are available and the goal is to benchmark how well each omics class individually predicts each metabolite via cross-validated ML.

## When NOT to use

- Predicting metabolomics from non-cell-line sources (e.g., in vivo tissue): models trained on cell lines in culture are not directly transferrable due to in vitro/in vivo differences and limited feature overlap
- Feature importance interpretation or network construction — use the separate Shapley analysis / Metab8D_network.xlsx workflow instead of the accuracy-benchmarking run
- Input omics have very few matched samples (cell lines); per the paper, omics with too few samples may not perform as well and accuracies are not comparable across unequal sample sizes

## Inputs

- CCLE multiomic feature matrices (9 biomolecular classes, excluding mutations), obtained from the DepMap portal
- Matched 225-metabolite relative abundance matrix across CCLE cell lines
- Trained RF models and CCLE training data from the Recon8D Synapse page (optional starting point)

## Outputs

- RF_results.csv: Pearson's correlations and P values for all metabolite models from each of nine feature sets
- Per-metabolite significance calls at the Bonferroni-corrected threshold 0.05/225
- Feature-set ranking of metabolome predictability (cross-validated accuracies per omics class)

## How to apply

Load the 9 CCLE feature sets (copy number variation, DNA methylation, histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics — mutations excluded due to their binary structure) and the matched 225-metabolite matrix; replace NaN values with zeros and remove zero-variance features. For each feature set and each metabolite, train scikit-learn's RandomForestRegressor with 5-fold cross-validation, applying Z-score normalization within each CV fold to avoid leakage. Assess every model with Pearson's correlation between predicted and true metabolite concentrations, and call predictions significant using the Bonferroni-corrected threshold 0.05/225. Compile correlations, P values, and significance calls for all 225 metabolites x 9 feature sets into RF_results.csv, then compare feature sets to rank predictability (transcriptomics, including coding and non-coding RNAs, is expected to win).

## Related tools

- **scikit-learn** (Provides the RandomForestRegressor used to train one model per metabolite per feature set under 5-fold cross-validation) — https://github.com/sriram-lab/Metab8D
- **Python** (Language in which the ML workflow scripts (ML_function.ipynb) are implemented) — https://github.com/sriram-lab/Metab8D
- **machine learning (random forest regression)** (Random forest regression was chosen because it generally outperformed Ridge, Lasso, XGBoost, and SVMs in cross-validation benchmarking)

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rf = RandomForestRegressor(); rf.fit(X_train_z, y_train_z); preds = rf.predict(X_test_z)
```

## Evaluation signals

- RF_results.csv contains entries for all 225 metabolites x 9 feature sets, each with a Pearson correlation and P value
- Every reported P value is assessed against the Bonferroni threshold 0.05/225 for significance
- Cross-validation folds show Z-score normalization fitted within (not across) folds — no data leakage into preprocessing
- Transcriptomics should yield the highest cross-validated correlations across algorithms and validation procedures, with coding and non-coding RNAs among top predictors
- Zero-variance features are absent from every feature matrix before model fitting

## Limitations

- The 225-metabolite panel is only a fraction of metabolites that exist within a cell across distinct cellular compartments
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo; models are not directly transferrable to other sources due to in vitro/in vivo differences and limited feature overlap
- Differences in sample quantities across omics can influence accuracies — omics with too few matched cell lines may underperform for reasons unrelated to biology

## Evidence

- [discussion] Pearson correlation metric: [evidence span withheld — anchor status: not_found]
