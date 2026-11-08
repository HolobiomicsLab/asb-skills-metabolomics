---
name: tumor-normal-class-separation-visualization
description: Use when a trained probabilistic encoder yields a compact latent representation
  (e.g., 5-D from a VAE Code layer) of high-dimensional spectra and you need to visually
  confirm class separation between normal and tumor groups.
when_to_use_negative:
- The input is raw high-dimensional mass spectra rather than a learned low-dimensional
  latent variable — the article applies UMAP to the latent code specifically to avoid
  the curse of dimensionality, not directly on 85,062 m/z variables.
- The goal is quantitative classification or performance metrics (ROC, accuracy, F1-score)
  — that is the role of massNet's probabilistic pixel-based classifier, not this visualization
  step.
- No trained VAE/encoder exists yet; the Code-layer latent variable must first be
  extracted from an optimized model before UMAP can be applied.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: massNet
  role: Trained VAE whose probabilistic encoder provides the 5-dimensional latent
    variable at the Code layer that is embedded via UMAP
  repo: https://github.com/wabdelmoula/massNet
- name: UMAP
  role: Non-linear dimensionality reduction method that embeds the 5-D latent variable
    into 2D for visualization of class separation
- name: Keras
  role: Platform in which the VAE encoder is implemented and run forward to extract
    the latent variable
- name: tensorflow
  role: Backend for the Keras implementation used to run the trained encoder
- name: Python
  role: Implementation language for the encoder-forward pass and UMAP rendering pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:b0b3d68d42a617c675241d832c770dd579e5d0d27df691709c6ca2850ff28d65
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# tumor-normal-class-separation-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embeds a trained VAE's low-dimensional latent variable (e.g., massNet's 5-dimensional Code layer output from MALDI FT-ICR MSI spectra) in 2D via UMAP to visually verify separation between normal and tumor tissue classes and inspect inter-tumor heterogeneity.

## When to use

Use when a trained probabilistic encoder yields a compact latent representation (e.g., 5-D from a VAE Code layer) of high-dimensional spectra and you need to visually confirm class separation between normal and tumor groups.

## When NOT to use

- The input is raw high-dimensional mass spectra rather than a learned low-dimensional latent variable — the article applies UMAP to the latent code specifically to avoid the curse of dimensionality, not directly on 85,062 m/z variables.
- The goal is quantitative classification or performance metrics (ROC, accuracy, F1-score) — that is the role of massNet's probabilistic pixel-based classifier, not this visualization step.
- No trained VAE/encoder exists yet; the Code-layer latent variable must first be extracted from an optimized model before UMAP can be applied.

## Inputs

- HDF5-converted MALDI FT-ICR MSI dataset (e.g., 85,062 m/z variables; 10,024 normal and 3,644 tumor training spectra from 4 intracranial GBM PDX models)
- Trained massNet VAE model (Python, Keras 2.2.0 with TensorFlow 1.8.0 backend)
- Binary ground-truth class labels (normal vs tumor) per spectrum, transferred from pathologist-annotated H&E images

## Outputs

- 2D UMAP embedding of the 5-dimensional Code-layer latent variable for all spectra
- Figure rendered as a PNG with points colored by normal vs tumor label (and optionally by GBM PDX model), e.g., Figure 3

## How to apply

Load the HDF5-converted MALDI FT-ICR MSI dataset (85,062 m/z variables; 10,024 normal and 3,644 tumor training spectra from 4 intracranial GBM PDX models) together with the trained massNet VAE implemented in Python with Keras/TensorFlow. Run the trained probabilistic encoder forward on all spectra to extract the 5-dimensional latent variable captured at the Code layer. Apply UMAP to this 5-D latent variable rather than the original high-dimensional MSI data, avoiding the curse of dimensionality. Render the 2D UMAP embedding as a scatter map, coloring each point by its binary ground-truth class label (normal vs tumor), and optionally by GBM PDX model to expose inter-tumor heterogeneity. Validate by confirming the exported figure exists as a PNG and expert-reviewing that normal and tumor spectra form separable clusters.

## Related tools

- **massNet** (Trained VAE whose probabilistic encoder provides the 5-dimensional latent variable at the Code layer that is embedded via UMAP) — https://github.com/wabdelmoula/massNet
- **UMAP** (Non-linear dimensionality reduction method that embeds the 5-D latent variable into 2D for visualization of class separation)
- **Keras** (Platform in which the VAE encoder is implemented and run forward to extract the latent variable)
- **tensorflow** (Backend for the Keras implementation used to run the trained encoder)
- **Python** (Implementation language for the encoder-forward pass and UMAP rendering pipeline)

## Examples

```
import umap; embedding = umap.UMAP(n_components=2).fit_transform(encoder.predict(h5py.File('msi_data.h5','r')['spectra'])[:]); plt.scatter(embedding[:,0], embedding[:,1], c=labels); plt.savefig('Figure3_umap.png')
```

## Evaluation signals

- Output figure exists as a PNG file on disk.
- Expert visual review confirms normal and tumor spectra form separable clusters in the 2D UMAP embedding.
- Coloring by GBM PDX model shows interpretable inter-tumor heterogeneity within the class-separated clusters.
- The embedded values derive from the 5-dimensional Code-layer latent variable (consistent with Supplementary Figure S2), not from raw 85,062-variable spectra.

## Limitations

- Separation in the UMAP plot is a qualitative, expert-reviewed visual check rather than a quantitative classification metric.
- Ground truth labels are transferred from pathologist-annotated H&E histology, which the authors note is an approximation that may not be optimal, especially for intra-tumor heterogeneity.
- The underlying massNet model is demonstrated only for binary normal-vs-tumor classification; multi-class separation cannot be assessed this way.
- No software version numbers or hyperparameter values for the trained massNet model are reported in the paper text, limiting reproducibility (README lists Python 3.6.12, Keras 2.2.0, TensorFlow 1.8.0).

## Evidence
