---
name: cross-validation-fold-normalization
description: Use when training per-feature-set ML models (e.g., random forest regressors
  on CCLE omics) where scaling must be fit only on training folds of a 5-fold CV scheme
  to avoid leakage into held-out metabolite predictions.
when_to_use_negative:
- Models are not directly transferrable to predicting metabolomics from other sources
  due to differences between in vitro and in vivo conditions and limited feature overlap
  — do not apply CCLE-derived fold scaling statistics to NCI60 or single-cell data
- Binary mutation features were excluded from random forest model results due to their
  binary structure, so Z-score scaling of the mutation feature set is not meaningful
  for the final Metab8D models
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Random forests were trained using Sci-kit Learn's Random Forest Regressor;
    Z-score normalization within each CV fold is applied to the matrices fed to these
    models
- name: Python
  role: Implements the preprocessing and fold-wise normalization pipeline; scripts
    available in the Recon8D GitHub repository
  repo: https://github.com/sriram-lab/Metab8D
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2024.08.17.608400
    title: 'Recon8D: A metabolic regulome network from oct-omics and machine learning'
  source_content_hash: sha256:5964b56bc7985860e12070858e3bf63fe48dbfba7c33cf0aa8497dfc45317a6c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/sriram-lab/Metab8D
  tools:
  - BioTools.fr
  - Python
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-validation-fold-normalization

## Summary

Z-score normalization of CCLE omics feature matrices performed inside each 5-fold cross-validation training split, as used by Metab8D/Recon8D random forest metabolite models to prevent information leakage from held-out folds into scaling parameters.

## When to use

Use when training per-feature-set ML models (e.g., random forest regressors on CCLE omics) where scaling must be fit only on training folds of a 5-fold CV scheme to avoid leakage into held-out metabolite predictions.

## When NOT to use

- Models are not directly transferrable to predicting metabolomics from other sources due to differences between in vitro and in vivo conditions and limited feature overlap — do not apply CCLE-derived fold scaling statistics to NCI60 or single-cell data
- Binary mutation features were excluded from random forest model results due to their binary structure, so Z-score scaling of the mutation feature set is not meaningful for the final Metab8D models

## Inputs

- Raw CCLE omics feature matrices per biomolecular class (CNV, mutations, DNA methylation, histone PTMs, coding transcripts, RNA splicing, miRNA, lncRNA, proteomics, phosphoproteomics) from the DepMap portal
- 5-fold CV fold assignments for ~1000 matched CCLE cancer cell lines
- 225-metabolite target matrix for matched cell lines

## Outputs

- Per-fold Z-score-normalized training and test feature matrices
- Per-class feature counts after zero-variance filtering (e.g., 3,767 lncRNAs, 2,957 splicing events)
- Normalized inputs feeding per-metabolite random forest regression models

## How to apply

First preprocess each raw CCLE omics feature matrix (10 feature sets across 8 biomolecular classes): replace NaNs with zeros, take medians of repeated observations, standardize feature names (e.g., RNA splicing ensemble IDs to common names via BioTools.fr), and remove zero-variance features that carry no predictive signal. Then, for each metabolite model, split the ~1000 matched CCLE cell lines into 5-fold cross-validation folds and compute the Z-score transformation (mean and standard deviation) exclusively on the training portion of each fold, applying the same transformation to the held-out fold before prediction with the Random Forest Regressor. This fold-wise normalization ensures that variance statistics from test samples never inform model training. Repeat for each of the nine feature sets and each metabolite, and record resulting feature counts per class (e.g., 3,767 lncRNAs, 2,957 splicing events, 17,203 mutation features).

## Related tools

- **scikit-learn** (Random forests were trained using Sci-kit Learn's Random Forest Regressor; Z-score normalization within each CV fold is applied to the matrices fed to these models)
- **Python** (Implements the preprocessing and fold-wise normalization pipeline; scripts available in the Recon8D GitHub repository) — https://github.com/sriram-lab/Metab8D

## Examples

```
from sklearn.preprocessing import StandardScaler; scaler = StandardScaler().fit(X_train); X_train_z = scaler.transform(X_train); X_test_z = scaler.transform(X_test)
```

## Evaluation signals

- Feature counts per omics class match reported values (e.g., 3,767 lncRNAs, 2,957 splicing events, 17,203 mutation features) after preprocessing
- Normalization statistics (mean/std) computed on each fold's training split only — no test-fold statistics enter scaling
- Model performance evaluated via Pearson's correlation between predicted and true metabolite concentrations, with significance at Bonferroni-corrected P value (0.05/225)
- Zero-variance features removed before normalization so scaling does not divide by zero
- RF_results files report Pearson's correlations and Bonferroni-corrected P values for all metabolite models from each of nine feature sets

## Limitations

- Normalization must be redone within each fold; normalizing once over the full dataset would leak held-out sample statistics into training
- Differences in sample quantities can influence accuracies, where omics with too few samples (cell lines) may not perform as well
- The metabolite panel of 225 metabolites is only a fraction of those that exist within a cell across compartments, bounding generalization of models trained on these normalized features

## Evidence
