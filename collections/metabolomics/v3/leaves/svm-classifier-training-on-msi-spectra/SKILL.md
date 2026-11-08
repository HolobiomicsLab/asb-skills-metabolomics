---
name: svm-classifier-training-on-msi-spectra
description: Use when you need a classical supervised classification baseline for
  raw high-dimensional mass spectra (e.g., 85,062 m/z variables from MALDI FT-ICR
  MSI) with pixel-level normal/tumor labels, to benchmark against a deep learning
  model.
when_to_use_negative:
- The input is already a reduced feature/peak table or dimensionality-reduced embedding
  (e.g., the 5-dimensional latent variable from the massNet VAE Code layer) rather
  than the full 85,062 m/z raw spectra.
- The goal is classification of more than the binary normal/tumor classes, which the
  source model does not support (multi-class classification is left for future development).
- No ground-truth pixel annotations are available; the paper notes that establishing
  ground truth annotation of MSI data is a challenging task requiring more pathological
  investigation.
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: support vector machine (SVM)
  role: Classical machine learning classifier trained on pixel-wise normal/tumor labels
    and benchmarked against massNet on the withheld test set
- name: Python
  role: Implementation environment used to train the SVM and evaluate classification
    metrics
  repo: https://github.com/wabdelmoula/massNet
- name: sklearn
  role: scikit-learn (0.23.2) package providing the SVM implementation and evaluation
    metrics in the massNet environment
  repo: https://github.com/wabdelmoula/massNet
- name: h5py
  role: h5py (2.7.1) used to read the imzML-to-HDF5 converted MSI spectral data for
    training and testing
  repo: https://github.com/wabdelmoula/massNet
- name: massNet
  role: Deep learning comparator whose accuracy (95%) and runtime define the benchmark
    targets for the SVM comparison
  repo: https://github.com/wabdelmoula/massNet
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:de8006fdde83554e90b4c451495e13c40732292ddf55fbf503bac5c8b3f03eeb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/wabdelmoula/massNet
  repo_url: https://github.com/wabdelmoula/massNet
  tools:
  - Python
  - h5py
  - massNet
  - sklearn
  - support vector machine (SVM)
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# svm-classifier-training-on-msi-spectra

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a support vector machine (SVM), a widely-used classical machine learning baseline, on pixel-wise normal/tumor-labeled MALDI FT-ICR MSI spectra to benchmark deep learning classifiers such as massNet on accuracy, precision, recall, F1-score, ROC, and runtime.

## When to use

Use when you need a classical supervised classification baseline for raw high-dimensional mass spectra (e.g., 85,062 m/z variables from MALDI FT-ICR MSI) with pixel-level normal/tumor labels, to benchmark against a deep learning model.

## When NOT to use

- The input is already a reduced feature/peak table or dimensionality-reduced embedding (e.g., the 5-dimensional latent variable from the massNet VAE Code layer) rather than the full 85,062 m/z raw spectra.
- The goal is classification of more than the binary normal/tumor classes, which the source model does not support (multi-class classification is left for future development).
- No ground-truth pixel annotations are available; the paper notes that establishing ground truth annotation of MSI data is a challenging task requiring more pathological investigation.

## Inputs

- MALDI FT-ICR MSI training set converted to HDF5 (10,024 normal and 3,644 tumor spectra; 85,062 m/z variables)
- MALDI FT-ICR MSI withheld test set converted to HDF5 (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables)
- Pixel-wise normal/tumor labels derived from pathologist-annotated H&E images transferred to the MSI data

## Outputs

- SVM test-set predictions (normal/tumor) for the withheld test set
- SVM performance metrics: accuracy, precision, recall, F1-score, and ROC
- Side-by-side SVM vs. massNet benchmark table including classification runtime and speed ratio (svm_benchmark_table.csv)

## How to apply

Load the withheld test set (9,798 normal and 2,861 tumor spectra with 85,062 m/z variables) and the training set (10,024 normal and 3,644 tumor spectra) from the HDF5-converted MSI data using h5py, with pixel-wise normal/tumor labels transferred from pathologist-annotated H&E images. Train the SVM classifier (scikit-learn, version 0.23.2 in the massNet environment) on the same pixel-wise normal/tumor labels used to train massNet. Apply the trained SVM to the withheld test set and record predictions, then evaluate with the established metrics reported in the paper: accuracy, precision, recall, F1-score, and ROC. Measure the SVM classification runtime on the test set and compute the speed ratio against the deep learning model to confirm the reported 174-fold gain (SVM 93.45% accuracy vs. massNet 95%). Assemble the side-by-side metrics and runtime values into a single comparison table (Table 1) saved as svm_benchmark_table.csv.

## Related tools

- **support vector machine (SVM)** (Classical machine learning classifier trained on pixel-wise normal/tumor labels and benchmarked against massNet on the withheld test set)
- **Python** (Implementation environment used to train the SVM and evaluate classification metrics) — https://github.com/wabdelmoula/massNet
- **sklearn** (scikit-learn (0.23.2) package providing the SVM implementation and evaluation metrics in the massNet environment) — https://github.com/wabdelmoula/massNet
- **h5py** (h5py (2.7.1) used to read the imzML-to-HDF5 converted MSI spectral data for training and testing) — https://github.com/wabdelmoula/massNet
- **massNet** (Deep learning comparator whose accuracy (95%) and runtime define the benchmark targets for the SVM comparison) — https://github.com/wabdelmoula/massNet

## Examples

```
from sklearn.svm import SVC; clf = SVC().fit(X_train, y_train); preds = clf.predict(X_test)
```

## Evaluation signals

- Reproduced SVM accuracy on the withheld test set should be ≈ 93.45% versus massNet's 95%.
- Computed SVM-to-massNet speed ratio should be consistent with the reported 174-fold gain in speed.
- Reproduced table must include SVM accuracy, precision, recall, F1-score, and runtime side-by-side with the massNet values (Table 1 format).
- Test-set composition must match the paper: 9,798 normal and 2,861 tumor spectra with 85,062 m/z variables.
- Same pixel-wise normal/tumor labels must be used for both SVM and massNet training to ensure a fair benchmark.

## Limitations

- The paper reports benchmark comparison but does not provide SVM hyperparameter values (kernel, C, gamma) in the section text, which may impede exact reproduction.
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity.
- The comparison is limited to binary normal/tumor classification; multi-class classification is left for future development.

## Evidence

- [full_text] massNet slightly outperformed SVM in classification accuracy on the withheld test set (95% vs. 93.45%) while being 174 times faster, as reported in the Table 1 side-by-side benchmark.: [evidence span withheld — anchor status: not_found]
