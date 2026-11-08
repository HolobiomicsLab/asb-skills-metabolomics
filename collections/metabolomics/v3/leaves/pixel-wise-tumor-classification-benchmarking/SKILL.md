---
name: pixel-wise-tumor-classification-benchmarking
description: Use when comparing a deep learning pixel-wise classifier against a classical
  ML baseline (e.g. SVM) on withheld MALDI MSI spectra, or when building a side-by-side
  accuracy/speed table such as the paper's Table 1.
when_to_use_negative:
- The task requires multi-class tissue classification — the article demonstrates only
  binary normal/tumor classification and leaves multi-class for future development
- No pathologist-annotated ground truth (H&E-derived labels) exists for the spectra,
  since the benchmark depends on pixel-wise normal/tumor annotation
- The input is already a preprocessed feature/peak table rather than raw pixel-level
  mass spectra with tens of thousands of m/z variables
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning classifier being benchmarked; performs probabilistic pixel-based
    classification directly from mass spectral data without prior peak picking
  repo: github.com/wabdelmoula/massNet
- name: sklearn
  role: Provides the support vector machine (SVM) classical ML baseline trained on
    the same pixel-wise labels
- name: Keras
  role: Platform in which the massNet architecture was implemented in Python
- name: tensorflow
  role: Backend for the Keras-implemented massNet architecture
- name: h5py
  role: Library for loading the HDF5-converted MSI spectra (85,062 m/z variables)
    into the training/test pipelines
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:7d0af1b2521633ada1fa317188b17489724ea39204c4f1b5cc18291740be62d7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  tools:
  - Keras
  - Python
  - h5py
  - massNet
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 24
---

# pixel-wise-tumor-classification-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a pixel-wise deep learning MSI classifier (massNet) against a classical baseline (SVM) on held-out MALDI FT-ICR mass spectra, reporting accuracy, precision, recall, F1-score, ROC, and classification runtime side-by-side to quantify both performance and speed gains.

## When to use

Use when comparing a deep learning pixel-wise classifier against a classical ML baseline (e.g. SVM) on withheld MALDI MSI spectra, or when building a side-by-side accuracy/speed table such as the paper's Table 1.

## When NOT to use

- The task requires multi-class tissue classification — the article demonstrates only binary normal/tumor classification and leaves multi-class for future development
- No pathologist-annotated ground truth (H&E-derived labels) exists for the spectra, since the benchmark depends on pixel-wise normal/tumor annotation
- The input is already a preprocessed feature/peak table rather than raw pixel-level mass spectra with tens of thousands of m/z variables

## Inputs

- Withheld MALDI FT-ICR MSI test set in HDF5 format (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables)
- Training set with pixel-wise normal/tumor labels (10,024 normal and 3,644 tumor spectra; 85,062 m/z variables)
- Pathologist-annotated H&E tumor regions transferred to the MSI data (ground truth labels)
- imzML data exported from SCiLS lab 2020a, converted to HDF5 for deep learning analysis

## Outputs

- Side-by-side benchmark table (svm_benchmark_table.csv) with SVM and massNet accuracy, precision, recall, F1-score, runtime, and speed-ratio
- SVM test-set classification predictions on the withheld spectra
- ROC curves and standard ML metrics for both classifiers
- SVM-to-massNet speed comparison ratio (expected ~174-fold)

## How to apply

Load the withheld test set (9,798 normal and 2,861 tumor spectra, 85,062 m/z variables) from the converted HDF5 file, and the training set (10,024 normal and 3,644 tumor spectra) with the same pixel-wise normal/tumor labels used for massNet. Train the SVM baseline on the raw spectra without prior preprocessing or peak picking, mirroring how massNet consumes the data directly. Apply the trained SVM to the test set and evaluate with accuracy, precision, recall, F1-score, and ROC — the same established machine learning metrics used for massNet — plus measure test-set classification runtime to compute the speed ratio against massNet (expected ~174-fold gain). Assemble both models' metrics and runtimes into a single side-by-side table (Table 1, saved as svm_benchmark_table.csv). Model stability of the primary classifier should also be checked via 5-fold cross validation. Reproducible environment per the README: Python 3.6.12, Keras 2.2.0 with TensorFlow 1.8.0 backend, sklearn 0.23.2, h5py 2.7.1.

## Related tools

- **massNet** (Deep learning classifier being benchmarked; performs probabilistic pixel-based classification directly from mass spectral data without prior peak picking) — github.com/wabdelmoula/massNet
- **sklearn** (Provides the support vector machine (SVM) classical ML baseline trained on the same pixel-wise labels)
- **Keras** (Platform in which the massNet architecture was implemented in Python)
- **tensorflow** (Backend for the Keras-implemented massNet architecture)
- **h5py** (Library for loading the HDF5-converted MSI spectra (85,062 m/z variables) into the training/test pipelines)

## Examples

```
from sklearn.svm import SVC; svm = SVC(); svm.fit(X_train, y_train); preds = svm.predict(X_test)
```

## Evaluation signals

- Reproduced SVM accuracy on the withheld test set matches the reported 93.45% (massNet reported at 95%)
- Computed SVM-to-massNet classification runtime ratio is consistent with the reported 174-fold gain in speed
- Benchmark table includes all five reported metrics for SVM (accuracy, precision, recall, F1-score, ROC) plus runtime
- Test-set composition matches 9,798 normal and 2,861 tumor spectra with 85,062 m/z variables; training set has 10,024 normal and 3,644 tumor spectra
- Primary classifier stability is verified via 5-fold cross validation before benchmarking

## Limitations

- Only binary class classification demonstrated; multi-class classification is left for future development
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity
- Establishing ground truth annotation of MSI data is challenging for other tissues and tumor types, and tumor margin annotation requires more pathological investigation

## Evidence
