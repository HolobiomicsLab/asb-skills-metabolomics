---
name: metabolomics-er-status-classification
description: Use when you have a labeled metabolomics matrix (e.g., 271 breast cancer
  samples × 162 metabolites with ER+/ER− labels) and need supervised classification
  plus model comparison.
when_to_use_negative:
- The analysis goal is prognosis/survival prediction rather than binary classification
  — use the Lilikoi prognosis module (lilikoi.prognosis with Cox-PH / Cox-nnet) instead
- Input is raw metabolite-level data where you want to keep metabolites as features
  rather than aggregate them into pathway-level features
- Only unlabeled metabolomics data is available (no phenotype labels), since classification
  requires Metadata labels to train and evaluate
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: Lilikoi v2.0
  role: 'R package providing the full classification workflow: Loaddata, MetaTOpathway,
    PDSfun, featuresSelection, and machine_learning (including the deep learning classifier)'
  repo: https://github.com/lanagarmire/lilikoi2
- name: H2O
  role: Backend used to train the deep learning model (Rectifier activation, 4 hidden
    layers of 25 neurons) inside lilikoi.machine_learning
- name: R
  role: Runtime environment in which the Lilikoi v2.0 classification workflow is executed
  repo: https://github.com/lanagarmire/lilikoi2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:beb4f87358927e060d2a5401e8751c254ecf8e38a8d44251e9994a530f0b2cd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/lilikoi2
  tools:
  - H2O
  - Lilikoi v2.0
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metabolomics-er-status-classification

## Summary

Classify disease/phenotype status from metabolomics data using Lilikoi v2.0's pathway-transformed features, benchmarking a H2O deep learning model against six classical ML classifiers under repeated 10-fold cross-validation.

## When to use

Use when you have a labeled metabolomics matrix (e.g., 271 breast cancer samples × 162 metabolites with ER+/ER− labels) and need supervised classification plus model comparison.

## When NOT to use

- The analysis goal is prognosis/survival prediction rather than binary classification — use the Lilikoi prognosis module (lilikoi.prognosis with Cox-PH / Cox-nnet) instead
- Input is raw metabolite-level data where you want to keep metabolites as features rather than aggregate them into pathway-level features
- Only unlabeled metabolomics data is available (no phenotype labels), since classification requires Metadata labels to train and evaluate

## Inputs

- Labeled metabolomics abundance matrix (samples × metabolites) with phenotype labels (e.g., plasma_breast_cancer.csv via lilikoi.Loaddata, returning Metadata and dataSet)
- Pathway-features matrix (PDSmatrix) produced by lilikoi.PDSfun
- Optional: user-supplied train/test split proportion, number of CV folds, and number of repeated runs

## Outputs

- Classification metrics per classifier (accuracy, SEN, SPEC, F1) on training and test sets, e.g., DL test: Accuracy 0.91, SEN 0.95, SPEC 0.69, F1 0.93
- Trained model objects for each enabled method (deep learning, LDA, SVM, RF, RPART, LOG, GBM)
- Selected significant pathway list from lilikoi.featuresSelection

## How to apply

Load the metabolomics dataset via lilikoi.Loaddata, map metabolite names to HMDB ids with lilikoi.MetaTOpathway, and transform metabolites into pathway features with lilikoi.PDSfun (pathway-based dimension transformation). Select significant pathways with lilikoi.featuresSelection (e.g., Weka 'gain' method, threshold = 0.50), then run lilikoi.machine_learning on the pathway matrix with an 80/20 train/test split (trainportion = 0.8), 10-fold cross-validation repeated 10 times (cvnum = 10, nrun = 10). Enable the comparison classifiers you want (LDA, SVM, RF, RPART, LOG, GBM) and set DL = TRUE to add the deep learning model, which uses H2O with Rectifier activation and 4 hidden layers of 25 neurons (dlround = 50 rounds). Compute accuracy, sensitivity, specificity, and F1 on the held-out test set; in the article's ER benchmark the deep learning model achieved Accuracy = 0.91, SEN = 0.95, SPEC = 0.69, F1 = 0.93, the highest test performance among all methods.

## Related tools

- **Lilikoi v2.0** (R package providing the full classification workflow: Loaddata, MetaTOpathway, PDSfun, featuresSelection, and machine_learning (including the deep learning classifier)) — https://github.com/lanagarmire/lilikoi2
- **H2O** (Backend used to train the deep learning model (Rectifier activation, 4 hidden layers of 25 neurons) inside lilikoi.machine_learning)
- **R** (Runtime environment in which the Lilikoi v2.0 classification workflow is executed) — https://github.com/lanagarmire/lilikoi2

## Examples

```
lilikoi.machine_learning(MLmatrix=Metadata, measurementLabels=Metadata$Label, significantPathways=0, trainportion=0.8, cvnum=10, dlround=50, nrun=10, Rpart=TRUE, LDA=TRUE, SVM=TRUE, RF=TRUE, GBM=TRUE, PAM=FALSE, LOG=TRUE, DL=TRUE)
```

## Evaluation signals

- Test-set metrics for the DL model on the Charité ER benchmark match Table 1: Accuracy = 0.91, SEN = 0.95, SPEC = 0.69, F1 = 0.93
- Training-set DL metrics match Table 1: F1 = 0.95, SEN = 0.98, SPEC = 0.75
- The DL model ranks highest in test-set performance among all enabled classifiers (LDA, SVM, RF, RPART, LOG, GBM, DL)
- Splitting used 4:1 train/test with 10-fold CV repeated 10 times on the training partition; deviation should be reported as a parameter change (trainportion, cvnum, nrun)
- Selected pathway features are a strict subset of the pathway table and pass the featuresSelection threshold (e.g., gain ≥ 0.50)

## Limitations

- Specificity of the best model was comparatively low (SPEC = 0.69 on the ER test set), so interpretation should include all metrics, not only accuracy
- Deep learning requires the H2O backend and explicit hyperparameters (Rectifier activation, 4 hidden layers of 25 neurons, dlround = 50); results depend on these settings
- Benchmark reported is on a specific breast cancer dataset (204 ER+ vs 67 ER−); performance on other datasets/phenotypes is not established in this context

## Evidence

- [results] Deep learning achieves highest test-set performance: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract: DL added for classification: "The new Lilikoi v2.0 R package has implemented a deep learning method for classification, in addition to popular machine learning methods."
- [intro] v1 scope: classification via ML methods: "Lilikoi v1 enables classifications using various machine learning methods."
- [readme] README: machine_learning call: "lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround=50,nrun=10, Rpart=TRUE,"
- [readme] README: dataset loading: "dt <- lilikoi.Loaddata(file=system.file("extdata", "plasma_breast_cancer.csv", package = "lilikoi"))"
