---
name: deep-learning-neural-network-training-h2o
description: Use when you have a labeled metabolomics matrix (samples x metabolites
  or pathway features, e.g. plasma_breast_cancer.csv) and want to train/compare a
  deep learning classifier against classic ML baselines under cross-validated train/test
  splits.
when_to_use_negative:
- Task is survival/prognosis prediction rather than supervised classification — use
  lilikoi.prognosis (Cox-PH / Cox-nnet) instead
- Input lacks labeled class labels (no Label/phenotype column), since the DL module
  is a supervised classifier — use the exploratory modules (PCA, t-SNE) instead
- Goal is metabolite-to-pathway mapping or pathway visualization only (use lilikoi.MetaTOpathway
  / lilikoi.KEGGplot), not model training
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Lilikoi v2.0
  role: R package exposing lilikoi.machine_learning with DL=TRUE for deep learning
    classification of metabolomics data
  repo: https://github.com/lanagarmire/lilikoi2
- name: R
  role: Programming environment in which the Lilikoi v2.0 classification workflow
    runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa162
    title: 'Lilikoi V2.0: a deep learning–enabled, personalized pathway-based R package
      for diagnosis and prognosis predictions using metabolomics data'
  source_content_hash: sha256:e8df9deb79aac80063b3d3faf7a23b95954763c3af54000c3bbea2eb8b90cfdf
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# deep-learning-neural-network-training-h2o

## Summary

Trains a supervised deep neural network (via H2O) inside the Lilikoi v2.0 R package to classify disease status from metabolomics features, benchmarked against LDA, SVM, RF, RPART, LOG, and GBM. It is the core classification advance of Lilikoi v2.0, enabling metabolomics researchers to test deep learning alongside popular machine learning methods.

## When to use

Use when you have a labeled metabolomics matrix (samples x metabolites or pathway features, e.g. plasma_breast_cancer.csv) and want to train/compare a deep learning classifier against classic ML baselines under cross-validated train/test splits.

## When NOT to use

- Task is survival/prognosis prediction rather than supervised classification — use lilikoi.prognosis (Cox-PH / Cox-nnet) instead
- Input lacks labeled class labels (no Label/phenotype column), since the DL module is a supervised classifier — use the exploratory modules (PCA, t-SNE) instead
- Goal is metabolite-to-pathway mapping or pathway visualization only (use lilikoi.MetaTOpathway / lilikoi.KEGGplot), not model training

## Inputs

- Labeled metabolomics data matrix (samples x metabolites) with a phenotype/Label column, e.g. plasma_breast_cancer.csv (271 samples x 162 metabolites, ER+/ER- labels)
- Optional pathway-transformed feature matrix (PDS matrix) from lilikoi.PDSfun
- Selected pathway feature list from lilikoi.featuresSelection (e.g. method='gain', threshold=0.50)

## Outputs

- Trained H2O deep learning model (Rectifier activation, 4 hidden layers x 25 neurons)
- Per-method test- and training-set metrics table (accuracy, SEN, SPEC, F1) for DL vs LDA, SVM, RF, RPART, LOG, GBM (e.g. table1_classification_metrics.csv)

## How to apply

Load the metabolomics data with lilikoi.Loaddata, optionally convert metabolites to pathways (lilikoi.MetaTOpathway, lilikoi.PDSfun) and select significant pathways (lilikoi.featuresSelection), then call lilikoi.machine_learning with DL=TRUE. In the Charité breast cancer ER benchmark (271 samples x 162 metabolites, ER+ vs ER-), the split is 4:1 (trainportion=0.8) with 10-fold cross-validation repeated 10 times (cvnum=10, nrun=10) and dlround=50. The H2O deep learning model uses a Rectifier activation with 4 hidden layers of 25 neurons each, and is trained on the same splits as LDA, SVM, RF, RPART, LOG, and GBM. Evaluate on the held-out test set with accuracy, sensitivity (SEN), specificity (SPEC), and F1; the published DL test performance was accuracy=0.91, SEN=0.95, F1=0.93, SPEC=0.69 (training F1=0.95, SEN=0.98, SPEC=0.75).

## Related tools

- **Lilikoi v2.0** (R package exposing lilikoi.machine_learning with DL=TRUE for deep learning classification of metabolomics data) — https://github.com/lanagarmire/lilikoi2
- **R** (Programming environment in which the Lilikoi v2.0 classification workflow runs)

## Examples

```
lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround=50, nrun=10, Rpart=TRUE, LDA=TRUE, SVM=TRUE, RF=TRUE, GBM=TRUE, PAM=FALSE, LOG=TRUE, DL=TRUE)
```

## Evaluation signals

- DL test-set metrics match published benchmark: accuracy=0.91, SEN=0.95, F1=0.93, SPEC=0.69 on the Charité ER task
- Training-set DL metrics consistent with Table 1: F1=0.95, SEN=0.98, SPEC=0.75
- All seven methods (DL, LDA, SVM, RF, RPART, LOG, GBM) are trained on identical 4:1 splits with 10-fold CV repeated 10 times, so metrics are directly comparable
- lilikoi.machine_learning call includes DL=TRUE, trainportion=0.8, cvnum=10, nrun=10, dlround=50

## Limitations

- Specificity of the DL model on the ER- minority class was substantially lower (SPEC=0.69) than sensitivity, reflecting class imbalance (204 ER+ vs 67 ER-)
- Deep learning is one of many options in Lilikoi v2.0; it is not guaranteed to win on every metabolomics dataset — the package deliberately compares it with LDA, SVM, RF, RPART, LOG, and GBM
- Prognosis (survival) modeling is handled by a different module (Cox-PH / Cox-nnet), not by the classification DL trainer

## Evidence

- [full_text] Deep learning achieves the highest test-set performance among all compared methods on the ER classification task, with Accuracy = 0.91, sensitivity = 0.95, and F1 statistic = 0.93.: [evidence span withheld — anchor status: not_found]
- [full_text] Train the deep learning model using H2O with a Rectifier activation and 4 hidden layers of 25 neurons each.: [evidence span withheld — anchor status: not_found]
- [abstract] The new Lilikoi v2.0 R package has implemented a deep learning method for classification, in addition to popular machine learning methods.: "The new Lilikoi v2.0 R package has implemented a deep learning method for classification, in addition to popular machine learning methods."
- [readme] lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround=50,nrun=10, Rpart=TRUE,: "lilikoi.machine_learning(MLmatrix = Metadata, measurementLabels = Metadata$Label, significantPathways = 0, trainportion = 0.8, cvnum = 10, dlround=50,nrun=10, Rpart=TRUE,"
