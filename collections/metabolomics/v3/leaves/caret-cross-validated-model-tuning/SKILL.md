---
name: caret-cross-validated-model-tuning
description: Use when building a classification model from metabolomics PDS score
  features and you need to compare multiple ML algorithms under caret-based hyperparameter
  tuning with n-fold cross-validation and held-out test metrics.
when_to_use_negative:
- Input is still a raw metabolite-by-sample measurement matrix that has not been transformed
  to a PDS score matrix by lilikoi.get_pd_scores
- Significant pathway features have not yet been selected (no lilikoi.select_pathways
  output, e.g. threshold = 0.42, method = "gain")
- The task is unsupervised or requires automatic algorithm selection — the paper notes
  AutoML for automatic classification algorithm selection is only planned for the
  future
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: lilikoi
  role: Provides lilikoi.machine_learning, the wrapper that trains the seven classifiers
    on selected pathway features
  repo: https://github.com/lanagarmire/lilikoi
- name: caret
  role: Drives hyperparameter tuning and 10-fold cross-validation of the classification
    models
- name: pROC
  role: Computes AUC for each algorithm's held-out test predictions
- name: randomForest
  role: Backend for the RF classifier among the seven tuned algorithms
- name: glmnet
  role: Backend for the LOG (logistic/regularized) classifier among the seven tuned
    algorithms
- name: gbm
  role: Backend for the GBM classifier among the seven tuned algorithms
- name: e1071
  role: Backend for the SVM classifier among the seven tuned algorithms
- name: pamr
  role: Backend for the PAM (nearest shrunken centroid) classifier among the seven
    tuned algorithms
- name: RWeka
  role: Backend for the RPART (decision tree) classifier among the seven tuned algorithms
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:3ed2e964de02f32064da45b3cb67b023c3fa6a1f3880ede925ec31a0876e9007
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi
  tools:
  - R
  - RWeka
  - caret
  - e1071
  - gbm
  - glmnet
  - lilikoi
  - pROC
  - pamr
  - randomForest
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# caret-cross-validated-model-tuning

## Summary

Train and tune seven classification algorithms (LDA, SVM, RF, RPART, PAM, LOG, GBM) on Lilikoi's selected pathway deregulation score (PDS) features via caret-based hyperparameter tuning with 10-fold cross-validation, then score held-out predictions with AUC, F1, balanced accuracy, sensitivity, and specificity.

## When to use

Use when building a classification model from metabolomics PDS score features and you need to compare multiple ML algorithms under caret-based hyperparameter tuning with n-fold cross-validation and held-out test metrics.

## When NOT to use

- Input is still a raw metabolite-by-sample measurement matrix that has not been transformed to a PDS score matrix by lilikoi.get_pd_scores
- Significant pathway features have not yet been selected (no lilikoi.select_pathways output, e.g. threshold = 0.42, method = "gain")
- The task is unsupervised or requires automatic algorithm selection — the paper notes AutoML for automatic classification algorithm selection is only planned for the future

## Inputs

- PDS score matrix (pathways x patients) from lilikoi.get_pd_scores
- Sample phenotype labels, e.g. metaboliteMeasurements$Label from plasma_breast_cancer.csv
- Significant pathway feature list from lilikoi.select_pathways (threshold = 0.42, method = "gain")
- Training/testing data split of the PDS matrix with labels

## Outputs

- Fitted classification models for LDA, SVM, RF, RPART, PAM, LOG, and GBM
- Predictions on the held-out test set per algorithm
- ml_metrics.csv: one row per algorithm with AUC, F1, balanced accuracy, sensitivity, and specificity
- mlResults object (mlResults$mlResults) usable by lilikoi.adjust_model

## How to apply

Load the PDS score matrix produced by lilikoi.get_pd_scores, the sample phenotype labels (metaboliteMeasurements$Label), and the significant pathway feature list from lilikoi.select_pathways (e.g. threshold = 0.42, method = "gain"). Call lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways), which trains LDA, SVM, RF, RPART, PAM, LOG, and GBM classifiers on the selected pathway features with caret-based hyperparameter tuning and 10-fold cross-validation (backed by e1071, randomForest, pamr, glmnet, RWeka, and gbm). Split the data into training and testing subsets, fit each tuned model, and predict on the held-out test set. Compute AUC with pROC plus F1, balanced accuracy, sensitivity, and specificity per algorithm, and write the per-algorithm metrics table to ml_metrics.csv. The rationale is that pathway-level PDS features at the individual sample level support biomarker classification, and cross-validated tuning guards against overfitting when features are few.

## Related tools

- **lilikoi** (Provides lilikoi.machine_learning, the wrapper that trains the seven classifiers on selected pathway features) — https://github.com/lanagarmire/lilikoi
- **caret** (Drives hyperparameter tuning and 10-fold cross-validation of the classification models)
- **pROC** (Computes AUC for each algorithm's held-out test predictions)
- **randomForest** (Backend for the RF classifier among the seven tuned algorithms)
- **glmnet** (Backend for the LOG (logistic/regularized) classifier among the seven tuned algorithms)
- **gbm** (Backend for the GBM classifier among the seven tuned algorithms)
- **e1071** (Backend for the SVM classifier among the seven tuned algorithms)
- **pamr** (Backend for the PAM (nearest shrunken centroid) classifier among the seven tuned algorithms)
- **RWeka** (Backend for the RPART (decision tree) classifier among the seven tuned algorithms)

## Examples

```
mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways)
```

## Evaluation signals

- ml_metrics.csv contains exactly one row per algorithm (7 rows: LDA, SVM, RF, RPART, PAM, LOG, GBM)
- All five metrics (AUC, F1, balanced accuracy, sensitivity, specificity) are present and finite for every algorithm
- AUC values are computed on held-out testing data via pROC and fall in the [0, 1] range
- Each model was fit with caret-based tuning under 10-fold cross-validation rather than default single-fit parameters
- The pipeline runs end-to-end from PDSmatrix + labels + significantPathways to mlResults usable by lilikoi.adjust_model

## Limitations

- No AutoML for automatic classification algorithm selection; the authors state this is planned for the future of the classification module
- Upstream metabolite name mapping gaps (about 20% of metabolites cannot be mapped to standard names, partly due to Levenshtein distance matching) can reduce the pathway features available for classification
- Updating Lilikoi's HMDB/SMPDB/MetaboAnalyst backing databases is expensive: datasets are large (>5GB) and the step may take greater than 20 minutes

## Evidence

- [abstract] classification and prediction module, which offers various machine learning classification algorithms: "classification and prediction module, which offers various machine learning classification algorithms"
- [intro] classification and prediction, which lists a series of classification algorithms to derive machine learning models and give predictions on testing datasets.: "classification and prediction, which lists a series of classification algorithms to derive machine learning models and give predictions on testing datasets."
- [readme] mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways): "mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label,
  significantPathways)"
- [methods] significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements, threshold = 0.42, method = "gain"): "significantPathways <- lilikoi.select_pathways(PDSmatrix, metaboliteMeasurements,
  threshold = 0.42, method = "gain")"
