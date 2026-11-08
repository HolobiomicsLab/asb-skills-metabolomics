---
name: feature-image-generation-for-image-registration
description: Use when you have m/z-binned, TIC-normalized serial-section MSI data
  (e.g. coronal3D_propagated_decoded.h5) and need per-section images with morphological
  contrast suitable as moving images for affine/B-spline registration to MRI.
when_to_use_negative:
- Input is already registered ion images or a final feature table — this skill operates
  on the raw m/z-binned pixel x feature MSI cube before registration.
- Only a single 2D section with no serial-section alignment requirement — a single-section
  UMAP visualization or supervised classification may suffice and full 3D embedding
  training may be unnecessary.
- The target registration reference is not an anatomical image (e.g. histology or
  a different modality whose contrast is not captured by the first three UMAP dimensions).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MEISTER
  role: framework providing the parametric UMAP embedding step that converts MSI hyperspectral
    data into feature images for registration
  repo: https://github.com/richardxie1119/MEISTER
- name: embed_3DCoronal.ipynb
  role: notebook implementing parametric UMAP to obtain coherent feature images for
    the coronal dataset
  repo: https://github.com/richardxie1119/multiscale_analysis/blob/main/embed_3DCoronal.ipynb
- name: embed3D.ipynb
  role: notebook implementing parametric UMAP to obtain coherent feature images for
    the sagittal dataset
  repo: https://github.com/richardxie1119/multiscale_analysis/blob/main/embed3D.ipynb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:c05d2964e2fd2b4d1a60bf62089a8592e5022ba89a4870458efce802930eb84e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - embed3D.ipynb
  - embed_3DCoronal.ipynb
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# feature-image-generation-for-image-registration

## Summary

Generates grayscale anatomical-contrast 'feature' images from a reconstructed 3D MSI data cube by training a parametric UMAP autoencoder and encoding each 2D tissue section, so the MSI sections can be co-registered to MRI atlas slices.

## When to use

Use when you have m/z-binned, TIC-normalized serial-section MSI data (e.g. coronal3D_propagated_decoded.h5) and need per-section images with morphological contrast suitable as moving images for affine/B-spline registration to MRI.

## When NOT to use

- Input is already registered ion images or a final feature table — this skill operates on the raw m/z-binned pixel x feature MSI cube before registration.
- Only a single 2D section with no serial-section alignment requirement — a single-section UMAP visualization or supervised classification may suffice and full 3D embedding training may be unnecessary.
- The target registration reference is not an anatomical image (e.g. histology or a different modality whose contrast is not captured by the first three UMAP dimensions).

## Inputs

- reconstructed and processed 3D MSI data cube (m/z-binned, TIC-normalized serial-section peak data, e.g. coronal3D_propagated_decoded.h5)
- number of m/z features per pixel (defines autoencoder input/output size)

## Outputs

- parametric_umap_feature_images.npy: per-section grayscale feature images with MRI-like anatomical contrast
- trained parametric UMAP encoder/decoder weights
- 3D UMAP embedding for every pixel across all serial sections

## How to apply

Load the reconstructed, processed 3D MSI dataset (m/z-binned in 3-ppm increments and TIC-normalized serial-section peak data, e.g. coronal3D_propagated_decoded.h5 from deposit 10.13012/B2IDB-9740536_V1). Build a parametric UMAP autoencoder whose input/output size equals the number of m/z features, with fully connected layers of 256, 128, and 64 neurons, training the encoder with the UMAP cross-entropy loss and the decoder with a reconstruction loss. Randomly sample 10% of the total pixels across all serial sections as training data and train for 20 epochs, so all pixels embed into a single shared embedding space — this is what makes feature images coherent across serial sections. Apply the trained encoder to embed all pixels of every tissue section, take the first three UMAP embedding dimensions per section as red/green/blue channels to form RGB feature images, then convert each section's RGB image to grayscale and save the per-section stack as parametric_umap_feature_images.npy. The grayscale images serve as the moving images that are subsequently registered to corresponding MRI slices by affine and B-spline transformations.

## Related tools

- **MEISTER** (framework providing the parametric UMAP embedding step that converts MSI hyperspectral data into feature images for registration) — https://github.com/richardxie1119/MEISTER
- **embed_3DCoronal.ipynb** (notebook implementing parametric UMAP to obtain coherent feature images for the coronal dataset) — https://github.com/richardxie1119/multiscale_analysis/blob/main/embed_3DCoronal.ipynb
- **embed3D.ipynb** (notebook implementing parametric UMAP to obtain coherent feature images for the sagittal dataset) — https://github.com/richardxie1119/multiscale_analysis/blob/main/embed3D.ipynb

## Examples

```
python -c "import nbformat; nbformat.reads(open('embed_3DCoronal.ipynb').read(), as_version=4)" && jupyter nbconvert --to notebook --execute embed_3DCoronal.ipynb --output embed_3DCoronal_executed.ipynb
```

## Evaluation signals

- parametric_umap_feature_images.npy exists and contains one feature image per tissue section
- grayscale feature images exhibit structurally informative, morphologically consistent (MRI-like) anatomical contrast across serial sections
- images are suitable as moving images for the subsequent affine and B-spline registration step
- all pixels across serial sections embed into one shared embedding space (no per-section drift in the embedding)

## Limitations

- Prior 2D-section-based approaches can produce incoherent image registration across serial sections in large 3D MSI datasets; embedding all sections into a single space is needed to avoid this
- FDR-controlled metabolite annotation methods cannot be applied downstream of image-guided SCMS data due to lack of spatial information, so feature-image-derived annotations must be treated cautiously
- UoSS fitting modeling errors can arise from falsely cross-annotated features caused by mass shifts, an inappropriate ppm window, or nonlinearities between MSI and SCMS data

## Evidence

- [results] The parametric UMAP step embeds the MSI hyperspectral data cube into 'feature' images for co-registration with MRI anatomical images: [evidence span withheld — anchor status: not_found]
- [results] Parametric UMAP autoencoder architecture and training regime: [evidence span withheld — anchor status: not_found]
- [results] Sampling fraction for training data: [evidence span withheld — anchor status: not_found]
- [readme] README notebook implementing the parametric UMAP embedding: "Parametric UMAP to obtain coherent feature images for sagittal and coronal data sets."
- [discussion] Motivation for coherent 3D embedding across serial sections: [evidence span withheld — anchor status: not_found]
