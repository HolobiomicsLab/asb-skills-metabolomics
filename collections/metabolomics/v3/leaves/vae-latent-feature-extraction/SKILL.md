---
name: vae-latent-feature-extraction
description: Use when you have raw, high-dimensional MSI spectra (e.g. 85,062 m/z
  variables in HDF5) and need a low-dimensional latent embedding (5-D Code layer)
  for classification, UMAP visualization, or feature extraction.
when_to_use_negative:
- When the goal is multi-class tumor classification or precise tumor-margin ground
  truthing — the demonstrated model only covers binary normal-vs-tumor classification
  and histology-based annotation is an approximation, especially with intra-tumor
  heterogeneity
- When input is already a reduced feature/peak table — the VAE is designed to operate
  on raw, TIC-normalized spectra without prior peak picking
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: massNet
  role: Deep learning VAE whose probabilistic encoder yields the 5-D latent variable
    at the Code layer and whose decoder spatial patterns are used to identify m/z
    peaks
  repo: https://github.com/wabdelmoula/massNet
- name: msiPL
  role: Applied on the decoder-identified spatial patterns to identify m/z peaks and
    compute Pearson correlation with the tumor-class prediction map
- name: UMAP
  role: Non-linear dimensionality reduction applied to the 5-D latent variable for
    2D visualization of normal vs tumor separation
- name: Keras
  role: Deep learning platform used to implement the VAE architecture
- name: h5py
  role: Python package (v2.7.1) for reading the HDF5-converted MSI data
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:3816625d745624ec6cd022d4265ecae96909af094ce0aa4538e62ae2ef12e81f
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
  - UMAP
  - h5py
  - massNet
  - msiPL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# vae-latent-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts a compact latent representation from a VAE trained on raw (non-peak-picked) MALDI FT-ICR mass spectrometry imaging data, enabling downstream visualization, peak identification, and tumor-correlated feature ranking without spectral preprocessing.

## When to use

Use when you have raw, high-dimensional MSI spectra (e.g. 85,062 m/z variables in HDF5) and need a low-dimensional latent embedding (5-D Code layer) for classification, UMAP visualization, or feature extraction.

## When NOT to use

- When the goal is multi-class tumor classification or precise tumor-margin ground truthing — the demonstrated model only covers binary normal-vs-tumor classification and histology-based annotation is an approximation, especially with intra-tumor heterogeneity
- When input is already a reduced feature/peak table — the VAE is designed to operate on raw, TIC-normalized spectra without prior peak picking

## Inputs

- HDF5-format MALDI FT-ICR MSI dataset (85,062 m/z variables; e.g. 10,024 normal and 3,644 tumor training spectra from 4 intracranial GBM PDX models)
- imzML data exported from SCiLS lab 2020a prior to HDF5 conversion
- Trained massNet VAE model (Keras/TensorFlow) with 5-dimensional Code layer

## Outputs

- 5-dimensional latent variable (Code layer output) per spectrum
- 2D UMAP embedding of the latent variable colored by normal vs tumor class (Figure 3-style PNG)
- Tumor-correlated m/z peak table (m/z, Pearson correlation, formula, adduct, database, ppm error) with correlations >= 0.7

## How to apply

Convert the imzML-exported MALDI FT-ICR MSI data to HDF5 and feed the TIC-normalized spectra directly into the trained massNet VAE (implemented in Python 3.6.12 with Keras 2.2.0 / Tensorflow 1.8.0), skipping prior preprocessing and peak picking. Run the trained probabilistic encoder forward to obtain the 5-dimensional latent variable captured at the Code layer; training should show stable convergence over ~100 epochs (~55.7 min) with a total reconstruction MSE of 5.69e-8. Apply UMAP to this 5-D latent space (not the raw spectra, to avoid the curse of dimensionality) to visualize normal vs tumor separation. For feature extraction, use the msiPL method on the decoder's spatial patterns and Pearson correlation to identify m/z peaks, retaining highly co-localized peaks at Pearson correlation coefficient >= 0.7.

## Related tools

- **massNet** (Deep learning VAE whose probabilistic encoder yields the 5-D latent variable at the Code layer and whose decoder spatial patterns are used to identify m/z peaks) — https://github.com/wabdelmoula/massNet
- **msiPL** (Applied on the decoder-identified spatial patterns to identify m/z peaks and compute Pearson correlation with the tumor-class prediction map)
- **UMAP** (Non-linear dimensionality reduction applied to the 5-D latent variable for 2D visualization of normal vs tumor separation)
- **Keras** (Deep learning platform used to implement the VAE architecture)
- **h5py** (Python package (v2.7.1) for reading the HDF5-converted MSI data)

## Examples

```
import h5py, numpy as np; X = h5py.File('msi_data.h5','r')['spectra'][:]; z = encoder.predict(X)  # 5-dim Code layer latent variable
```

## Evaluation signals

- Latent variable extracted at the Code layer has exactly 5 dimensions
- VAE reconstruction achieves a total mean squared error of approximately 5.69e-8 on TIC-normalized MSI data
- 2D UMAP embedding of the latent variable visually separates normal and tumor spectra clusters
- Peak table contains only features with Pearson correlation >= 0.7 against the tumor-class prediction map
- Table 2-style assignments reproduced with known ppm errors, e.g. C9H13N2O9P [M+2K-H]+ at -2.1 ppm (HMDB) and C10H16N5O13P3 [M+K]+ at 0.36 ppm (Metlin)

## Limitations

- Demonstrated only for binary (normal vs tumor) classification; multi-class extension is left to future development
- Ground truth annotation based solely on annotated histology is an approximation that may not capture intra-tumor heterogeneity or tumor margins
- Reproducibility gap: no software version numbers or hyperparameter values for the trained model are reported in the article text (versions only appear in the repository README)

## Evidence
