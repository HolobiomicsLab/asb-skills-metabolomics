---
name: mass-spectrometry-imaging-spectra-embedding
description: Use when a trained massNet-style VAE over high-dimensional MALDI MSI
  spectra (e.g., 85,062 m/z variables) exists and you need to inspect its 5-D latent
  space or check normal-vs-tumor separability in a 2D UMAP map.
when_to_use_negative:
- No trained encoder is available and you would have to run UMAP directly on the raw
  85,062-dimensional spectra — the paper deliberately embeds the 5-D latent variable
  instead of the original MSI data to avoid the curse of dimensionality.
- The data are already reduced to a peak/feature table or low-dimensional representation
  — re-embedding with this skill is redundant.
- The goal is spectral preprocessing or peak picking (e.g., msiPL-based peak identification)
  rather than latent-space embedding — massNet consumes raw spectra without prior
  preprocessing and peak picking.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_0092
tools:
- name: massNet
  role: Trained VAE whose probabilistic encoder yields the 5-dimensional Code-layer
    latent variable that is embedded
  repo: https://github.com/wabdelmoula/massNet
- name: UMAP
  role: Non-linear dimensionality reduction applied to the 5-D latent variable to
    produce the 2D visualization colored by class label
- name: Keras
  role: Deep learning platform (v2.2.0) in which the VAE encoder is implemented and
    run forward (predict) on the spectra
- name: tensorflow
  role: Backend (v1.8.0) executing the Keras-defined probabilistic encoder during
    latent extraction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:5e8fb44d61e2cad991ce39cc5107826e19d031227722c2798e15f17870b34ac9
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
  - massNet
  - tensorflow
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# mass-spectrometry-imaging-spectra-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embeds high-dimensional MALDI FT-ICR mass spectra into a low-dimensional latent space by forwarding them through a trained massNet variational autoencoder (5-dimensional Code layer) and then visualizes the codes with UMAP to expose normal-vs-tumor separation and inter-tumor heterogeneity.

## When to use

Use when a trained massNet-style VAE over high-dimensional MALDI MSI spectra (e.g., 85,062 m/z variables) exists and you need to inspect its 5-D latent space or check normal-vs-tumor separability in a 2D UMAP map.

## When NOT to use

- No trained encoder is available and you would have to run UMAP directly on the raw 85,062-dimensional spectra — the paper deliberately embeds the 5-D latent variable instead of the original MSI data to avoid the curse of dimensionality.
- The data are already reduced to a peak/feature table or low-dimensional representation — re-embedding with this skill is redundant.
- The goal is spectral preprocessing or peak picking (e.g., msiPL-based peak identification) rather than latent-space embedding — massNet consumes raw spectra without prior preprocessing and peak picking.

## Inputs

- HDF5-format MSI spectral dataset converted from imzML (e.g., 85,062 m/z variables; TIC-normalized spectra)
- Trained massNet VAE model (Keras/TensorFlow) with a probabilistic encoder exposing a 5-dimensional Code layer
- Binary ground-truth class labels per spectrum (normal vs tumor), transferred from pathologist-annotated H&E images
- Optional per-spectrum GBM PDX model identifiers for coloring by tumor model

## Outputs

- 5-dimensional latent codes, one per spectrum, extracted at the VAE Code layer
- 2D UMAP embedding of the latent codes
- PNG visualization (analogous to Figure 3) of the 2D UMAP colored by class label and/or GBM PDX model

## How to apply

Load the HDF5-format, TIC-normalized spectra (converted from imzML; the paper's training set is 10,024 normal and 3,644 tumor spectra over 85,062 m/z variables from 4 intracranial GBM PDX models) together with the trained massNet VAE implemented in Keras 2.2.0 with a TensorFlow 1.8.0 backend (Python 3.6.12). Forward all spectra through the trained probabilistic encoder to extract the 5-dimensional latent variable captured at the Code layer; a properly trained model reconstructs the TIC-normalized input at a total MSE of 5.69 x 10^-8. Run UMAP on these 5-D latent codes rather than on the original 85,062-dimensional spectra, since the paper applies the non-linear dimensionality reduction to the learned latent variable specifically to avoid the curse of dimensionality. Color each 2D point by its binary ground-truth label (normal vs tumor, transferred from pathologist H&E annotations) and optionally by GBM PDX model (e.g., GBM 12_1, GBM 22_1, GBM 39_1) to reveal inter-tumor heterogeneity, reproducing Figure 3.

## Related tools

- **massNet** (Trained VAE whose probabilistic encoder yields the 5-dimensional Code-layer latent variable that is embedded) — https://github.com/wabdelmoula/massNet
- **UMAP** (Non-linear dimensionality reduction applied to the 5-D latent variable to produce the 2D visualization colored by class label)
- **Keras** (Deep learning platform (v2.2.0) in which the VAE encoder is implemented and run forward (predict) on the spectra)
- **tensorflow** (Backend (v1.8.0) executing the Keras-defined probabilistic encoder during latent extraction)

## Examples

```
code = encoder.predict(spectra_h5); emb = umap.UMAP(n_components=2).fit_transform(code)  # 5-D Code layer -> 2D UMAP
```

## Evaluation signals

- The encoder output has exactly 5 features per spectrum (the 5-dimensional latent variable at the Code layer reported in Supplementary Figure S2).
- Before embedding, the trained VAE should reconstruct the TIC-normalized input at a total MSE of approximately 5.69 x 10^-8.
- The output PNG exists and visually shows separable normal and tumor clusters when points are colored by binary ground-truth label (the Figure 3 acceptance criterion).
- Embedding row count matches the input spectra (10,024 normal + 3,644 tumor = 13,668 training spectra in the paper's set).
- Coloring points by GBM PDX model (e.g., GBM 12_1, GBM 22_1, GBM 39_1) should expose inter-tumor heterogeneity across the 4 models rather than one homogeneous tumor cluster.

## Limitations

- Separation was demonstrated only for binary normal-vs-tumor labels; the paper leaves multi-class classification to future development.
- Ground truth derives from pathologist annotations of H&E images transferred to the MSI data, an approximation that may not be optimal for intra-tumor heterogeneity.
- The article text reports no software version numbers or hyperparameter values for the trained model; reproducibility depends on the versions pinned in the README (Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0).

## Evidence
