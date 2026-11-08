---
name: multiomic-feature-importance-comparison
description: Use when top features from models trained on one dataset (e.g. CCLE-trained
  metabolite predictors) must be validated in an independent dataset by comparing
  their feature-importance distribution against that of the full feature set.
when_to_use_negative:
- When the goal is predictor discovery or model training on the discovery dataset
  itself rather than validating already-identified top features in independent data
- When the models need to predict metabolite values in vivo or from other sample sources
  — the article notes models are not directly transferrable due to in vitro/in vivo
  differences and limited feature overlap
- When causal regulatory mechanisms are required; the guilt-by-association importance
  comparison is descriptive, not mechanistic
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn Random Forest Regressor
  role: Trains a separate random forest regression model per metabolite in the independent
    dataset and extracts feature importances
- name: Python
  role: Implementation language for the validation workflow
  repo: https://github.com/sriram-lab/Metab8D
- name: NCI CellMiner portal
  role: Source of the NCI60 validation omics data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:c7fe6fcea28b1fb53e953da11799820835ba440edd4dde8cc1334ecbf57d572e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - NCI CellMiner portal
  - Python
  - scikit-learn
  - scikit-learn Random Forest Regressor
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multiomic-feature-importance-comparison

## Summary

A generalization-validation skill that tests whether top features identified from CCLE-trained random forest metabolite models show significantly higher feature importance in independent datasets (NCI60, single-cell cell-cycle transcriptomics) than the full feature set, using pooled importance distributions compared by t-test and KS test.

## When to use

Use when top features from models trained on one dataset (e.g. CCLE-trained metabolite predictors) must be validated in an independent dataset by comparing their feature-importance distribution against that of the full feature set.

## When NOT to use

- When the goal is predictor discovery or model training on the discovery dataset itself rather than validating already-identified top features in independent data
- When the models need to predict metabolite values in vivo or from other sample sources — the article notes models are not directly transferrable due to in vitro/in vivo differences and limited feature overlap
- When causal regulatory mechanisms are required; the guilt-by-association importance comparison is descriptive, not mechanistic

## Inputs

- Independent omics feature tables (NCI60 data from the NCI CellMiner portal; single-cell cell-cycle transcriptomics data)
- List of CCLE-identified top features per metabolite (e.g. from Metab8D_network.xlsx top-20 features with confidence scores)
- Metabolite list (225-metabolite panel) defining one model per metabolite

## Outputs

- Per-model random forest feature importance scores for each metabolite in the independent dataset
- Pooled feature-importance distributions for the top-feature group and the full feature set
- T-test and KS test statistics with p-values reported in tables replicating Supplementary Tables 3-5

## How to apply

Load the independent omics data (NCI60 from the NCI CellMiner portal, or single-cell cell-cycle transcriptomics), replace missing NaN values with zeros, and remove zero-variance features. Train a separate scikit-learn Random Forest Regressor for each metabolite in the independent dataset using 5-fold cross-validation with Z-score normalization computed within each CV fold. Extract feature importances from each trained model, then pool the importance scores for the CCLE-identified top features and for the full feature set across models. Apply a t-test and a Kolmogorov-Smirnov (KS) test to the two pooled importance distributions; significantly higher importance for the top-feature group (report test statistics and p-values in tables replicating Supplementary Tables 3-5) demonstrates that CCLE-derived feature-metabolite relationships generalize beyond the training panel.

## Related tools

- **scikit-learn Random Forest Regressor** (Trains a separate random forest regression model per metabolite in the independent dataset and extracts feature importances)
- **Python** (Implementation language for the validation workflow) — https://github.com/sriram-lab/Metab8D
- **NCI CellMiner portal** (Source of the NCI60 validation omics data)

## Examples

```
from sklearn.ensemble import RandomForestRegressor; rf = RandomForestRegressor(); rf.fit(X_train_z, y_train); importances = rf.feature_importances_  # per metabolite, 5-fold CV, Z-score within fold
```

## Evaluation signals

- Top-feature group's pooled importance distribution must be significantly higher than the full feature set by both t-test and KS test (significance reported in Supplementary Tables 3-5)
- NCI60 and single-cell data preprocessing must show no remaining NaNs and no zero-variance features after filtering
- Cross-validation must use Z-score normalization computed within each CV fold, not on the full dataset before splitting
- A separate random forest model must exist for each metabolite (225-metabolite panel) in the validation dataset
- Reported test statistics and p-values should match the structure of Supplementary Tables 3-5

## Limitations

- Models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap
- Validation depends on feature overlap between CCLE-trained model features and the independent dataset's features
- Significance of importance differences is a descriptive, 'guilt by association' generalization check and does not establish causative mechanisms

## Evidence

- [discussion] Random forests were trained using Sci-kit Learn's Random Forest Regressor algorithm: [evidence span withheld — anchor status: not_found]
