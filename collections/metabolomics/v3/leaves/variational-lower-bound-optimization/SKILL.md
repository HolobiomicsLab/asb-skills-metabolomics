---
name: variational-lower-bound-optimization
description: Use when high-dimensional, TIC-normalized mass spectrometry imaging spectra
  (e.g. 85,062 m/z variables from MALDI FT-ICR MSI) must be compressed into a low-dimensional
  latent space and reconstructed to a target MSE, without prior peak picking or preprocessing.
when_to_use_negative:
- The goal is supervised pixel classification (tumor vs. normal) rather than unsupervised
  spectral reconstruction — use the full massNet classification pipeline with benchmarking
  against SVM instead.
- Input spectra are raw and unnormalized or require prior preprocessing/peak picking
  — massNet's VAE operates directly on TIC-normalized data without prior preprocessing.
- The task requires multi-class tissue classification — the article demonstrates only
  binary class classification and leaves multi-class extension to future development.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning framework containing the VAE module whose encoder/decoder parameters
    are optimized by maximizing the variational lower bound on MSI spectra
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Deep learning platform used to implement the VAE architecture (probabilistic
    encoder/decoder) in Python
- name: tensorflow
  role: Backend for Keras executing the variational lower bound optimization over
    100 epochs with shuffled data batches
- name: Python
  role: Implementation language for the VAE training and reconstruction pipeline (Python
    3.6.12 per README)
- name: h5py
  role: Package for loading the HDF5-format TIC-normalized MSI spectra into the training
    pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:583152d2da426197eaa41082135e51b83449d016a791f2b5a4953b751b8d1a38
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
  - h5py
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 28
---

# variational-lower-bound-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a probabilistic encoder–decoder (variational autoencoder, as implemented in the massNet VAE module) by maximizing the variational lower bound over TIC-normalized MSI spectra, yielding a compact latent representation that reconstructs the original spectra at sub-1e-7 MSE.

## When to use

Use when high-dimensional, TIC-normalized mass spectrometry imaging spectra (e.g. 85,062 m/z variables from MALDI FT-ICR MSI) must be compressed into a low-dimensional latent space and reconstructed to a target MSE, without prior peak picking or preprocessing.

## When NOT to use

- The goal is supervised pixel classification (tumor vs. normal) rather than unsupervised spectral reconstruction — use the full massNet classification pipeline with benchmarking against SVM instead.
- Input spectra are raw and unnormalized or require prior preprocessing/peak picking — massNet's VAE operates directly on TIC-normalized data without prior preprocessing.
- The task requires multi-class tissue classification — the article demonstrates only binary class classification and leaves multi-class extension to future development.

## Inputs

- TIC-normalized MALDI FT-ICR MSI spectra in HDF5 format (85,062 m/z variables; training set of 10,024 normal and 3,644 tumor spectra from four intracranial GBM PDX models)
- 80/20 train/validation split of the spectra with pathologist-derived pixel annotations as class labels

## Outputs

- Trained VAE model weights (encoder/decoder parameters maximizing the variational lower bound)
- 5-dimensional latent variable sampled from a normal distribution at the Code layer
- Reconstructed TIC-normalized MSI spectra with total mean squared error of 5.69 × 10^-8
- JSON file recording the training reconstruction MSE

## How to apply

Load TIC-normalized spectra in HDF5 format (the massNet training set held 10,024 normal and 3,644 tumor spectra with 85,062 m/z variables from four intracranial GBM PDX models) and split 80% training / 20% validation. Implement the VAE in Python using Keras (2.2.0) with a TensorFlow (1.8.0) backend, with a probabilistic encoder and decoder and a 5-dimensional latent variable sampled from a normal distribution at the Code layer. Optimize encoder/decoder parameters by maximizing the variational lower bound over 100 epochs with random shuffling of data batches, repeating under 5-fold cross-validation on the training set to assess learning stability. Use the optimized decoder to reconstruct the original TIC-normalized MSI spectra from the learned latent variable and compute the total mean squared error; the article's target reconstruction accuracy is MSE = 5.69 × 10^-8. Save the trained model weights and a JSON recording the training reconstruction MSE to verify reproduction. Regularization via batch normalization and drop out maintains learning stabilization and faster optimization.

## Related tools

- **massNet** (Deep learning framework containing the VAE module whose encoder/decoder parameters are optimized by maximizing the variational lower bound on MSI spectra) — https://github.com/wabdelmoula/massNet
- **Keras** (Deep learning platform used to implement the VAE architecture (probabilistic encoder/decoder) in Python)
- **tensorflow** (Backend for Keras executing the variational lower bound optimization over 100 epochs with shuffled data batches)
- **Python** (Implementation language for the VAE training and reconstruction pipeline (Python 3.6.12 per README))
- **h5py** (Package for loading the HDF5-format TIC-normalized MSI spectra into the training pipeline)

## Examples

```
python train_vae.py --input tic_normalized_msi.h5 --latent_dim 5 --epochs 100 --folds 5 --split 80/20 --output_mse training_mse.json
```

## Evaluation signals

- Total reconstruction MSE on the TIC-normalized training spectra matches the reported value of 5.69 × 10^-8.
- VAE optimization shows stable convergence distribution across 100 epochs with random shuffling of data batches, with total running time on the order of the reported 55.7 minutes.
- The optimized model captures a 5-dimensional latent variable at the Code layer (verify latent dimensionality = 5).
- 5-fold cross-validation on the training set yields consistent convergence/stability across folds (reproducing massNet's training stability).
- Saved artifacts include trained model weights plus a JSON file whose recorded MSE is within tolerance of 5.69 × 10^-8.

## Limitations

- No software version numbers or hyperparameter values for the trained massNet model are reported in the article text; only the README specifies versions (Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0, numpy 1.15.4, sklearn 0.23.2, scipy 1.0.0, seaborn 0.9.0, Pandas 1.1.1, h5py 2.7.1).
- Ground truth annotation based solely on annotated histology is an approximation and may not be optimal, especially for intra-tumor heterogeneity.
- Demonstrated only on binary class classification of GBM PDX MALDI FT-ICR MSI data; applicability to other tissues/tumor types depends on establishing ground truth annotation.
- massNet code is shared under the 3D Slicer Software License agreement, which may constrain reuse.

## Evidence
