---
name: umap-dimensionality-reduction-visualization
description: Use when a trained VAE has produced a low-dimensional latent variable
  (e.g. 5-dim Code layer output) for tens of thousands of MALDI FT-ICR MSI spectra
  and you need a 2D embedding to visually assess normal/tumor class separation.
when_to_use_negative:
- When the goal is classification accuracy metrics (ROC, accuracy, recall, precision,
  F1) — use the massNet classifier evaluation on the test set instead; UMAP here is
  only a visualization of the training-set latent space
- When you only have raw mass spectra (85,062 m/z variables) and no trained encoder
  — applying UMAP directly to the high-dimensional spectra is exactly the curse-of-dimensionality
  situation this skill avoids
- When multi-class or quantitative cluster statistics are required — the paper demonstrates
  only binary normal vs tumor visualization and notes multi-class extension is future
  work
tools:
- name: UMAP
  role: Non-linear dimensionality reduction applied to the 5-dimensional Code-layer
    latent variable to produce the 2D embedding
- name: massNet
  role: Trained VAE whose probabilistic encoder supplies the 5-dimensional latent
    variable that UMAP is fit on
  repo: github.com/wabdelmoula/massNet
- name: Keras
  role: Platform in which the massNet VAE encoder generating the latent variable is
    implemented
- name: tensorflow
  role: Backend for the Keras-implemented massNet VAE used to compute the latent variable
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btac032
    title: 'massNet: integrated processing and classification of spatially resolved
      mass spectrometry data using deep learning for rapid tumor delineation'
  source_content_hash: sha256:fee0d200e612f202f0000abb0784a2d9a58d476e8da4dab89fb4946d43ef025e
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# umap-dimensionality-reduction-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies UMAP, a non-linear dimensionality reduction method, to the 5-dimensional latent variable captured at the Code layer of the trained massNet VAE to embed MALDI FT-ICR mass spectra in a 2D space for visualizing normal vs tumor class separation.

## When to use

Use when a trained VAE has produced a low-dimensional latent variable (e.g. 5-dim Code layer output) for tens of thousands of MALDI FT-ICR MSI spectra and you need a 2D embedding to visually assess normal/tumor class separation.

## When NOT to use

- When the goal is classification accuracy metrics (ROC, accuracy, recall, precision, F1) — use the massNet classifier evaluation on the test set instead; UMAP here is only a visualization of the training-set latent space
- When you only have raw mass spectra (85,062 m/z variables) and no trained encoder — applying UMAP directly to the high-dimensional spectra is exactly the curse-of-dimensionality situation this skill avoids
- When multi-class or quantitative cluster statistics are required — the paper demonstrates only binary normal vs tumor visualization and notes multi-class extension is future work

## Inputs

- 5-dimensional latent variable extracted at the massNet VAE Code layer for all spectra (shape ~13,668 x 5)
- Binary ground-truth class labels (normal vs tumor) per spectrum, transferred from pathologist-annotated H&E images
- Trained massNet VAE model implemented in Python with Keras 2.2.0 / TensorFlow 1.8.0 backend

## Outputs

- 2D UMAP embedding matrix (one (x,y) coordinate per spectrum)
- Figure 3-style 2D scatter plot (PNG) colored by normal vs tumor label and inspectable by GBM PDX model

## How to apply

First run the trained massNet probabilistic encoder forward to extract the 5-dimensional latent variable captured at the Code layer for every spectrum; do NOT apply UMAP to the raw spectra, which have 85,062 m/z variables, since the rationale is to avoid the curse of dimensionality by embedding the learned representation instead. Fit UMAP on this 5-dimensional latent matrix to produce a 2D embedding of each spectrum (10,024 normal and 3,644 tumor training spectra from the 4 intracranial GBM PDX models). Render the 2D map coloring each point by its binary ground-truth class label (normal vs tumor) so inter-tumor heterogeneity across the GBM PDX models can be inspected, as done for Figure 3. Validate by exporting the figure as a PNG and confirming by expert visual review that normal and tumor spectra form separable clusters in the 2D embedding, consistent with the paper's reported latent-space separation at the Code layer.

## Related tools

- **UMAP** (Non-linear dimensionality reduction applied to the 5-dimensional Code-layer latent variable to produce the 2D embedding)
- **massNet** (Trained VAE whose probabilistic encoder supplies the 5-dimensional latent variable that UMAP is fit on) — github.com/wabdelmoula/massNet
- **Keras** (Platform in which the massNet VAE encoder generating the latent variable is implemented)
- **tensorflow** (Backend for the Keras-implemented massNet VAE used to compute the latent variable)

## Examples

```
import umap; emb2d = umap.UMAP(n_components=2).fit_transform(code_layer_latent_5d)
```

## Evaluation signals

- A PNG figure of the 2D UMAP map exists as the final artifact (Figure 3 analog)
- Normal and tumor spectra form visually separable clusters in the 2D embedding on expert review
- UMAP was fit on the 5-dimensional Code-layer latent variable, not on the 85,062 m/z raw spectra
- Every training spectrum (10,024 normal + 3,644 tumor) appears as one point in the embedding
- Coloring by GBM PDX model allows inspection of inter-tumor heterogeneity within the class clusters

## Limitations

- Separation is assessed by expert visual review of the 2D map rather than a quantitative metric, so the validation is subjective
- The visualization reflects the training set's latent space; it does not itself measure test-set classification performance
- Ground-truth labels derive from pathologist-annotated H&E, which the authors note is an approximation that may not capture intra-tumor heterogeneity
- The paper demonstrates the workflow only for binary normal vs tumor classification; multi-class visualization/extension is left to future development

## Evidence
