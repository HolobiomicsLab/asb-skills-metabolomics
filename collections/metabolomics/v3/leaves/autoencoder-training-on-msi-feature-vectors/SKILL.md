---
name: autoencoder-training-on-msi-feature-vectors
description: Use when you have a reconstructed, peak-binned and TIC-normalized 3D
  MSI dataset and need coherent per-section feature images with anatomical contrast
  to serve as moving images for multimodal MRI-atlas registration.
when_to_use_negative:
- You are analyzing a single 2D tissue section with no serial-section 3D volume to
  co-embed (single-pixel UMAP visualization suffices)
- You need supervised anatomical classification with gradient-boosted trees rather
  than feature-image generation for registration
- The MSI data are raw short-transient transients that have not yet undergone deep-learning
  signal reconstruction and peak binning/normalization
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MEISTER
  role: Framework containing the parametric UMAP autoencoder training and embedding
    code used to convert the 3D MSI hyperspectral data cube into feature images for
    MRI co-registration
  repo: https://github.com/richardxie1119/MEISTER
- name: embed_3DCoronal.ipynb / embed3D.ipynb
  role: Notebooks executing the parametric UMAP training on coronal and sagittal MSI
    data to obtain coherent feature images
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:6b3ed642f82fd6dc0f1e031e9428bb74370f8f185a2d4354b6e8a914489b6076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - embed_3DCoronal.ipynb / embed3D.ipynb
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# autoencoder-training-on-msi-feature-vectors

## Summary

Trains a parametric UMAP autoencoder on the m/z-feature vectors of reconstructed serial-section MSI data (e.g. coronal3D_propagated_decoded.h5) so all pixels across sections embed into one shared space, yielding the MRI-like grayscale feature images needed for 3D registration.

## When to use

Use when you have a reconstructed, peak-binned and TIC-normalized 3D MSI dataset and need coherent per-section feature images with anatomical contrast to serve as moving images for multimodal MRI-atlas registration.

## When NOT to use

- You are analyzing a single 2D tissue section with no serial-section 3D volume to co-embed (single-pixel UMAP visualization suffices)
- You need supervised anatomical classification with gradient-boosted trees rather than feature-image generation for registration
- The MSI data are raw short-transient transients that have not yet undergone deep-learning signal reconstruction and peak binning/normalization

## Inputs

- Reconstructed and processed 3D MSI dataset of serial sections (m/z-binned in 3-ppm increments, TIC-normalized), e.g. coronal3D_propagated_decoded.h5 from deposit 10.13012/B2IDB-9740536_V1
- Number of m/z features defining the autoencoder input/output dimensionality

## Outputs

- parametric_umap_feature_images.npy — one grayscale feature image per tissue section
- Trained parametric UMAP encoder/decoder model weights
- 3D UMAP embedding vectors for all pixels of every tissue section

## How to apply

Build a parametric UMAP autoencoder whose input and output size equals the number of m/z features, with fully connected layers of 256, 128 and 64 neurons: the encoder is trained with the UMAP cross-entropy loss and the decoder with a reconstruction loss. Randomly sample 10% of total pixels across all serial sections as training data and train for 20 epochs so that every pixel of the 3D volume embeds into a single shared embedding space (avoiding the incoherent per-section registration that 2D-section approaches suffer from). Run the trained encoder on all pixels of every tissue section, take the first three UMAP embedding dimensions per section as red, green and blue channels, then convert each RGB feature image to grayscale. Save the per-section stack as parametric_umap_feature_images.npy and feed these grayscale feature images as moving images into the subsequent affine and B-spline registration against the corresponding Waxholm Space rat brain MRI slice. The implementation lives in the embed_3D.ipynb / embed_3DCoronal.ipynb notebooks of richardxie1119/multiscale_analysis.

## Related tools

- **MEISTER** (Framework containing the parametric UMAP autoencoder training and embedding code used to convert the 3D MSI hyperspectral data cube into feature images for MRI co-registration) — https://github.com/richardxie1119/MEISTER
- **embed_3DCoronal.ipynb / embed3D.ipynb** (Notebooks executing the parametric UMAP training on coronal and sagittal MSI data to obtain coherent feature images) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook embed_3DCoronal.ipynb  # loads coronal3D_propagated_decoded.h5, trains the parametric UMAP autoencoder (10% pixels, 20 epochs), saves parametric_umap_feature_images.npy
```

## Evaluation signals

- parametric_umap_feature_images.npy exists and contains exactly one feature image per input tissue section
- Grayscale feature images exhibit structurally informative, morphologically consistent (MRI-like) anatomical contrast across serial sections
- Feature images are suitable as moving images: affine and B-spline registration to the corresponding MRI slice converges and the transformations apply cleanly to all ion images
- Encoder input/output dimension equals the number of m/z features retained after 3-ppm binning and cross-section feature prevalence filtering
- Trained on exactly 10% of total pixels for 20 epochs; embedding of all sections lands in one shared space (no per-section drift)

## Limitations

- Prior 2D-section-based approaches can produce incoherent image registration across serial sections in large 3D MSI datasets; this skill avoids that only if ALL sections are embedded together in one training run
- Choice of which three UMAP dimensions map to RGB channels affects the anatomical contrast of the resulting grayscale feature image and therefore registration quality
- Reconstruction/annotation errors upstream (mass shifts, inappropriate ppm window, nonlinearities between MSI datasets) propagate into the embedding

## Evidence

- [methods] autoencoder architecture: [evidence span withheld — anchor status: not_found]
- [methods] training data and epochs: [evidence span withheld — anchor status: not_found]
- [results] RGB to grayscale feature images: [evidence span withheld — anchor status: not_found]
- [results] registration pipeline: "registered to its corresponding MRI slice by affine and B-spline"
- [readme] notebook entry point: [evidence span withheld — anchor status: not_found]
