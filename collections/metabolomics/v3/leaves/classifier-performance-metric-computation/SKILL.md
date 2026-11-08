---
name: classifier-performance-metric-computation
description: Use when several classifiers (e.g. DL plus LDA/SVM/RF/RPART/LOG/GBM)
  have been fitted on the same train/test split of a metabolomics feature matrix and
  you need comparable accuracy, SEN, SPEC and F1 values to rank them, as in the Lilikoi
  ER benchmark (accuracy 0.91, SEN 0.95, F1 0.93 for DL).
when_to_use_negative:
- When a single model has been fitted and there is no held-out test partition — the
  skill requires a train/test split (e.g. 4:1) so test-set metrics can be computed
- When the task is regression or survival prognosis (e.g. Cox-PH / Cox-nnet in Lilikoi)
  rather than binary classification, since Accuracy/SEN/SPEC/F1 are not applicable
- When comparing classifiers trained on different feature matrices or splits — metrics
  are only comparable when all methods were fitted on the same partitioned data
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3520
tools:
- name: Lilikoi v2.0
  role: The lilikoi.machine_learning function fits the DL/LDA/SVM/RF/RPART/GBM/LOG
    classifiers and computes their performance statistics (trainportion=0.8, cvnum=10,
    nrun=10) on the PDS pathway-level features.
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Environment in which Lilikoi v2.0 is run to compute and tabulate the classification
    metrics.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:c24ea2010604fb1bd8935894b29e6afe52081173c403ec92ee0d944cb0a64da7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# classifier-performance-metric-computation

## Summary

Computes test-set and training-set Accuracy, sensitivity (SEN), specificity (SPEC), and F1 statistics for the deep learning and six comparison classifiers (LDA, SVM, RF, RPART, LOG, GBM) trained within the Lilikoi v2.0 classification module, enabling direct head-to-head benchmarking on metabolomics data such as the Charité breast cancer ER dataset.

## When to use

Use when several classifiers (e.g. DL plus LDA/SVM/RF/RPART/LOG/GBM) have been fitted on the same train/test split of a metabolomics feature matrix and you need comparable accuracy, SEN, SPEC and F1 values to rank them, as in the Lilikoi ER benchmark (accuracy 0.91, SEN 0.95, F1 0.93 for DL).

## When NOT to use

- When a single model has been fitted and there is no held-out test partition — the skill requires a train/test split (e.g. 4:1) so test-set metrics can be computed
- When the task is regression or survival prognosis (e.g. Cox-PH / Cox-nnet in Lilikoi) rather than binary classification, since Accuracy/SEN/SPEC/F1 are not applicable
- When comparing classifiers trained on different feature matrices or splits — metrics are only comparable when all methods were fitted on the same partitioned data

## Inputs

- Fitted classifier models (deep learning, LDA, SVM, RF, RPART, LOG, GBM) produced by lilikoi.machine_learning on a metabolomics feature matrix
- Held-out test-set partition (20% of samples) with binary class labels (e.g. ER+ vs ER−)
- Training-set partition (80% of samples) with binary class labels for training-set metric computation

## Outputs

- Per-classifier table of Accuracy, sensitivity (SEN), specificity (SPEC), and F1 statistic on the test set and on the training set
- Ranked comparison across methods (e.g. DL accuracy 0.91 / SEN 0.95 / F1 0.93 vs GBM 0.86 on the ER task)

## How to apply

After fitting each classifier on the 4:1 train/test split with 10-fold cross-validation repeated 10 times, apply the fitted model to the held-out test set and compute Accuracy = (TP+TN)/N, sensitivity = TP/(TP+FN), specificity = TN/(TN+FP), and F1 = 2·SEN·PPV/(SEN+PPV) on both the test partition and the training partition. For the Lilikoi ER task the deep learning model (H2O Rectifier activation, 4 hidden layers of 25 neurons) is expected to reach test-set Accuracy = 0.91, SEN = 0.95, SPEC = 0.69, F1 = 0.93, and training-set F1 = 0.95, SEN = 0.98, SPEC = 0.75, with GBM reaching test accuracy 0.86 and training F1 0.88 as the next-best comparison. Collect the metrics for all methods (DL, LDA, SVM, RF, RPART, LOG, GBM) into a single table so test and training statistics can be compared side by side; run the computation repeatedly across the nrun=10 outer repetitions to obtain stable estimates rather than a single split. Report test-set statistics as the primary ranking metric to avoid over-optimistic training-fit values, and use the training F1/SEN/SPEC only as a sanity check that models fit the training data.

## Related tools

- **Lilikoi v2.0** (The lilikoi.machine_learning function fits the DL/LDA/SVM/RF/RPART/GBM/LOG classifiers and computes their performance statistics (trainportion=0.8, cvnum=10, nrun=10) on the PDS pathway-level features.) — https://github.com/lanagarmire/lilikoi2
- **R** (Environment in which Lilikoi v2.0 is run to compute and tabulate the classification metrics.)

## Examples

```
lilikoi.machine_learning(MLmatrix = PDSmatrix, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround = 50, nrun = 10, Rpart = TRUE, LDA = TRUE, SVM = TRUE, RF = TRUE, GBM = TRUE, LOG = TRUE, DL = TRUE)
```

## Evaluation signals

- Test-set metrics for the deep learning model on the Charité ER dataset should match Table 1: Accuracy = 0.91, SEN = 0.95, SPEC = 0.69, F1 = 0.93
- Training-set metrics for the same model should be F1 = 0.95, SEN = 0.98, SPEC = 0.75 — training F1 should be ≥ test F1, and a large gap signals overfitting
- Accuracy should lie between the sensitivity and specificity bounds; e.g. DL test accuracy 0.91 must be consistent with SEN 0.95 and SPEC 0.69 given the class balance (204 ER+ vs 67 ER−)
- All seven methods (DL, LDA, SVM, RF, RPART, LOG, GBM) must be evaluated on the identical test partition so rankings are comparable; GBM test accuracy 0.86 should rank below DL 0.91
- Metrics should be reproducible across the repeated 10-fold CV runs (nrun=10), not varying wildly between repetitions

## Limitations

- Test-set statistics are computed on a single held-out split (20% of samples), so they carry sampling noise; the 10-fold CV repeated 10 times mitigates but does not remove this.
- Metric computation assumes balanced-enough binary labels; the ER dataset has 204 ER+ vs 67 ER− samples, so accuracy alone can be misleading without SEN/SPEC/F1.
- The skill as grounded here applies to Lilikoi's classification module; it does not cover the prognosis (Cox-PH/Cox-nnet) metrics.

## Evidence

- [full_text] deep learning test metrics: [evidence span withheld — anchor status: not_found]
- [readme] machine learning call: "lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround=50,nrun=10, Rpart=TRUE,"
- [abstract] scope of classification module: "The new Lilikoi v2.0 R package has implemented a deep learning method for classification, in addition to popular machine learning methods."
- [intro] feature pipeline: "It has 4 modules: feature mapper, dimension transformer, feature selector, and classification predictor"
