---
name: train-test-split-evaluation-metabolomics
description: Use when you have a labeled metabolomics feature matrix (samples × metabolites
  or samples × pathway scores) and must rank classifiers (deep learning vs LDA/SVM/RF/RPART/LOG/GBM)
  under a consistent train/test split with repeated cross-validation.
when_to_use_negative:
- Input is already a reduced pathway-level feature matrix and the goal is feature
  selection or pathway regression rather than classifier benchmarking (use lilikoi.featuresSelection
  or lilikoi.meta_path instead).
- The data have no labels and the objective is unsupervised exploratory analysis (PCA/t-SNE)
  rather than train/test supervised evaluation.
- The sample size is too small for a 4:1 held-out test set to be representative; repeated
  CV alone without a held-out test partition is safer.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Lilikoi v2.0
  role: R package whose lilikoi.machine_learning() function orchestrates the train/test
    split, repeated cross-validation, and evaluation of DL plus LDA/SVM/RF/RPART/LOG/GBM
    classifiers
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which Lilikoi v2.0 runs the splitting, cross-validation,
    and metric computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:f1c8d64be1cf596035784dda7d2f15aaac6736db553a9566245c720d41180cf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# train-test-split-evaluation-metabolomics

## Summary

Evaluate metabolomics classifiers by splitting labeled samples (e.g. 271-sample Charité breast cancer ER+/ER− cohort) 4:1 into train/test, using repeated 10-fold cross-validation on the train portion and comparing multiple ML models plus a deep learning (H2O rectifier, 4×25 hidden layers) model on accuracy, sensitivity, specificity, and F1.

## When to use

Use when you have a labeled metabolomics feature matrix (samples × metabolites or samples × pathway scores) and must rank classifiers (deep learning vs LDA/SVM/RF/RPART/LOG/GBM) under a consistent train/test split with repeated cross-validation.

## When NOT to use

- Input is already a reduced pathway-level feature matrix and the goal is feature selection or pathway regression rather than classifier benchmarking (use lilikoi.featuresSelection or lilikoi.meta_path instead).
- The data have no labels and the objective is unsupervised exploratory analysis (PCA/t-SNE) rather than train/test supervised evaluation.
- The sample size is too small for a 4:1 held-out test set to be representative; repeated CV alone without a held-out test partition is safer.

## Inputs

- Labeled metabolomics feature matrix (e.g. 271 samples × 162 metabolites, Charité breast cancer cohort)
- Phenotype labels (ER+ / ER−)
- Split parameters: train/test ratio (4:1), number of CV folds (10), number of CV repeats (10)

## Outputs

- Per-model classification metrics table (accuracy, SEN, SPEC, F1) for test and training sets
- Comparison of deep learning vs ML classifiers (e.g. Table 1: DL test Accuracy = 0.91, SEN = 0.95, F1 = 0.93)

## How to apply

Load the labeled metabolomics dataset (e.g. 271 samples × 162 metabolites with ER+/ER− labels) and split 4:1 into train and test partitions (trainportion = 0.8). Run 10-fold cross-validation (cvnum = 10) repeated 10 times (nrun = 10) on the training partition. Train the deep learning model via H2O with Rectifier activation and 4 hidden layers of 25 neurons each (dlround = 50), alongside LDA, SVM, RF, RPART, LOG, and GBM. Compute accuracy, sensitivity (SEN), specificity (SPEC), and F1 on the held-out test set and training set, then assemble the metrics into a comparison table (e.g. table1_classification_metrics.csv) to judge which model generalizes best.

## Related tools

- **Lilikoi v2.0** (R package whose lilikoi.machine_learning() function orchestrates the train/test split, repeated cross-validation, and evaluation of DL plus LDA/SVM/RF/RPART/LOG/GBM classifiers) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which Lilikoi v2.0 runs the splitting, cross-validation, and metric computation)

## Examples

```
R> lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround = 50, nrun = 10, Rpart = TRUE, LDA = TRUE, SVM = TRUE, RF = TRUE, GBM = TRUE, PAM = FALSE, LOG = TRUE, DL = TRUE)
```

## Evaluation signals

- Deep learning test metrics should match published values: Accuracy = 0.91, sensitivity = 0.95, F1 = 0.93, SPEC = 0.69; training F1 = 0.95, SEN = 0.98, SPEC = 0.75.
- Each model must be evaluated on the identical held-out test partition generated by the same 4:1 split.
- Cross-validation must use 10 folds repeated 10 times (cvnum = 10, nrun = 10) on the training portion (trainportion = 0.8).
- The deep learning model uses Rectifier activation with 4 hidden layers of 25 neurons each (dlround = 50).
- The output table contains metrics for all seven classifiers (DL, LDA, SVM, RF, RPART, LOG, GBM) so direct comparison is possible.

## Limitations

- Specificity of the deep learning model is notably lower (0.69) than its sensitivity (0.95), so the method may under-detect the minority ER− class.
- The benchmark is grounded in one breast cancer metabolomics dataset; other data types may not behave similarly.
- Requires a labeled dataset large enough to support a meaningful held-out test set.

## Evidence

- [abstract] Lilikoi v2.0 adds a deep learning method for classification alongside popular machine learning methods: "The new Lilikoi v2.0 R package has implemented a deep learning method for classification, in addition to popular machine learning methods."
- [readme] README call signature for machine_learning evaluation: "lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label,
 significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround=50,nrun=10, Rpart=TRUE,"
- [intro] Lilikoi v1 enables classifications using various machine learning methods: "Lilikoi v1 enables classifications using various machine learning methods."
- [full_text] GigaScience article describing Lilikoi v2 classification benchmark on ER+ vs ER− breast cancer metabolomics: [evidence span withheld — anchor status: not_found]
