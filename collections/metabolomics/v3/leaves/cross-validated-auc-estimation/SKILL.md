---
name: cross-validated-auc-estimation
description: Use when fitting a supervised multi-omics model (e.g. MB-PLS with latent
  variables) and you need a cross-validated AUC estimate plus tuned hyperparameters
  like the optimal number of latent variables.
when_to_use_negative:
- The data has not been split into train/test structure compatible with nested CV,
  e.g. PathIntegrate requires all input omics to be measured on the same individuals,
  so samples missing any omics must be discarded before CV splitting
- The goal is unsupervised exploratory modelling with no outcome variable, since AUC
  estimation requires a class label
- Comparing molecular vs pathway-level signal detection on semi-synthetic data where
  pathway transformation itself is the object of study rather than predictive performance
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0128
tools:
- name: PathIntegrate
  role: Multi-View MB-PLS framework whose models are evaluated by 5-times repeated
    nested 5-fold CV to tune latent variables and estimate AUC
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Cross-validation and model optimisation engine compatible with PathIntegrate
    for nested CV parameter tuning
- name: Python
  role: Language in which the cross-validation pipeline and comparison scripts are
    implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:af298377f7544861b85de56b5e66f673699d913aa251b7adfd6c8d9836231e40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# cross-validated-auc-estimation

## Summary

Estimate predictive performance (AUC) of multi-omics MB-PLS models using 5-times repeated nested 5-fold cross-validation, which simultaneously tunes the number of latent variables and yields an unbiased generalisation estimate. Used in PathIntegrate to compare molecular-level vs pathway-level models on COPDgene data.

## When to use

Use when fitting a supervised multi-omics model (e.g. MB-PLS with latent variables) and you need a cross-validated AUC estimate plus tuned hyperparameters like the optimal number of latent variables.

## When NOT to use

- The data has not been split into train/test structure compatible with nested CV, e.g. PathIntegrate requires all input omics to be measured on the same individuals, so samples missing any omics must be discarded before CV splitting
- The goal is unsupervised exploratory modelling with no outcome variable, since AUC estimation requires a class label
- Comparing molecular vs pathway-level signal detection on semi-synthetic data where pathway transformation itself is the object of study rather than predictive performance

## Inputs

- Sample-by-molecule multi-omics data matrix (e.g. COPDgene plasma data, samples as rows, molecule IDs as columns), log-transformed and z-score standardised
- Binary outcome labels (e.g. COPD status) aligned to sample rows
- Model specification with hyperparameter grid for number of latent variables

## Outputs

- Average cross-validated AUC with standard deviation (e.g. 0.70 +/- 0.02)
- Optimal number of latent variables selected by nested CV (e.g. 4 vs 6)
- Comparison table such as copdgene_molecular_vs_pathway_mbpls_performance.csv

## How to apply

Wrap the model fitting in a 5-times repeated nested 5-fold cross-validation scheme: the inner loop tunes the number of latent variables while the outer loop estimates generalisation AUC. PathIntegrate is designed to be compatible with SciKitLearn, so GridSearchCV-style nested CV can drive the parameter tuning of the MB-PLS classifier. Fit the model (molecular-level MB-PLS on raw molecule abundances, or pathway-level MB-PLS on ssPA pathway-transformed data) within each outer fold using the inner-fold-optimal latent variable count, then aggregate outer-fold AUCs into an average with a standard deviation (expected magnitude on COPDgene: AUC 0.70 +/- 0.02). Record both the average cross-validated AUC and the selected number of latent variables for each competing model so complexity (4 vs 6 latent variables) can be compared alongside discrimination.

## Related tools

- **PathIntegrate** (Multi-View MB-PLS framework whose models are evaluated by 5-times repeated nested 5-fold CV to tune latent variables and estimate AUC) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Cross-validation and model optimisation engine compatible with PathIntegrate for nested CV parameter tuning)
- **Python** (Language in which the cross-validation pipeline and comparison scripts are implemented)

## Examples

```
from sklearn.model_selection import GridSearchCV, RepeatedKFold; cv = RepeatedKFold(n_splits=5, n_repeats=5); gs = GridSearchCV(mbpls, {'n_components': [2,3,4,5,6]}, cv=cv)
```

## Evaluation signals

- The number of latent variables is selected by inner-loop tuning only, never using outer-fold test data
- Reported AUC carries a dispersion measure (e.g. 0.70 +/- 0.02) computed across the 5 repeats
- Models being compared use the identical CV scheme so AUC differences and latent-variable counts (4 vs 6) are directly comparable
- Output CSV (copdgene_molecular_vs_pathway_mbpls_performance.csv) contains both average AUC and optimal latent variable count per model
- AUC values fall in a plausible range around 0.70 for the COPDgene plasma multi-omics task, not near-degenerate (~0.5) or suspiciously perfect (~1.0)

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded, reducing the CV sample pool
- It remains a challenge to compare molecular vs pathway-level models because the number of differentially abundant molecules per pathway and pathway definitions/sizes vary between databases
- Lack of appropriate benchmarking data limits external validation of cross-validated AUC estimates

## Evidence

- [supplementary] 5-times repeated nested 5-fold cross-validation for latent variable tuning: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [full_text] The molecular-level MB-PLS model achieved the same average AUC of 0.70 (+/-0.02) as the pathway-level model: [evidence span withheld — anchor status: not_found]
- [intro] PathIntegrate is compatible with SciKitLearn for model optimisation and evaluation: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
- [readme] Log transform and standardise multi-omics data before PathIntegrate input: "Log transform the data
log2_mat = np.log2(imputed_mat)

# Standardise the data (metabolite values) using z-score (mean=0, std=1)"
