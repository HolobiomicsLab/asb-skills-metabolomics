---
name: msi-hyperspectral-datacube-dimensionality-reduction
description: Use when a reconstructed serial-section 3D MSI dataset (e.g. an m/z-binned,
  TIC-normalized .h5/.imzML volume) must be converted into grayscale feature images
  that show anatomical contrast for image registration against MRI/atlas slices.
when_to_use_negative:
- The MSI dataset is a single 2D section only and will not be registered to an MRI/atlas
  volume — plain UMAP visualization or supervised classification suffices.
- The data are single-cell MS spectra (peak_data_sc_regional_decode.imzML) rather
  than spatial tissue imaging pixels — no anatomical feature images can be formed.
- Input is already registered spatially or is a non-hyperspectral modality (e.g. microscopy
  or MRI images); dimensionality reduction of an MSI data cube would add no value.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3869
tools:
- name: MEISTER
  role: Framework providing the parametric UMAP embedding workflow (embed_3D.ipynb
    / embed_3DCoronal.ipynb) that converts the MSI data cube into coherent feature
    images
  repo: https://github.com/richardxie1119/MEISTER
- name: multiscale_analysis
  role: Notebook repository containing embed3D.ipynb and embed_3DCoronal.ipynb that
    run the parametric UMAP feature-image generation
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:a218aff068a47eb684b5341ebca390794887e55addbac446218ac95b913d2f95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3869
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# msi-hyperspectral-datacube-dimensionality-reduction

## Summary

Parametric UMAP embedding of a reconstructed 3D MALDI FT-ICR mass spectrometry imaging (MSI) hyperspectral data cube into per-section 'feature' images with MRI-like anatomical contrast, enabling co-registration of serial MSI sections to a rat brain MRI atlas.

## When to use

Use when a reconstructed serial-section 3D MSI dataset (e.g. an m/z-binned, TIC-normalized .h5/.imzML volume) must be converted into grayscale feature images that show anatomical contrast for image registration against MRI/atlas slices.

## When NOT to use

- The MSI dataset is a single 2D section only and will not be registered to an MRI/atlas volume — plain UMAP visualization or supervised classification suffices.
- The data are single-cell MS spectra (peak_data_sc_regional_decode.imzML) rather than spatial tissue imaging pixels — no anatomical feature images can be formed.
- Input is already registered spatially or is a non-hyperspectral modality (e.g. microscopy or MRI images); dimensionality reduction of an MSI data cube would add no value.

## Inputs

- Reconstructed and processed 3D MSI hyperspectral data cube (e.g. coronal3D_propagated_decoded.h5), with pixels m/z-binned in 3-ppm increments and TIC-normalized
- Serial-section MSI pixel data (approximately 2 million pixels across 37 coronal or 39 sagittal rat brain sections)

## Outputs

- parametric_umap_feature_images.npy — stack of one grayscale feature image per tissue section
- Trained parametric UMAP encoder embedding all 3D pixels into a shared embedding space

## How to apply

Load the reconstructed, processed 3D MSI data (e.g. coronal3D_propagated_decoded.h5 from deposit 10.13012/B2IDB-9740536_V1), whose pixels have been m/z-binned in 3-ppm increments and TIC-normalized. Train a parametric UMAP autoencoder whose input/output size equals the number of m/z features, with fully connected layers of 256, 128 and 64 neurons, optimizing the encoder with the UMAP cross-entropy loss and the decoder with a reconstruction loss; sample 10% of total pixels across all serial sections as training data and train for 20 epochs so every pixel embeds into one shared space. Apply the trained encoder to all pixels of every section and take the first three UMAP embedding dimensions per section as red, green and blue channels to form RGB feature images, then convert each to a single grayscale image. The rationale is that three selected UMAP dimensions collapse the lipid hyperspectral contrast into a single image with morphologically informative, MRI-like structure suitable as the moving image for subsequent affine and B-spline registration.

## Related tools

- **MEISTER** (Framework providing the parametric UMAP embedding workflow (embed_3D.ipynb / embed_3DCoronal.ipynb) that converts the MSI data cube into coherent feature images) — https://github.com/richardxie1119/MEISTER
- **multiscale_analysis** (Notebook repository containing embed3D.ipynb and embed_3DCoronal.ipynb that run the parametric UMAP feature-image generation) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook embed_3DCoronal.ipynb  # run parametric UMAP on coronal3D_propagated_decoded.h5; save parametric_umap_feature_images.npy
```

## Evaluation signals

- parametric_umap_feature_images.npy exists and contains one feature image per serial section
- Grayscale feature images show structurally informative, morphologically consistent contrast across serial sections (MRI-like appearance)
- The RGB feature images are suitable as moving images for the subsequent affine and B-spline registration step
- All pixels from all sections embed into a single shared UMAP embedding space (trained on a random 10% pixel sample for 20 epochs)

## Limitations

- Poorly chosen UMAP dimensions or nonlinearities between modalities can yield feature images with misleading contrast, propagating into registration errors
- The RGB-to-grayscale conversion of three UMAP dimensions is a heuristic choice; there is no guarantee a particular triple of dimensions maximizes anatomical contrast for every tissue.
- Parametric UMAP training on only a 10% pixel sample may under-represent rare anatomical structures.

## Evidence

- [results] Parametric UMAP converts MSI hyperspectral cube into feature images for MRI co-registration: [evidence span withheld — anchor status: not_found]
- [results] Registration then applied to all ion images for final volumetric reconstruction: [evidence span withheld — anchor status: not_found]
- [readme] README identifies parametric UMAP notebook as the feature-image step: [evidence span withheld — anchor status: external_doc]
