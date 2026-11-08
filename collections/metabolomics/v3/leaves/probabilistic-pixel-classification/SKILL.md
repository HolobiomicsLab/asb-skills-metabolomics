---
name: probabilistic-pixel-classification
description: Use when raw MALDI-MSI pixel spectra (e.g. HDF5-format FT-ICR MSI with
  tens of thousands of m/z variables) must be classified normal vs tumor per pixel
  with probabilistic scores, and spatially mapped uncertainty at tumor margins is
  the target output.
when_to_use_negative:
- Multi-class tissue classification is required — the demonstrated model is binary
  (normal vs tumor); multi-class is left for future development
- Input is already a reduced peak/feature table or requires interpretable picked peaks
  — massNet operates on full raw spectra without prior preprocessing and peak picking
- Ground-truth annotation cannot be established from annotated histology — for other
  tissues/tumor types the bottleneck becomes establishing ground truth annotation
  of the MSI data
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3444
tools:
- name: massNet
  role: Deep learning model performing probabilistic pixel-based classification directly
    from raw mass spectral data
  repo: github.com/wabdelmoula/massNet
- name: Keras
  role: Platform in which the massNet architecture is implemented and from which probabilistic
    predictions are obtained
- name: tensorflow
  role: Backend for the Keras implementation of massNet
- name: Python
  role: Implementation language (3.6.12) for the massNet model and spatial reconstruction
    of predictions
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:d3dbbaa92a0a978a9b6bb65e6fe41d95111f732764addb6ce627bf9d918b2648
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
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9736842105263158
n_claims: 38
---

# probabilistic-pixel-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A deep learning skill (massNet) that performs probabilistic pixel-based classification directly from raw, high-dimensional mass spectrometry imaging spectra — no prior preprocessing or peak picking — yielding per-pixel normal/tumor probabilities and spatial uncertainty maps for tumor delineation.

## When to use

Use when raw MALDI-MSI pixel spectra (e.g. HDF5-format FT-ICR MSI with tens of thousands of m/z variables) must be classified normal vs tumor per pixel with probabilistic scores, and spatially mapped uncertainty at tumor margins is the target output.

## When NOT to use

- Multi-class tissue classification is required — the demonstrated model is binary (normal vs tumor); multi-class is left for future development
- Input is already a reduced peak/feature table or requires interpretable picked peaks — massNet operates on full raw spectra without prior preprocessing and peak picking
- Ground-truth annotation cannot be established from annotated histology — for other tissues/tumor types the bottleneck becomes establishing ground truth annotation of the MSI data

## Inputs

- HDF5-format MALDI FT-ICR MSI data (converted from imzML; e.g. 85,062 m/z variables), TIC-normalized, without peak picking
- Pathologist-annotated H&E labels transferred onto the MSI pixel grid (training/test spectra labels)
- Trained massNet Keras/Tensorflow model

## Outputs

- Per-spectrum probabilistic class scores (0–1) for normal vs tumor
- Per-pixel predicted class labels
- Spatially reconstructed probability maps in image space
- Per-pixel classification uncertainty maps revealing elevated uncertainty at tumor margins

## How to apply

Export MSI data from SCiLS lab in imzML and convert to HDF5 for deep learning analysis; do not peak-pick or preprocess — massNet learns predictive features directly from the full spectra (85,062 m/z variables in the GBM PDX study). Train on pathologist-annotated spectra (10,024 normal and 3,644 tumor training spectra from four intracranial GBM PDX models), implementing the architecture in Python 3.6.12 with Keras 2.2.0 on a Tensorflow 1.8.0 backend; regularization relies on batch normalization and dropout for stable, fast optimization. Apply the trained model to test spectra to obtain a per-spectrum probabilistic estimate (0 to 1) of the predicted class, then reconstruct predicted labels and probabilities back into image space. Compute per-pixel classification uncertainty from the probabilistic outputs and map it back spatially; correct output shows higher uncertainty at the tumor margin than in tumor core or normal tissue, consistent with mixed infiltrative-edge composition. Benchmark against SVM to confirm higher accuracy and the reported 174-fold speed gain.

## Related tools

- **massNet** (Deep learning model performing probabilistic pixel-based classification directly from raw mass spectral data) — github.com/wabdelmoula/massNet
- **Keras** (Platform in which the massNet architecture is implemented and from which probabilistic predictions are obtained)
- **tensorflow** (Backend for the Keras implementation of massNet)
- **Python** (Implementation language (3.6.12) for the massNet model and spatial reconstruction of predictions)

## Evaluation signals

- Spatially reconstructed uncertainty maps show higher classification uncertainty at tumor margins than in tumor core or normal tissue
- Per-spectrum probabilistic outputs fall within [0, 1] and predicted labels spatially align with pathologist annotations transferred from consecutive H&E sections
- Cross-validation/test accuracy exceeds the SVM benchmark on the same spectra (9,798 normal and 2,861 tumor test spectra)
- Runtime on the test set is consistent with the reported 174-fold speed gain over SVM
- Batch-normalization/dropout regularization yields stable convergence during optimization (VAE module reference: stable convergence in 100 epochs, MSE 5.69 × 10^-8 on training reconstruction)

## Limitations

- Only binary class classification is demonstrated; multi-class classification is left for future development
- Annotating MSI data based solely on annotated histology is an approximation that may not be optimal, especially with intra-tumor heterogeneity
- Tumor margin ground-truth annotation is challenging and would require more pathological investigation; the paper's section text does not report software version numbers or hyperparameter values (the README lists Python 3.6.12, Keras 2.2.0, Tensorflow 1.8.0)

## Evidence
