---
name: cross-validated-latent-variable-selection
description: Use when fitting a PathIntegrate Multi-View MB-PLS (or comparable PLS-based
  multi-omics model) and the latent-variable count must be tuned to avoid over- or
  under-fitting before pathway importance is interpreted.
when_to_use_negative:
- The task is a purely unsupervised decomposition with no outcome variable, so AUC-based
  tuning is undefined.
- The downstream model is a Single-View SciKitLearn classifier with its own hyperparameters,
  not the multi-block PLS latent-structure model.
- Sample size is too small for repeated nested 5-fold CV splits to stratify the outcome
  reliably.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PathIntegrate
  role: Provides the Multi-View MB-PLS model whose latent-variable count is tuned
    by cross-validation
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Supplies GridSearchCV/Pipeline-compatible repeated nested cross-validation
    used to select the number of latent variables
- name: sspa
  role: Produces the pathway-transformed views (ssPA kPCA scores) that are the CV
    input matrices
  repo: https://github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:351faaa418fae570e76237699b1b3a9a3cf8c405e9abb42753ccb7a7fda1f4c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - sspa
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# cross-validated-latent-variable-selection

## Summary

Selects the optimal number of latent variables for multi-block PLS (MB-PLS) models by 5-times repeated nested 5-fold cross-validation, choosing the setting that maximises cross-validated AUC. In the COPDgene PathIntegrate Multi-View application this identified 4 latent variables as optimal, with a mean cross-validated AUC of 0.70.

## When to use

Use when fitting a PathIntegrate Multi-View MB-PLS (or comparable PLS-based multi-omics model) and the latent-variable count must be tuned to avoid over- or under-fitting before pathway importance is interpreted.

## When NOT to use

- The task is a purely unsupervised decomposition with no outcome variable, so AUC-based tuning is undefined.
- The downstream model is a Single-View SciKitLearn classifier with its own hyperparameters, not the multi-block PLS latent-structure model.
- Sample size is too small for repeated nested 5-fold CV splits to stratify the outcome reliably.

## Inputs

- Pathway-transformed (ssPA kPCA) multi-omics matrices (samples x Reactome pathway scores) for metabolomics, proteomics and transcriptomics
- Outcome labels (e.g. COPD case/control status) aligned to sample rows
- Grid of candidate latent-variable counts for MB-PLS

## Outputs

- Optimal number of latent variables (expected: 4 for the COPDgene Multi-View model)
- Cross-validated performance metrics per setting (mean AUC; expected optimum 0.70)
- Tuned MB-PLS model ready for MB-VIP importance and permutation testing

## How to apply

Wrap the MB-PLS model in a SciKitLearn-compatible parameter-tuning loop (e.g. GridSearchCV) because PathIntegrate and ssPA components expose fit/transform/fit_transform for SciKitLearn Pipeline and GridSearchCV use. Sweep the number of latent variables as the tuning parameter, evaluating each setting with 5-times repeated nested 5-fold cross-validation on the pathway-transformed (ssPA kPCA) multi-omics views. Select the number of latent variables with the best cross-validated AUC; for the COPDgene metabolomics/proteomics/transcriptomics case this was 4 latent variables at mean AUC 0.70. Fix the selected setting before computing block-wise MB-VIP pathway importances, so that importance statistics and their permutation p-values (e.g. carnitine metabolism, metabolomics p = 0.003) refer to the tuned model. Optionally accompany tuning with 100 bootstrap resamples to estimate stability of y-variance explained per latent variable (metabolomics: 6-17%).

## Related tools

- **PathIntegrate** (Provides the Multi-View MB-PLS model whose latent-variable count is tuned by cross-validation) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Supplies GridSearchCV/Pipeline-compatible repeated nested cross-validation used to select the number of latent variables)
- **sspa** (Produces the pathway-transformed views (ssPA kPCA scores) that are the CV input matrices) — https://github.com/cwieder/py-ssPA

## Examples

```
from sklearn.model_selection import GridSearchCV; gs = GridSearchCV(model, param_grid={'n_components': [2, 3, 4, 5, 6]}, scoring='roc_auc', cv=RepeatedStratifiedKFold(n_splits=5, n_repeats=5))
```

## Evaluation signals

- Tuned optimum should be 4 latent variables with mean cross-validated AUC of 0.70 on the COPDgene Multi-View model.
- Nested CV structure must be 5-times repeated 5-fold, i.e. outer/inner split separation identical to the manuscript's protocol.
- Latent-variable count must be fixed before MB-VIP permutation testing; reported p-values (e.g. 0.003) only hold for the tuned model.
- Bootstrap y-variance explained per latent variable should fall in the 6-17% range for the metabolomics view.
- Tuning must run without leakage of pathway-transformation parameters into test folds (ssPA kPCA fit on training folds only).

## Limitations

- No appropriate large benchmarking dataset was available; ideally benchmarking data would have two or more high-quality omics views, n >= 1000, and validated pathway-level signals.
- PathIntegrate requires all input omics measured on the same individuals; samples missing any omics must be discarded, which constrains CV sample availability.
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, affecting the features available for CV fitting.

## Evidence

- [supplementary] 5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [intro] designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation.: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
- [readme] This allows integration of ssPA transformation with various machine learning functions in SKLearn such as `Pipeline` and `GridSearchCV`.: [evidence span withheld — anchor status: external_doc]
- [intro] PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets: "PathIntegrate Multi-View, which uses a multi-block partial least regression (MB-PLS) model to model interactions between pathway-transformed omics datasets"
