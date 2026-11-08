---
name: msi-spectrum-binary-classification-inference
description: Use when you have raw MALDI FT-ICR MSI spectra (e.g., 85,062 m/z variables)
  with ground-truth normal/tumor labels and a trained massNet model, and need probabilistic
  binary pixel-level classification plus ROC AUC, accuracy, confusion matrix, and
  runtime.
when_to_use_negative:
- The task requires multi-class tissue classification — the article demonstrates only
  binary (normal/tumor) classification and explicitly leaves multi-class extension
  to future development
- The input is already a preprocessed, peak-picked feature table — massNet's core
  advantage is probabilistic pixel-based classification directly from raw high-dimensional
  mass spectral data without prior peak picking
- No trained massNet model or reproducible ground-truth annotation is available —
  the article notes ground-truth annotation of MSI data is a challenging task for
  other tissues and tumor types
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3366
- http://edamontology.org/topic_0091
tools:
- name: massNet
  role: Deep learning model performing probabilistic pixel-based classification directly
    from raw mass spectral data; provides the VAE encoder and sigmoid classifier used
    for inference
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Deep learning platform in which the massNet architecture is implemented and
    from which inference (model.predict) is run on test spectra
- name: tensorflow
  role: Backend for the Keras implementation of massNet used during probabilistic
    inference
- name: Python
  role: Implementation language for loading HDF5 spectra, running inference, and computing
    evaluation metrics (sklearn, numpy)
- name: SCiLS lab 2020a
  role: Upstream export of MSI data in imzML format, subsequently converted to HDF5
    for deep learning analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:dc1f745f122380b2c5cc08f63cf77be536dd8d8410a826d0f8c9d84928e58ea8
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
  - SCiLS lab 2020a
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3366
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# msi-spectrum-binary-classification-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs the trained massNet deep learning model (VAE encoder plus sigmoid-activated classifier on a 5-dimensional latent variable) directly on raw MALDI FT-ICR MSI spectra to obtain normal/tumor class probabilities without any prior preprocessing or peak picking, then evaluates robustness with standard classification metrics and runtime benchmarks.

## When to use

Use when you have raw MALDI FT-ICR MSI spectra (e.g., 85,062 m/z variables) with ground-truth normal/tumor labels and a trained massNet model, and need probabilistic binary pixel-level classification plus ROC AUC, accuracy, confusion matrix, and runtime.

## When NOT to use

- The task requires multi-class tissue classification — the article demonstrates only binary (normal/tumor) classification and explicitly leaves multi-class extension to future development
- The input is already a preprocessed, peak-picked feature table — massNet's core advantage is probabilistic pixel-based classification directly from raw high-dimensional mass spectral data without prior peak picking
- No trained massNet model or reproducible ground-truth annotation is available — the article notes ground-truth annotation of MSI data is a challenging task for other tissues and tumor types

## Inputs

- Withheld test set of MALDI FT-ICR MSI spectra in HDF5 format (e.g., 9,798 normal and 2,861 tumor spectra; 85,062 m/z variables) from GBM PDX mouse brain tumor models
- Ground-truth normal/tumor labels transferred from expert pathologist annotations of H&E images
- Trained massNet model (VAE encoder + sigmoid-activated classifier on 5-dimensional latent variable, hyper-parameter set H = {phi, theta}) in Keras/TensorFlow

## Outputs

- Per-spectrum predicted class probabilities (normal vs. tumor) from the sigmoid output layer
- Confusion matrix, accuracy, per-class precision, recall, and F1-score
- Per-class ROC AUC values and ROC curve plot (roc_curve.png)
- Classification runtime (spectra per unit time) with SVM comparison
- Metrics table written to massnet_test_metrics.csv

## How to apply

Load the withheld test set of MALDI FT-ICR MSI spectra (e.g., 9,798 normal and 2,861 tumor spectra with 85,062 m/z variables) in HDF5 format together with ground-truth labels. Load the trained massNet model — a VAE encoder with a 5-dimensional latent variable at the Code layer and a sigmoid-activated output layer — implemented in Keras/TensorFlow. Run probabilistic inference on every spectrum through the sigmoid output layer to obtain predicted class probabilities for the two classes, deliberately applying no prior preprocessing or peak picking (this is the skill's defining rationale: avoiding bias from user-optimized spectral preprocessing parameters). Compute the confusion matrix, accuracy, per-class precision, recall, F1-score, and per-class AUC from the ROC curve over the full independent test set. Finally, measure classification runtime (spectra per unit time) and compare against an SVM benchmark, against which massNet reported higher accuracy and a 174-fold speed gain. Write the metrics table to massnet_test_metrics.csv and save the ROC curve as roc_curve.png.

## Related tools

- **massNet** (Deep learning model performing probabilistic pixel-based classification directly from raw mass spectral data; provides the VAE encoder and sigmoid classifier used for inference) — https://github.com/wabdelmoula/massNet
- **Keras** (Deep learning platform in which the massNet architecture is implemented and from which inference (model.predict) is run on test spectra)
- **tensorflow** (Backend for the Keras implementation of massNet used during probabilistic inference)
- **Python** (Implementation language for loading HDF5 spectra, running inference, and computing evaluation metrics (sklearn, numpy))
- **SCiLS lab 2020a** (Upstream export of MSI data in imzML format, subsequently converted to HDF5 for deep learning analysis)

## Examples

```
import h5py, tensorflow as tf; X = h5py.File('test_set.h5','r')['spectra'][:]; probs = massnet_model.predict(X)  # sigmoid output -> normal/tumor probabilities
```

## Evaluation signals

- The evaluation must cover the full independent test set of 12,659 spectra (9,798 normal + 2,861 tumor) with 85,062 m/z variables
- Per-class ROC AUC should approach the reported values of 95.01% (normal) and 94.94% (tumor)
- Accuracy must be higher than the SVM benchmark, and measured classification runtime should reflect the reported 174-fold speed gain over SVM
- Predictions must be produced without any prior preprocessing or peak picking — verifying no peak-picking step was inserted before inference
- Outputs must include massnet_test_metrics.csv and roc_curve.png with a confusion matrix and per-class precision/recall/F1

## Limitations

- Only binary (normal/tumor) classification is demonstrated; multi-class classification is left for future development
- Ground-truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity; tumor margin annotation would require more pathological investigation
- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text (README specifies Python 3.6.12, Keras 2.2.0 with TensorFlow 1.8.0 backend, numpy 1.15.4, sklearn 0.23.2, scipy 1.0.0, seaborn 0.9.0, pandas 1.1.1, h5py 2.7.1)

## Evidence
