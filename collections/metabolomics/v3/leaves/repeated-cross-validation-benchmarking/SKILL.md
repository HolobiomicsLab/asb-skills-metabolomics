---
name: repeated-cross-validation-benchmarking
description: Use when a labeled metabolomics or other small-n feature matrix must
  be benchmarked across several classifiers, and a stable estimate of held-out performance
  (accuracy, sensitivity, specificity, F1) under repeated k-fold cross-validation
  is needed.
when_to_use_negative:
- Input is unlabeled — no class labels exist, so a supervised 4:1 split with cross-validated
  classification benchmark cannot be performed (use the prognosis/Cox-nnet module
  instead)
- The dataset is too small to sustain a 20% held-out test set with 10-fold CV repeated
  10 times without unstable metric estimates
- The goal is pathway analysis or metabolite-pathway regression rather than comparing
  classifier performance
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: Lilikoi v2.0
  role: R package providing lilikoi.machine_learning, which runs the repeated 10-fold
    CV benchmark of DL against LDA, SVM, RF, RPART, LOG, and GBM
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the classification benchmark is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:9e9201fbd4cd8902cdb08c4b26fb2371a3b688eb3065bb184870ded28c20dbe2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# repeated-cross-validation-benchmarking

## Summary

Benchmark multiple classifiers (deep learning plus LDA, SVM, RF, RPART, LOG, GBM) on a metabolomics dataset using a 4:1 train/test split with 10-fold cross-validation repeated 10 times, and compare held-out accuracy, sensitivity, specificity, and F1. This mirrors the Lilikoi v2.0 ER status classification benchmark on the Charité breast cancer metabolomics dataset (204 ER+ vs 67 ER−, 271 samples × 162 metabolites).

## When to use

Use when a labeled metabolomics or other small-n feature matrix must be benchmarked across several classifiers, and a stable estimate of held-out performance (accuracy, sensitivity, specificity, F1) under repeated k-fold cross-validation is needed.

## When NOT to use

- Input is unlabeled — no class labels exist, so a supervised 4:1 split with cross-validated classification benchmark cannot be performed (use the prognosis/Cox-nnet module instead)
- The dataset is too small to sustain a 20% held-out test set with 10-fold CV repeated 10 times without unstable metric estimates
- The goal is pathway analysis or metabolite-pathway regression rather than comparing classifier performance

## Inputs

- Labeled metabolomics feature matrix (e.g., Charité breast cancer dataset: 271 samples × 162 metabolites with ER+/ER− labels)
- R environment with the lilikoi (v2.0) package and the H2O deep learning backend
- Classifier configuration flags (LDA, SVM, RF, RPART, GBM, LOG, DL) and split parameters (trainportion = 0.8, cvnum = 10, nrun = 10, dlround = 50)

## Outputs

- Per-classifier test-set and training-set metrics: accuracy, sensitivity (SEN), specificity (SPEC), and F1 score
- Classification metrics table (e.g., table1_classification_metrics.csv) ranking the deep learning model against LDA, SVM, RF, RPART, LOG, and GBM

## How to apply

Load the feature matrix with class labels (e.g., 271 samples × 162 metabolites with ER+/ER− labels) in R via Lilikoi v2.0, and split it 4:1 (trainportion = 0.8) into training and test partitions. Run 10-fold cross-validation repeated 10 times (cvnum = 10, nrun = 10) on the training partition for each method: the H2O deep learning model with Rectifier activation and 4 hidden layers of 25 neurons (dlround = 50), plus LDA, SVM, RF, RPART, LOG, and GBM toggled by their Boolean flags in lilikoi.machine_learning. Compute accuracy, sensitivity (SEN), specificity (SPEC), and F1 on the held-out test set (and the training set) for each method, then assemble the metrics into a table (e.g., table1_classification_metrics.csv). Rationale: the personalization/pathway-transformed features and repeated-CV design let the deep learning model's advantage (test accuracy 0.91, SEN 0.95, F1 0.93, SPEC 0.69) be compared fairly against the six classical ML methods on identical splits.

## Related tools

- **Lilikoi v2.0** (R package providing lilikoi.machine_learning, which runs the repeated 10-fold CV benchmark of DL against LDA, SVM, RF, RPART, LOG, and GBM) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the classification benchmark is executed)

## Examples

```
lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround = 50, nrun = 10, Rpart = TRUE, LDA = TRUE, SVM = TRUE, RF = TRUE, GBM = TRUE, PAM = FALSE, LOG = TRUE, DL = TRUE)
```

## Evaluation signals

- The deep learning test-set metrics match the published benchmark: accuracy = 0.91, SEN = 0.95, F1 = 0.93, SPEC = 0.69
- The deep learning training-set metrics match: F1 = 0.95, SEN = 0.98, SPEC = 0.75
- All seven methods (DL, LDA, SVM, RF, RPART, LOG, GBM) were trained on identical 4:1 splits with 10-fold CV repeated 10 times before metric comparison
- The output CSV contains accuracy, SEN, SPEC, and F1 columns for both test and training sets for every enabled classifier

## Limitations

- Benchmarked on a single dataset (Charité breast cancer metabolomics, 271 samples × 162 metabolites); performance may not transfer to other cohorts
- The deep learning model's high sensitivity comes with lower specificity (SPEC = 0.69 on the test set), so the DL advantage over classical methods is not uniform across metrics

## Evidence

- [results] Deep learning achieves the highest test-set performance among all compared methods on the ER classification task, with Accuracy = 0.91, sensitivity = 0.95, and F1 statistic = 0.93.: [evidence span withheld — anchor status: not_found]
- [readme] Split and CV parameters: "trainportion = 0.8, cvnum = 10, dlround=50,nrun=10, Rpart=TRUE, LDA=TRUE,SVM=TRUE,RF=TRUE,GBM=TRUE,PAM=FALSE,LOG=TRUE,DL=TRUE"
- [methods] DL architecture: [evidence span withheld — anchor status: not_found]
- [abstract] Package scope: "The new Lilikoi v2.0 R package has implemented a deep learning method for classification, in addition to popular machine learning methods."
