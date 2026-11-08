---
name: single-cell-transcriptomics-predictive-modeling
description: Use when you have a trained multi-omics predictor's ranked top features
  and need to check whether those features show elevated feature importance in an
  independent dataset (bulk NCI60 or single-cell transcriptomics) beyond chance.
when_to_use_negative:
- The trained models are not directly transferable to predicting metabolomics from
  other sources such as in vivo conditions or different cell line panels — this skill
  is a validation of feature importance enrichment, not direct metabolite prediction.
- The metabolite panel of 225 metabolites is only a fraction of those in a cell; if
  your question concerns metabolites outside this panel or the targeted 90 phosphoproteins,
  this skill's validation scope does not cover it.
- '''Guilt by association'' network inference may not identify causative mechanisms
  — do not use this skill to claim causal metabolic regulation, only predictive transferability.'
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0102
tools:
- name: scikit-learn
  role: Random forests are trained using Sci-kit Learn's Random Forest Regressor algorithm
    to produce per-metabolite models whose feature importances are compared across
    datasets.
- name: machine learning (random forest regression)
  role: Random forest regression ML algorithms generate metabolomic predictions and
    feature importances for validation in NCI60 and single-cell data.
- name: Python
  role: Implementation language for the validation scripts available in the Recon8D
    GitHub repository.
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:781810623bd59900712658a65802f26c21b6fcbd000e6c82bb41454c8e49af3c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - Python
  - machine learning (random forest regression)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-transcriptomics-predictive-modeling

## Summary

Generalization-validation skill from Recon8D: take the top features isolated from CCLE-trained random forest metabolite models and test whether their feature importances are significantly enriched in independent NCI60 bulk and single-cell cell-cycle transcriptomics data.

## When to use

Use when you have a trained multi-omics predictor's ranked top features and need to check whether those features show elevated feature importance in an independent dataset (bulk NCI60 or single-cell transcriptomics) beyond chance.

## When NOT to use

- The trained models are not directly transferable to predicting metabolomics from other sources such as in vivo conditions or different cell line panels — this skill is a validation of feature importance enrichment, not direct metabolite prediction.
- The metabolite panel of 225 metabolites is only a fraction of those in a cell; if your question concerns metabolites outside this panel or the targeted 90 phosphoproteins, this skill's validation scope does not cover it.
- 'Guilt by association' network inference may not identify causative mechanisms — do not use this skill to claim causal metabolic regulation, only predictive transferability.

## Inputs

- NCI60 omics data from the NCI CellMiner portal
- single-cell cell-cycle transcriptomics data
- list of top features isolated from CCLE-trained random forest metabolite models
- full feature set from the independent dataset

## Outputs

- pooled feature importance distributions for top features vs. full feature set
- t-test statistics and p-values comparing importance distributions
- KS-test statistics and p-values comparing importance distributions
- tables replicating Supplementary Tables 3-5

## How to apply

Load the independent NCI60 omics data from the NCI CellMiner portal or the single-cell cell-cycle transcriptomics data, replacing missing NaN values with zeros and removing zero-variance features. For each metabolite, train a separate scikit-learn Random Forest Regressor with 5-fold cross-validation, Z-score normalizing the data within each CV fold to avoid leakage. Extract feature importances from each trained model, then pool the importances for the CCLE-identified top features versus the full feature set. Apply both a t-test and a Kolmogorov–Smirnov test to the pooled importance distributions to test whether top features show significantly higher importance than expected by chance. The rationale is that a feature's predictive utility should transfer across independent datasets if the multiomic relationship is real, mirroring how the original CCLE models treated each metabolite separately.

## Related tools

- **scikit-learn** (Random forests are trained using Sci-kit Learn's Random Forest Regressor algorithm to produce per-metabolite models whose feature importances are compared across datasets.)
- **machine learning (random forest regression)** (Random forest regression ML algorithms generate metabolomic predictions and feature importances for validation in NCI60 and single-cell data.)
- **Python** (Implementation language for the validation scripts available in the Recon8D GitHub repository.) — https://github.com/sriram-lab/Metab8D

## Examples

```
from sklearn.ensemble import RandomForestRegressor; model = RandomForestRegressor(); model.fit(X_train, y_train); importances = model.feature_importances_
```

## Evaluation signals

- CCLE-identified top features should show significantly higher feature importance scores in the NCI60 data than the whole feature set (t-test and KS test p-values < 0.05).
- CCLE-identified top features should show significantly higher feature importance scores in the single-cell transcriptomics data than expected by random chance.
- Each metabolite model should be evaluated independently with 5-fold cross-validation and Z-score normalization applied within each fold, not on the full dataset.
- Test statistics and p-values should be reported in tables replicating Supplementary Tables 3-5.
- Data preprocessing should replace NaN with zeros and remove features with 0 variance before model training.

## Limitations

- Models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions, cell line panels, and limited feature overlap.
- The analysis was done in cancer cell lines in culture and may not reflect metabolism in vivo.
- Differences in sample quantities can influence accuracies; omics with too few samples may not perform as well.
- Network associations from systems-level data are descriptive and not necessarily mechanistic.

## Evidence
