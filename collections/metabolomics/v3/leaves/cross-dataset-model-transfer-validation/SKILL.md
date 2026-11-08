---
name: cross-dataset-model-transfer-validation
description: Use when ML models trained on one cell-line panel (e.g. CCLE) have produced
  top features, and you need to confirm those features generalize by measuring their
  importance in independent datasets such as NCI60 or single-cell data.
when_to_use_negative:
- When the goal is direct prediction of metabolites from another data source — the
  article notes models are 'not directly transferrable' due to differences between
  in vitro and in vivo conditions, cell line panels, and limited feature overlap
- When you only need model performance (Pearson's correlation) on the training data
  rather than generalization of feature importance in independent data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides Random Forest Regressor used to train per-metabolite models on the
    independent datasets and extract feature importances
- name: machine learning (random forest regression)
  role: Core modeling method whose feature importances are the basis for the cross-dataset
    comparison
- name: NCI CellMiner portal
  role: Source of the independent NCI60 validation data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:3651e2c2633c25c524cc4fdbbe487b65572d763942748b998b1b6d215e1cf201
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - NCI CellMiner portal
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-dataset-model-transfer-validation

## Summary

Validates that features selected by machine-learning models trained on one omics dataset (e.g. CCLE-trained random forest metabolite models) are genuinely informative by testing whether they show significantly higher feature importance in independent datasets (NCI60, single-cell transcriptomics) than expected by chance.

## When to use

Use when ML models trained on one cell-line panel (e.g. CCLE) have produced top features, and you need to confirm those features generalize by measuring their importance in independent datasets such as NCI60 or single-cell data.

## When NOT to use

- When the goal is direct prediction of metabolites from another data source — the article notes models are 'not directly transferrable' due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap
- When you only need model performance (Pearson's correlation) on the training data rather than generalization of feature importance in independent data

## Inputs

- NCI60 omics data from the NCI CellMiner portal
- single-cell cell-cycle transcriptomics data
- CCLE-trained top feature lists (top 20 features per metabolite model from Metab8D_network.xlsx)

## Outputs

- Pooled feature importance distributions for CCLE top features vs. full feature set
- t-test and KS test statistics with p-values per metabolite
- Tables replicating Supplementary Tables 3–5

## How to apply

First load the independent omics data — NCI60 from the NCI CellMiner portal, or single-cell cell-cycle transcriptomics — replacing missing NaN values with zeros and removing zero-variance features, mirroring the CCLE preprocessing. Train a separate random forest regression model (scikit-learn RandomForestRegressor, 5-fold cross-validation with Z-score normalization within each fold) for each metabolite in the new dataset, then extract feature importances from each trained model. Pool the importances of the CCLE-identified top features separately from the full feature set, and apply a t-test and a Kolmogorov–Smirnov test to compare the two distributions. Report test statistics and p-values in tables replicating Supplementary Tables 3–5; significance demonstrates the top features carry information beyond random chance in the independent setting.

## Related tools

- **scikit-learn** (Provides Random Forest Regressor used to train per-metabolite models on the independent datasets and extract feature importances)
- **machine learning (random forest regression)** (Core modeling method whose feature importances are the basis for the cross-dataset comparison)
- **NCI CellMiner portal** (Source of the independent NCI60 validation data)

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rf = RandomForestRegressor(); rf.fit(X_train, y_train); importances = rf.feature_importances_
```

## Evaluation signals

- Top features from CCLE-trained models show significantly higher feature importance scores in the NCI60 data than the whole feature set (t-test and KS test p-values)
- CCLE-identified features show significantly higher feature importance scores in the single-cell transcriptomics data than expected out of random chance
- Missing values replaced with zeros and zero-variance features removed in the validation datasets, matching the CCLE preprocessing
- Each metabolite model uses 5-fold cross-validation with Z-score normalization within each fold
- Reported statistics replicate the structure of Supplementary Tables 3–5

## Limitations

- Models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap
- Analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo
- Network associations from systems-level data are descriptive and not necessarily mechanistic

## Evidence

- [discussion] Random forests were trained using Sci-kit Learn's Random Forest Regressor algorithm with the following parameters: [evidence span withheld — anchor status: not_found]
