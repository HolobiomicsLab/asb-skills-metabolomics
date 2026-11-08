---
name: random-forest-regression-model-training
description: Use when you have matched multi-omics and metabolite abundance matrices
  across samples (e.g. CCLE cancer cell lines) and need cross-validated, per-target
  regression to identify which feature set most accurately predicts each metabolite.
when_to_use_negative:
- The candidate input features are genomic mutations — these are excluded from random
  forest model results due to their binary structure
- The goal is causal mechanism identification — this 'guilt by association' regression
  approach yields descriptive associations that are not necessarily mechanistic
- The omics modality has too few matched samples, since differences in sample quantities
  can influence accuracies
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3172
tools:
- name: scikit-learn
  role: Provides the RandomForestRegressor used to train one model per metabolite
    per feature set with 5-fold CV
  repo: https://github.com/sriram-lab/Metab8D
- name: Python
  role: Implementation language for the ML workflow (ML_function.ipynb) and preprocessing
    scripts
  repo: https://github.com/sriram-lab/Metab8D
- name: XGBoost
  role: Benchmarking alternative to random forests (alongside Ridge, Lasso, SVMs)
    in the ML_function.ipynb script
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:b10e48532c250cbdc92541c9fd2de102d38bf9406d0c22563513ee05a9064e25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - XGBoost
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# random-forest-regression-model-training

## Summary

Trains a separate scikit-learn RandomForestRegressor for each metabolite (225 metabolites) from each omics feature set (9 classes from the CCLE), then scores predictions by Pearson's correlation to rank which omics classes best predict metabolomic variation.

## When to use

Use when you have matched multi-omics and metabolite abundance matrices across samples (e.g. CCLE cancer cell lines) and need cross-validated, per-target regression to identify which feature set most accurately predicts each metabolite.

## When NOT to use

- The candidate input features are genomic mutations — these are excluded from random forest model results due to their binary structure
- The goal is causal mechanism identification — this 'guilt by association' regression approach yields descriptive associations that are not necessarily mechanistic
- The omics modality has too few matched samples, since differences in sample quantities can influence accuracies

## Inputs

- CCLE multi-omics feature matrices (CNV, DNA methylation, histone PTMs, coding RNA, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics) from the DepMap portal
- Matched 225-metabolite abundance matrix for the same cell lines
- Metab8D example_datasets (metabolomics and histone PTM, original and zero-imputed) for testing the code

## Outputs

- RF_results: Pearson's correlations and P values for all metabolite models from each of nine feature sets, with Bonferroni-corrected significance calls
- Metab8D_network.xlsx: top 20 features per metabolite per feature set (2,025 models) with confidence scores 0–8
- Per-metabolite Shapley feature importance rankings for select metabolites

## How to apply

Load each CCLE omics feature set (copy number variation, DNA methylation, histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics — mutations excluded for their binary structure) alongside the matched 225-metabolite abundance matrix. Replace NaN values with zeros and remove features with 0 variance before training. For each metabolite, train a scikit-learn RandomForestRegressor with 5-fold cross-validation, applying Z-score normalization within each CV fold to avoid leakage. Assess performance via Pearson's correlation between predicted and true metabolite concentrations, and call predictions significant at the Bonferroni-corrected threshold 0.05/225. Extract Shapley feature importances for select metabolites and compile per-feature-set results (RF_results.csv; top-20 features with confidence scores in Metab8D_network.xlsx) to compare feature sets — transcriptomics was expected to outperform other omics classes.

## Related tools

- **scikit-learn** (Provides the RandomForestRegressor used to train one model per metabolite per feature set with 5-fold CV) — https://github.com/sriram-lab/Metab8D
- **Python** (Implementation language for the ML workflow (ML_function.ipynb) and preprocessing scripts) — https://github.com/sriram-lab/Metab8D
- **XGBoost** (Benchmarking alternative to random forests (alongside Ridge, Lasso, SVMs) in the ML_function.ipynb script) — https://github.com/sriram-lab/Metab8D

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rf = RandomForestRegressor(); rf.fit(X_train, y_met); r = scipy.stats.pearsonr(rf.predict(X_test), y_met)
```

## Evaluation signals

- RF_results contains entries for exactly 225 metabolites × 9 feature sets (2,025 models) with correlations, P values, and significance flags
- Significance calls match the Bonferroni threshold 0.05/225 — no metabolite is called significant without meeting it
- Z-score normalization statistics computed within each CV fold, not on the full dataset before splitting
- Zero-variance features and NaNs are removed/imputed (zeros) before fitting; no NaNs reach the regressor
- Reproduced ranking should confirm transcriptomics (coding and non-coding RNAs) as the most accurate feature set across algorithms

## Limitations

- Models trained on CCLE cell lines in culture are not directly transferrable to predicting metabolomics from other sources (in vitro vs in vivo differences, cell line panels, limited feature overlap)
- The 225-metabolite panel is only a fraction of metabolites that exist within a cell across compartments; the 90 targeted phosphoproteins do not represent the entire cellular phosphoprotein array
- Network associations from this regression strategy are descriptive ('guilt by association') and not necessarily mechanistic; experimental validation would require an immense effort

## Evidence

- [discussion] Random forests were trained using Sci-kit Learn's Random Forest Regressor algorithm with the following parameters: [evidence span withheld — anchor status: not_found]
