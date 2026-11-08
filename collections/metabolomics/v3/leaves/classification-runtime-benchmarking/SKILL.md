---
name: classification-runtime-benchmarking
description: Use when a trained model (e.g., massNet on MALDI FT-ICR MSI spectra)
  must be evaluated on an independent test set for accuracy, per-class AUC, and classification
  speed relative to a baseline method like SVM.
when_to_use_negative:
- The data has not been split into independent training/test sets and no held-out
  labels exist — only cross-validation assessment is possible
- The task is model training or hyperparameter optimization rather than evaluating
  a already-trained classifier on withheld data
- Ground-truth annotations for the tissue classes are unavailable (e.g., no pathologist-annotated
  H&E sections consecutive to the MSI sections)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Trained deep learning model (VAE + sigmoid classifier) whose probabilistic
    inference on test spectra is being benchmarked for accuracy and runtime
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Deep learning platform in which the massNet architecture is implemented and
    through which test-set inference is executed
- name: tensorflow
  role: Backend for Keras used to run probabilistic classification of each test spectrum
- name: Python
  role: Implementation language for the evaluation script computing metrics and runtime
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:2b787e951567cde795712d24b526a92f658ae8fcc60fab5c80d8eb5e0eb6b5eb
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
  - Keras
  - Python
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# classification-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a trained classifier's accuracy and inference speed on a withheld test set using standard ML metrics (ROC AUC, confusion matrix, accuracy, precision/recall/F1) plus spectra-per-unit-time runtime, compared against a classical baseline such as SVM.

## When to use

Use when a trained model (e.g., massNet on MALDI FT-ICR MSI spectra) must be evaluated on an independent test set for accuracy, per-class AUC, and classification speed relative to a baseline method like SVM.

## When NOT to use

- The data has not been split into independent training/test sets and no held-out labels exist — only cross-validation assessment is possible
- The task is model training or hyperparameter optimization rather than evaluating a already-trained classifier on withheld data
- Ground-truth annotations for the tissue classes are unavailable (e.g., no pathologist-annotated H&E sections consecutive to the MSI sections)

## Inputs

- Withheld test set of MALDI FT-ICR MSI spectra in HDF5 format (9,798 normal and 2,861 tumor spectra; 85,062 m/z variables)
- Ground-truth normal/tumor labels derived from expert pathologist annotations of H&E images transferred to the MSI data
- Trained massNet model (VAE encoder plus sigmoid-activated classifier on the 5-dimensional latent variable) in Keras/TensorFlow

## Outputs

- Metrics table (e.g., massnet_test_metrics.csv) with confusion matrix, accuracy, per-class precision, recall, F1-score, and per-class ROC AUC
- ROC curve plot (e.g., roc_curve.png)
- Classification runtime measurement (spectra per unit time) for comparison against the SVM benchmark

## How to apply

Load the withheld test set (here 9,798 normal and 2,861 tumor MALDI FT-ICR MSI spectra with 85,062 m/z variables in HDF5 format) together with ground-truth labels from pathologist-annotated H&E regions. Run probabilistic inference through the model's sigmoid output layer (massNet: VAE encoder plus sigmoid classifier on the 5-dimensional latent variable, implemented in Keras 2.2.0 with a TensorFlow 1.8.0 backend) without any prior preprocessing or peak picking. Compute the confusion matrix, accuracy, per-class precision, recall, F1-score, and per-class ROC AUC over the full independent test set (12,659 spectra). Time the classification to obtain spectra per unit time and compare against the SVM benchmark, for which massNet reported a 174-fold speed gain and higher accuracy. Write the metrics table (e.g., massnet_test_metrics.csv) and save the ROC curve (e.g., roc_curve.png) for validation.

## Related tools

- **massNet** (Trained deep learning model (VAE + sigmoid classifier) whose probabilistic inference on test spectra is being benchmarked for accuracy and runtime) — https://github.com/wabdelmoula/massNet
- **Keras** (Deep learning platform in which the massNet architecture is implemented and through which test-set inference is executed)
- **tensorflow** (Backend for Keras used to run probabilistic classification of each test spectrum)
- **Python** (Implementation language for the evaluation script computing metrics and runtime)

## Examples

```
model.predict_proba(X_test)  # massNet sigmoid layer over 12,659 withheld HDF5 spectra; then compute confusion_matrix, roc_auc_score per class, and time.perf_counter() for runtime
```

## Evaluation signals

- Metrics are computed over the full independent test set: 12,659 spectra total (9,798 normal + 2,861 tumor)
- Reproduced per-class ROC AUC values are consistent with the reported 95.01% (normal) and 94.94% (tumor)
- Reproduced accuracy exceeds the SVM benchmark's accuracy as reported
- Measured classification runtime shows the reported ~174-fold speed gain relative to SVM
- Confusion matrix, precision, recall, and F1-score are computed per class and written to massnet_test_metrics.csv; ROC curve saved as roc_curve.png

## Limitations

- massNet demonstrated only binary (normal vs. tumor) classification; multi-class classification is left for future development
- Ground truth annotation based solely on annotated histology is an approximation and may be suboptimal for intra-tumor heterogeneity
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text (the README lists Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0)

## Evidence

- [abstract] Test set evaluation with AUC and speed comparison: "the results demonstrate higher accuracy and a 174-fold gain in speed compared to the established classical machine learning method, support vector machine."
- [intro] Model stability and speed/robustness assessment on a test set: "The model stability was evaluated using 5-fold cross validation, and the classification speed and robustness were assessed on a test set using various established machine learning metrics such as"
- [results] Test set composition: "This test set encompasses mass spectra with 85,062 m/z variables that were collected from two different classes of normal and tumor tissue types with total number of spectra 9,798 and 2,861"
- [results] Implementation platform: "The architecture was implemented in python using the publicly available deep learning platforms of Keras (Chollet, 2017) and tensorflow (Abadi et al., 2016)"
- [methods] HDF5 input format: "exported from SCiLS lab 2020a (Bruker, Bremen, Germany) in the standardized format imzML (Race et al., 2012) and converted to the HDF5 format (Folk et al., 2011) for deep learning analysis"
- [readme] Software versions from README: "2- Keras (2.2.0) with a Tensorflow(1.8.0) backend."
