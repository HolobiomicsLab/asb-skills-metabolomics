---
name: deep-learning-model-inference
description: Use when a trained massNet VAE must be applied to MSI spectra (HDF5-converted
  imzML, 85,062 m/z variables) to obtain latent codes or downstream 2D UMAP class-separation
  maps.
when_to_use_negative:
- No trained massNet VAE is available — training/optimization (100 epochs, 55.7 min)
  is a separate skill; inference requires existing validated weights
- Data are not mass spectra in imzML/HDF5 form or have not been TIC-normalized — the
  encoder expects the same representation it was trained on
- The goal is peak-level interpretation rather than latent-space visualization — co-localized
  m/z peak selection at Pearson correlation ≥ 0.7 via msiPL is the appropriate path
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: massNet
  role: Provides the trained VAE whose probabilistic encoder is run forward to extract
    the 5-dim latent variable at the Code layer
  repo: github.com/wabdelmoula/massNet
- name: Keras
  role: Deep learning platform in which the VAE encoder is implemented and executed
    for inference
- name: tensorflow
  role: Backend that executes the Keras encoder forward pass on the spectra
- name: UMAP
  role: Non-linear dimensionality reduction applied to the 5-dim latent variable to
    produce the 2D visualization
- name: Python
  role: Language in which the inference and UMAP embedding pipeline is implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:1e31729217b07caf82b807c8e41bdea7f109a5601512bc08b64e14f3aed95551
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
  - UMAP
  - massNet
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# deep-learning-model-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs a trained massNet VAE forward (via Keras/TensorFlow) to extract the 5-dimensional latent variable at the Code layer from TIC-normalized MALDI FT-ICR mass spectra, then embeds it with UMAP for 2D visualization of normal vs tumor separation.

## When to use

Use when a trained massNet VAE must be applied to MSI spectra (HDF5-converted imzML, 85,062 m/z variables) to obtain latent codes or downstream 2D UMAP class-separation maps.

## When NOT to use

- No trained massNet VAE is available — training/optimization (100 epochs, 55.7 min) is a separate skill; inference requires existing validated weights
- Data are not mass spectra in imzML/HDF5 form or have not been TIC-normalized — the encoder expects the same representation it was trained on
- The goal is peak-level interpretation rather than latent-space visualization — co-localized m/z peak selection at Pearson correlation ≥ 0.7 via msiPL is the appropriate path

## Inputs

- HDF5-converted MALDI FT-ICR MSI dataset (85,062 m/z variables; imzML originally exported from SCiLS lab 2020a and converted to HDF5)
- Trained massNet VAE model with the 5-dimensional Code layer (Keras/Python/TensorFlow)
- Binary ground-truth class labels (normal vs tumor) per spectrum, optionally with GBM PDX model identity (e.g. GBM 12_1, GBM 22_1, GBM 39_1)

## Outputs

- 5-dimensional latent variable at the Code layer for every spectrum (numpy array via the probabilistic encoder)
- 2D UMAP embedding (Figure 3) colored by normal vs tumor class label and by GBM model, exported as PNG

## How to apply

Load the HDF5-converted MALDI 9.4 Tesla FT-ICR MSI data (e.g. the training set of 10,024 normal and 3,644 tumor spectra from 4 intracranial GBM PDX models) together with the trained massNet VAE implemented in Python using Keras (2.2.0) with a TensorFlow (1.8.0) backend. Run the trained probabilistic encoder forward on the TIC-normalized spectra to extract the 5-dimensional latent variable captured at the Code layer, rather than applying UMAP to the raw 85,062-dimensional spectra — this avoids the curse of dimensionality. Then apply UMAP, a non-linear dimensionality reduction method, on the learned 5-dim latent codes to embed each spectrum in 2D and color points by ground-truth class label (normal vs tumor) and by GBM PDX model to inspect inter-tumor heterogeneity. The VAE was optimized with stable convergence in 100 epochs (55.7 min total running time) and reconstructs the training data with a total MSE of 5.69 × 10^-8, so the encoder inference step inherits those validated weights.

## Related tools

- **massNet** (Provides the trained VAE whose probabilistic encoder is run forward to extract the 5-dim latent variable at the Code layer) — github.com/wabdelmoula/massNet
- **Keras** (Deep learning platform in which the VAE encoder is implemented and executed for inference)
- **tensorflow** (Backend that executes the Keras encoder forward pass on the spectra)
- **UMAP** (Non-linear dimensionality reduction applied to the 5-dim latent variable to produce the 2D visualization)
- **Python** (Language in which the inference and UMAP embedding pipeline is implemented)

## Evaluation signals

- The 2D UMAP output exists as a PNG (Figure 3) and normal vs tumor spectra form visually separable clusters
- Latent extraction returns a 5-dimensional vector per spectrum, matching the Code layer's 5-dim latent variable reported for the optimized model
- UMAP is computed on the 5-dim latent codes, not on the 85,062-dim raw spectra
- Encoder inference reproduces the validated model's reconstruction behavior (training data total MSE of 5.69 × 10^-8) rather than degraded codes
- Environment versions match the README (Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0 backend) to avoid weight-incompatibility failures

## Limitations

- massNet was demonstrated for binary (normal vs tumor) classification only; multi-class classification is left for future development, so latent separation beyond two classes is untested
- Ground truth labels derived solely from pathologist-annotated H&E histology are an approximation and may not be optimal for intra-tumor heterogeneity
- No software version numbers or hyperparameter values for the trained model are reported in the paper text; the README gives versions (Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0, numpy 1.15.4, sklearn 0.23.2, scipy 1.0.0, seaborn 0.9.0, pandas 1.1.1, h5py 2.7.1) but exact UMAP settings must be established
- massNet code is shared under the 3D Slicer Software License agreement, which constrains reuse terms

## Evidence
