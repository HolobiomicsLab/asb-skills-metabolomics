---
name: sklearn-predictive-model-cross-validation
description: Use when a pathway-transformed (ssPA) multi-omics matrix with outcome
  labels is available and a cross-validated classification performance metric (AUROC/AUC)
  plus a ranked pathway importance list is required from an SKlearn-compatible model.
when_to_use_negative:
- Input is molecular-level abundance data not yet transformed by single-sample pathway
  analysis (ssPA) — PathIntegrate Single-View requires pathway-level scores as input
- Samples are missing data on one or more omics layers — PathIntegrate requires all
  input omics measured on the same individuals and does not support entire rows of
  missing data
- The goal is to model interactions between omics datasets via a shared latent space
  — use PathIntegrate Multi-View (MB-PLS) instead of Single-View
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: PathIntegrate
  role: Provides the Single-View framework that combines multi-omics pathway scores
    into one matrix and applies the SKlearn-compatible predictive model
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Supplies the predictive classifier (LogisticRegression, LinearSVC, RandomForest),
    repeated k-fold cross-validation utilities, and model-evaluation functions
- name: py-ssPA
  role: Generates the ssPA pathway-score input matrix that the classifier is fit on
  repo: https://github.com/cwieder/py-ssPA
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:24a4ef48a3bee8263eaf80d3268214e258ea1afcc99a92d0fa7632f67b079908
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - Python
  - SciKitLearn
  - py-ssPA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# sklearn-predictive-model-cross-validation

## Summary

Applies any SciKitLearn-compatible predictive classifier (e.g. LogisticRegression, LinearSVC, RandomForest) to a multi-omics pathway-score matrix within the PathIntegrate Single-View framework, using repeated k-fold cross-validation to obtain an unbiased out-of-fold AUROC benchmarked against the published random-forest result of 0.95 on COVID-19 data.

## When to use

Use when a pathway-transformed (ssPA) multi-omics matrix with outcome labels is available and a cross-validated classification performance metric (AUROC/AUC) plus a ranked pathway importance list is required from an SKlearn-compatible model.

## When NOT to use

- Input is molecular-level abundance data not yet transformed by single-sample pathway analysis (ssPA) — PathIntegrate Single-View requires pathway-level scores as input
- Samples are missing data on one or more omics layers — PathIntegrate requires all input omics measured on the same individuals and does not support entire rows of missing data
- The goal is to model interactions between omics datasets via a shared latent space — use PathIntegrate Multi-View (MB-PLS) instead of Single-View

## Inputs

- ssPA pathway-transformed multi-omics score matrix (pandas DataFrame, samples as rows, pathway scores as columns)
- Binary or categorical outcome labels aligned to samples (e.g. COVID vs non-COVID, mild vs severe)
- SciKitLearn-compatible estimator (e.g. LogisticRegression, LinearSVC, RandomForestClassifier)

## Outputs

- Cross-validated test AUROC computed from held-out out-of-fold predictions
- Ranked table of the most important multi-omics pathways (model-specific feature importance)
- CSV file containing the AUROC result and the ranked top-pathway table

## How to apply

Load the ssPA pathway-transformed multi-omics score matrix (samples as rows, multi-omics pathway scores as columns) and the outcome labels (e.g. COVID/non-COVID, or mild vs. severe cases) with pandas. Initialise the PathIntegrate Single-View model, which combines the multi-omics pathway scores into one matrix and accepts any SKlearn-compatible predictive model — instantiate, for example, sklearn's LogisticRegression or LinearSVC (the published benchmark used a random forest). Fit the classifier with repeated k-fold cross-validation to generate out-of-fold test predictions, then compute the cross-validated test AUROC from the held-out predictions. Extract model-specific feature importance (standardised coefficients for logistic regression, permutation importance for SVM) to rank the most important multi-omics pathways. Report the AUROC against the random-forest Single-View benchmark of 0.95 (molecular-level model: 0.98) achieved on the COVID-19 multi-omics data.

## Related tools

- **PathIntegrate** (Provides the Single-View framework that combines multi-omics pathway scores into one matrix and applies the SKlearn-compatible predictive model) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Supplies the predictive classifier (LogisticRegression, LinearSVC, RandomForest), repeated k-fold cross-validation utilities, and model-evaluation functions)
- **py-ssPA** (Generates the ssPA pathway-score input matrix that the classifier is fit on) — https://github.com/cwieder/py-ssPA

## Examples

```
from sklearn.linear_model import LogisticRegression; from sklearn.model_selection import RepeatedKFold; model = LogisticRegression(); cv = RepeatedKFold(n_splits=5, n_repeats=5)
```

## Evaluation signals

- The output contains a numeric cross-validated AUROC computed strictly from held-out (out-of-fold) test predictions, never training-set scores
- The AUROC is reported against the random-forest Single-View benchmark of 0.95 on the same COVID-19 multi-omics dataset (molecular-level model: 0.98)
- The ranked pathway importance table is non-empty and uses model-appropriate importance (standardised coefficients for logistic regression, permutation importance for SVM)
- Cross-validation is repeated k-fold (e.g. 5-times repeated), matching the manuscript's repeated cross-validated evaluation scheme
- Both the AUROC result and the ranked top-pathway table are written to CSV

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded
- Comparing molecular vs. pathway-level models is challenging because it is unknown how many molecules in a pathway are differentially abundant at any one time
- The AUROC benchmark (0.95) is specific to the COVID-19 multi-omics dataset (Su et al. 2020); other datasets will require their own benchmark or baseline model

## Evidence

- [methods] Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model: "Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model"
- [intro] designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation.: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
- [abstract] PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level: "PathIntegrate employs single-sample pathway analysis to transform multi-omics datasets from the molecular to the pathway-level"
- [readme] All ssPA methods now have a `fit()`, `transform()` and `fit_transform()` method for compatibility with SciKitLearn.: [evidence span withheld — anchor status: external_doc]
- [supplementary] 5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
