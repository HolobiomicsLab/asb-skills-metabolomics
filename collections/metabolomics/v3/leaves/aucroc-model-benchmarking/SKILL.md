---
name: aucroc-model-benchmarking
description: Use when a SciKitLearn-compatible classifier must be evaluated on pathway-transformed
  multi-omics scores and compared against the random-forest Single-View benchmark
  AUROC of 0.95 on COVID-19 data.
when_to_use_negative:
- The input is still molecular-level abundance data (e.g. raw metabolomics or proteomics
  matrices) rather than an ssPA pathway-transformed score matrix — pathway transformation
  via py-ssPA must come first.
- The task is unsupervised or lacks outcome labels, since Single-View benchmarking
  requires a supervised classification target and held-out AUROC.
- A full head-to-head comparison with conventional pathway analysis approaches (ORA,
  GSEA, MultiGSEA, IMPaLA) is required — the authors state such comparison was beyond
  the scope of the work.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3570
tools:
- name: PathIntegrate
  role: Single-View framework that combines multi-omics ssPA pathway scores into one
    matrix and accepts any SKlearn-compatible predictive model for benchmarking
  repo: https://github.com/cwieder/PathIntegrate
- name: SciKitLearn
  role: Source of the alternative classifiers (LogisticRegression, LinearSVC, RandomForestClassifier)
    and cross-validation utilities used to compute held-out AUROC
- name: py-ssPA
  role: Provides the single-sample pathway analysis (e.g. sspa_kpca) that generates
    the pathway score matrices input to the benchmark model
  repo: https://github.com/cwieder/py-ssPA
- name: PathIntegrate_scripts
  role: Benchmarking scripts and COVID-19 data (Su et al. 2020) needed to replicate
    the published AUC 0.95 random-forest benchmark
  repo: cwieder/PathIntegrate_scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:f1dc21720beb0297b79bd1e9da4ca9cc7e641e74087f0cb7df517e7665797202
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - PathIntegrate
  - PathIntegrate_scripts
  - Python
  - SciKitLearn
  - py-ssPA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3570
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 24
---

# aucroc-model-benchmarking

## Summary

Benchmarks predictive classification performance of PathIntegrate Single-View models on multi-omics ssPA pathway scores by computing cross-validated test AUROC and comparing it to the published random-forest benchmark (AUC 0.95) on the COVID-19 multi-omics data (Su et al. 2020).

## When to use

Use when a SciKitLearn-compatible classifier must be evaluated on pathway-transformed multi-omics scores and compared against the random-forest Single-View benchmark AUROC of 0.95 on COVID-19 data.

## When NOT to use

- The input is still molecular-level abundance data (e.g. raw metabolomics or proteomics matrices) rather than an ssPA pathway-transformed score matrix — pathway transformation via py-ssPA must come first.
- The task is unsupervised or lacks outcome labels, since Single-View benchmarking requires a supervised classification target and held-out AUROC.
- A full head-to-head comparison with conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) is required — the authors state such comparison was beyond the scope of the work.

## Inputs

- ssPA pathway-transformed COVID-19 multi-omics score matrix (samples as rows, multi-omics pathway scores as columns)
- Binary outcome labels (COVID vs non-COVID, or mild vs severe cases) aligned to samples
- SciKitLearn-compatible classifier (e.g. RandomForestClassifier, LogisticRegression, LinearSVC)
- Published benchmark AUC value (random-forest Single-View AUC of 0.95 on COVID-19 multi-omics pathway scores)

## Outputs

- Cross-validated test AUROC on held-out fold predictions
- Ranked table of the most important multi-omics pathways (standardised coefficients or permutation importance)
- CSV file containing the AUROC result and the ranked top-pathway table

## How to apply

Load the ssPA pathway-transformed COVID-19 multi-omics score matrix (samples as rows, pathway scores as columns) and the COVID/non-COVID outcome labels with pandas. PathIntegrate Single-View combines the multi-omics pathway scores into one matrix and accepts any SciKitLearn-compatible predictive model, so instantiate an alternative estimator such as LogisticRegression or LinearSVC. Fit the classifier using repeated k-fold cross-validation to generate out-of-fold test predictions (the paper uses 5-times repeated nested 5-fold cross-validated schemes for parameter tuning). Compute the cross-validated test AUROC from held-out predictions and report it against the published random-forest Single-View benchmark AUC of 0.95 (and the molecular-level model AUC of 0.98) on the same COVID-19 dataset. Extract model-specific feature importance — standardised coefficients for logistic regression or permutation importance for SVM — to produce a ranked multi-omics pathway table, then write the AUROC and ranked top-pathway table to CSV.

## Related tools

- **PathIntegrate** (Single-View framework that combines multi-omics ssPA pathway scores into one matrix and accepts any SKlearn-compatible predictive model for benchmarking) — https://github.com/cwieder/PathIntegrate
- **SciKitLearn** (Source of the alternative classifiers (LogisticRegression, LinearSVC, RandomForestClassifier) and cross-validation utilities used to compute held-out AUROC)
- **py-ssPA** (Provides the single-sample pathway analysis (e.g. sspa_kpca) that generates the pathway score matrices input to the benchmark model) — https://github.com/cwieder/py-ssPA
- **PathIntegrate_scripts** (Benchmarking scripts and COVID-19 data (Su et al. 2020) needed to replicate the published AUC 0.95 random-forest benchmark) — cwieder/PathIntegrate_scripts

## Examples

```
from sklearn.model_selection import cross_val_score, RepeatedKFold; auc = cross_val_score(model, X_pathway_scores, y, cv=RepeatedKFold(n_splits=5, n_repeats=5), scoring='roc_auc')
```

## Evaluation signals

- The output contains a numeric cross-validated AUROC computed from held-out out-of-fold test predictions, not training-set fit.
- The AUROC is reported against the random-forest Single-View benchmark of 0.95 on the same COVID-19 multi-omics data (molecular-level model reference: 0.98).
- The ranked pathway table is non-empty and model-specific (standardised coefficients for logistic regression, permutation importance for SVM).
- Cross-validation scheme matches the paper's repeated k-fold style (e.g. 5-times repeated 5-fold) rather than a single train/test split.
- The AUROC and pathway table are written to CSV in a reproducible pipeline consistent with the cwieder/PathIntegrate_scripts benchmarking scripts.

## Limitations

- PathIntegrate requires all input omics to be measured on the same individuals; samples from individuals without data on all omics must be discarded.
- There is a lack of appropriate benchmarking data for multi-omics integration methods (ideally n >= 1000 with validated molecular/pathway-level signals), so benchmark AUROCs are dataset-specific.
- Pathway coverage (proportion of molecules in a pathway observable in the data) is an inherent bottleneck, particularly for metabolomics, affecting which pathways can be scored.
- The published benchmark is demonstrated with a random-forest model; alternative classifiers are compared informally against that number rather than through a formal statistical test.

## Evidence

- [methods] Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model: "Single-View model: Integrates multi-omics data into one set of multi-omics pathway scores and applies an SKlearn-compatible predictive model"
- [intro] designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation.: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
- [supplementary] 5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View: "5-times repeated nested 5-fold cross-validated results for number of latent variables parameter tuning in PathIntegrate Multi-View"
- [readme] All ssPA methods now have a `fit()`, `transform()` and `fit_transform()` method for compatibility with SciKitLearn.: [evidence span withheld — anchor status: external_doc]
- [readme] Model outputs include multi-omics pathways ranked by their contribution to the outcome prediction, the contribution of each omics layer: "Model outputs include multi-omics pathways ranked by their contribution to the outcome prediction, the contribution of each omics layer"
