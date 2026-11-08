---
name: variational-autoencoder-training
description: Use when you need to learn a compressed latent representation of high-dimensional,
  TIC-normalized MSI spectra stored in HDF5 (tens of thousands of m/z variables) as
  the probabilistic feature layer of a pixel-based tissue classification model.
when_to_use_negative:
- The analysis goal is multi-class tissue classification — the article demonstrates
  only binary (normal vs. tumor) classification and defers multi-class to future development.
- Input data has already been peak-picked or reduced to a feature/peak table — this
  skill is designed to learn directly from full, raw TIC-normalized spectra to avoid
  user preprocessing bias.
- Ground-truth annotations for the tissue/tumor type of interest cannot be reliably
  established (e.g., intra-tumor heterogeneity or tumor-margin cases requiring further
  pathological investigation).
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Deep learning architecture providing the VAE encoder/decoder with 5-dimensional
    latent variable for pixel-based probabilistic classification of MSI spectra
  repo: https://github.com/wabdelmoula/massNet
- name: Keras
  role: Framework used to implement the VAE encoder/decoder architecture (version
    2.2.0)
- name: tensorflow
  role: Backend for the Keras implementation of the VAE (version 1.8.0)
- name: Python
  role: Implementation language for the VAE training pipeline (version 3.6.12)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:d02b06473d8845b879e1d2bbfa890627fb8797ba9787261f47949eb79d81b920
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 24
---

# variational-autoencoder-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a probabilistic VAE (Keras/TensorFlow) on raw TIC-normalized MALDI FT-ICR mass spectrometry imaging spectra (85,062 m/z variables) to learn a 5-dimensional latent representation used for downstream tumor classification, with no prior preprocessing or peak picking.

## When to use

Use when you need to learn a compressed latent representation of high-dimensional, TIC-normalized MSI spectra stored in HDF5 (tens of thousands of m/z variables) as the probabilistic feature layer of a pixel-based tissue classification model.

## When NOT to use

- The analysis goal is multi-class tissue classification — the article demonstrates only binary (normal vs. tumor) classification and defers multi-class to future development.
- Input data has already been peak-picked or reduced to a feature/peak table — this skill is designed to learn directly from full, raw TIC-normalized spectra to avoid user preprocessing bias.
- Ground-truth annotations for the tissue/tumor type of interest cannot be reliably established (e.g., intra-tumor heterogeneity or tumor-margin cases requiring further pathological investigation).

## Inputs

- TIC-normalized MALDI FT-ICR MSI spectra in HDF5 format (e.g., 85,062 m/z variables; 10,024 normal and 3,644 tumor training spectra from four intracranial GBM PDX models)
- Pathologist-derived pixel class annotations (normal vs. tumor) transferred from annotated H&E images
- 80/20 train/validation split of the annotated spectra

## Outputs

- Trained VAE model weights (Keras/TensorFlow model)
- 5-dimensional latent variable (Code layer) per spectrum for downstream classification
- Reconstructed TIC-normalized MSI spectra
- JSON file recording the training reconstruction MSE (target 5.69 × 10^-8)

## How to apply

Load TIC-normalized spectra in HDF5 format (converted from imzML exported from SCiLS lab 2020a) — in the massNet GBM PDX case, 10,024 normal and 3,644 tumor spectra with 85,062 m/z variables plus pathologist-derived pixel annotations. Implement the VAE in Python using Keras (2.2.0) with a TensorFlow (1.8.0) backend: a probabilistic encoder and decoder with a 5-dimensional latent variable sampled from a normal distribution at the Code layer, regularized with batch normalization and dropout. Optimize encoder/decoder parameters by maximizing the variational lower bound over 100 epochs with random shuffling of data batches, repeated under 5-fold cross-validation to assess learning stability. Use the optimized decoder to reconstruct the original TIC-normalized spectra and compute the total mean squared error (target: 5.69 × 10^-8); save model weights and a JSON file recording the training MSE. The rationale is that learning directly from raw spectra avoids bias from user-parameterized spectral preprocessing and peak picking.

## Related tools

- **massNet** (Deep learning architecture providing the VAE encoder/decoder with 5-dimensional latent variable for pixel-based probabilistic classification of MSI spectra) — https://github.com/wabdelmoula/massNet
- **Keras** (Framework used to implement the VAE encoder/decoder architecture (version 2.2.0))
- **tensorflow** (Backend for the Keras implementation of the VAE (version 1.8.0))
- **Python** (Implementation language for the VAE training pipeline (version 3.6.12))

## Examples

```
python -c "import keras; vae = build_massnet_vae(latent_dim=5); vae.fit(X_train, y=None, epochs=100, shuffle=True)"
```

## Evaluation signals

- Training reconstruction MSE on TIC-normalized spectra reaches ≈ 5.69 × 10^-8 (the reported target accuracy).
- VAE optimization shows stable convergence distribution over 100 epochs with random batch shuffling (reported total running time ≈ 55.7 minutes).
- The Code layer captures a 5-dimensional latent variable; model summary must confirm latent_dim = 5.
- 5-fold cross-validation on the training set yields consistent learning stability across folds.
- Saved artifacts include both the trained model weights and a JSON file whose recorded MSE matches the reproduced value.

## Limitations

- The paper reports no software version numbers or hyperparameter values in its text (only README versions: Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0), so exact hyperparameter reproduction may require inspection of the repository code.
- Only binary (normal vs. tumor) classification is demonstrated; multi-class extension is left for future development.
- Ground-truth labels derived solely from annotated histology are an approximation and may be suboptimal for intra-tumor heterogeneity; establishing ground truth for other tissues/tumor types (e.g., tumor margins) remains challenging.
- The reference stack is old (Keras 2.2.0 / TensorFlow 1.8.0), which may complicate use with modern environments.

## Evidence
