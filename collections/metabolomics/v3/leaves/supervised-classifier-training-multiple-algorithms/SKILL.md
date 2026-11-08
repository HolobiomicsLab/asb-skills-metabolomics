---
name: supervised-classifier-training-multiple-algorithms
description: Use when you have a pathway deregulation score (PDS) matrix and phenotype
  labels and need to build and compare multiple classification models with cross-validated
  performance metrics.
when_to_use_negative:
- The input is raw metabolite measurements, not a pathway-level feature matrix — run
  lilikoi.metab_to_pathway, lilikoi.get_pd_scores, and lilikoi.select_pathways first
- You need automatic algorithm selection (AutoML) — the article notes AutoML is only
  planned for the future, so algorithms must be chosen/compared manually here
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: lilikoi
  role: R package whose lilikoi.machine_learning function trains and evaluates the
    seven classifiers on pathway features
  repo: https://github.com/lanagarmire/lilikoi
- name: caret
  role: hyperparameter tuning and n-fold cross-validation framework for the classifiers
- name: e1071
  role: provides the SVM classifier backend
- name: randomForest
  role: provides the random forest (RF) classifier backend
- name: gbm
  role: provides the gradient boosting machine (GBM) classifier backend
- name: glmnet
  role: provides the regularized logistic regression (LOG) classifier backend
- name: pamr
  role: provides the prediction analysis for microarrays (PAM) classifier backend
- name: RWeka
  role: provides the decision tree (RPART/J48) classifier backend
- name: pROC
  role: computes AUC from test-set predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy136
    title: 'Lilikoi: an R package for personalized pathway-based classification modeling
      using metabolomics data'
  source_content_hash: sha256:07c6e162e57cae65120b3143f714ce9f7dd339ef2a8b779f23da6f7fbfed4e2f
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# supervised-classifier-training-multiple-algorithms

## Summary

Train and benchmark seven supervised classification algorithms (LDA, SVM, RF, RPART, PAM, LOG, GBM) with n-fold cross-validation on selected pathway features using Lilikoi's classification-and-prediction module, and compare them by AUC, F1, balanced accuracy, sensitivity, and specificity.

## When to use

Use when you have a pathway deregulation score (PDS) matrix and phenotype labels and need to build and compare multiple classification models with cross-validated performance metrics.

## When NOT to use

- The input is raw metabolite measurements, not a pathway-level feature matrix — run lilikoi.metab_to_pathway, lilikoi.get_pd_scores, and lilikoi.select_pathways first
- You need automatic algorithm selection (AutoML) — the article notes AutoML is only planned for the future, so algorithms must be chosen/compared manually here

## Inputs

- PDS score matrix (pathways x samples) from lilikoi.get_pd_scores
- Sample phenotype label vector (e.g. metaboliteMeasurements$Label)
- Significant pathway feature list from lilikoi.select_pathways

## Outputs

- Fitted classification models for LDA, SVM, RF, RPART, PAM, LOG, and GBM
- Per-algorithm metrics table (AUC, F1, balanced accuracy, sensitivity, specificity), e.g. ml_metrics.csv
- mlResults object consumable by lilikoi.adjust_model

## How to apply

Load the PDS score matrix produced by lilikoi.get_pd_scores, the sample phenotype labels (e.g. metaboliteMeasurements$Label), and the significant pathway feature list from lilikoi.select_pathways. Split the data into training and testing subsets, then call lilikoi.machine_learning to train LDA, SVM, RF, RPART, PAM, LOG, and GBM classifiers on the selected pathway features, using caret-based hyperparameter tuning with 10-fold cross-validation (backed by caret, e1071, randomForest, pamr, glmnet, RWeka, and gbm). Predict on the held-out test set with each fitted model and compute AUC with pROC plus F1, balanced accuracy, sensitivity, and specificity per algorithm. Compare algorithms on these five metrics to select the best-performing classifier for the phenotype-prediction task.

## Related tools

- **lilikoi** (R package whose lilikoi.machine_learning function trains and evaluates the seven classifiers on pathway features) — https://github.com/lanagarmire/lilikoi
- **caret** (hyperparameter tuning and n-fold cross-validation framework for the classifiers)
- **e1071** (provides the SVM classifier backend)
- **randomForest** (provides the random forest (RF) classifier backend)
- **gbm** (provides the gradient boosting machine (GBM) classifier backend)
- **glmnet** (provides the regularized logistic regression (LOG) classifier backend)
- **pamr** (provides the prediction analysis for microarrays (PAM) classifier backend)
- **RWeka** (provides the decision tree (RPART/J48) classifier backend)
- **pROC** (computes AUC from test-set predictions)

## Examples

```
mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways)
```

## Evaluation signals

- The script runs end-to-end without errors on the PDS matrix and labels
- ml_metrics.csv contains exactly one row per algorithm (seven rows) with all five metrics (AUC, F1, balanced accuracy, sensitivity, specificity) present and finite
- Each classifier is trained with n-fold (10-fold) cross-validation via caret and predicts on the held-out test set
- AUC values are within [0, 1] and sensitivity/specificity within [0, 1]
- The mlResults object is compatible with downstream lilikoi.adjust_model (mlResults$mlResults)

## Limitations

- No AutoML for automatic classification algorithm selection; the authors only plan to add it in the future
- Feature space is limited to the significant pathways selected upstream, so classifier performance depends on the feature-selection threshold (e.g. threshold = 0.42 with method = 'gain')
- Docker/binder images and large external databases (HMDB, SMPDB, MetaboAnalyst, >5GB) mean setup can take significant time and resources

## Evidence

- [other] The classification module builds a model from the selected pathway features, letting the user choose among seven classification algorithms with n-fold cross-validation, and reports AUC, F1, balanced accuracy, sensitivity, and specificity for the testing data.: [evidence span withheld — anchor status: not_found]
- [abstract] classification and prediction module: offers various machine learning classification algorithms: "classification and prediction module, which offers various machine learning classification algorithms"
- [methods] mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label, significantPathways): "mlResults <- lilikoi.machine_learning(PDSmatrix, metaboliteMeasurements$Label,
  significantPathways)"
- [intro] classification and prediction, which lists a series of classification algorithms to derive machine learning models and give predictions on testing datasets.: "classification and prediction, which lists a series of classification algorithms to derive machine learning models and give predictions on testing datasets."
- [discussion] We plan to use AutoML in our classification module in the future: "We plan to use AutoML in our classification module in the future"
