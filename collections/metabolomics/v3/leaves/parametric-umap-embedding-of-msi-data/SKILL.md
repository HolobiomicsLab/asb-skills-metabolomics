---
name: parametric-umap-embedding-of-msi-data
description: Use when you have a reconstructed, m/z-binned and TIC-normalized serial-section
  MSI dataset (e.g. coronal3D_propagated_decoded.h5) and need anatomically contrasted
  2D feature images to co-register against MRI atlas slices.
when_to_use_negative:
- Input is raw transient data (e.g. 64k/1M files from MassIVE) — reconstruction and
  post-processing (m/z binning, TIC normalization) must precede embedding
- The MSI volume is already registered to the MRI/atlas — feature images are only
  the moving-image precursor for registration, not the registration itself
- Analysis targets single-cell SCMS spectra rather than tissue-section MSI pixel data
  (use the single-cell / cross-annotation workflows instead)
edam_operation: http://edamontology.org/operation_2939
tools:
- name: MEISTER
  role: Framework whose parametric UMAP autoencoder module embeds all 3D MSI pixels
    into a shared space for feature-image generation
  repo: https://github.com/richardxie1119/MEISTER
- name: multiscale_analysis notebooks (embed_3D.ipynb / embed_3DCoronal.ipynb)
  role: Notebooks implementing parametric UMAP to obtain coherent feature images for
    the sagittal and coronal datasets
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:5fd2ca897967a3740436f63f4ad2c68100b54dda92697622e7d3f94fd8e68bed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis notebooks (embed_3D.ipynb / embed_3DCoronal.ipynb)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# parametric-umap-embedding-of-msi-data

## Summary

Embeds every pixel of a processed 3D MSI volume into a shared parametric UMAP space (via an autoencoder with 256/128/64-neuron fully connected layers) and renders the first three UMAP dimensions per section as grayscale 'feature' images with MRI-like anatomical contrast, used downstream for MRI-atlas registration.

## When to use

Use when you have a reconstructed, m/z-binned and TIC-normalized serial-section MSI dataset (e.g. coronal3D_propagated_decoded.h5) and need anatomically contrasted 2D feature images to co-register against MRI atlas slices.

## When NOT to use

- Input is raw transient data (e.g. 64k/1M files from MassIVE) — reconstruction and post-processing (m/z binning, TIC normalization) must precede embedding
- The MSI volume is already registered to the MRI/atlas — feature images are only the moving-image precursor for registration, not the registration itself
- Analysis targets single-cell SCMS spectra rather than tissue-section MSI pixel data (use the single-cell / cross-annotation workflows instead)

## Inputs

- Reconstructed and processed 3D MSI data cube of serial tissue sections (e.g. coronal3D_propagated_decoded.h5), after m/z binning in 3-ppm increments and TIC normalization
- Number of m/z features (defines autoencoder input/output size)

## Outputs

- Per-section stack of RGB feature images from the first three UMAP dimensions
- Grayscale feature images per section saved as parametric_umap_feature_images.npy (one image per section, for use as moving images in registration)

## How to apply

Load the processed 3D MSI dataset (m/z-binned in 3-ppm increments and TIC-normalized, e.g. coronal3D_propagated_decoded.h5 from deposit 10.13012/B2IDB-9740536_V1). Build a parametric UMAP autoencoder whose input and output size equals the number of m/z features, with fully connected layers of 256, 128 and 64 neurons: the encoder is trained to minimize the UMAP cross-entropy loss while the decoder minimizes a reconstruction loss. Randomly sample 10% of total pixels across all serial sections as training data and train for 20 epochs, so every pixel of every section embeds into a single shared embedding space — this shared space is what keeps serial sections coherent. Apply the trained encoder to all pixels of each 2D section and take the first three UMAP dimensions as red, green and blue channels to form RGB feature images, then convert each to grayscale (yielding the MRI-like contrast needed as the moving image for subsequent affine and B-spline registration) and save the per-section stack as parametric_umap_feature_images.npy. The paper's rationale is that raw ion images lack anatomical contrast for multimodal registration, whereas the UMAP feature images exhibit tissue-structure contrast suitable for alignment to Waxholm Space T2*-weighted MRI.

## Related tools

- **MEISTER** (Framework whose parametric UMAP autoencoder module embeds all 3D MSI pixels into a shared space for feature-image generation) — https://github.com/richardxie1119/MEISTER
- **multiscale_analysis notebooks (embed_3D.ipynb / embed_3DCoronal.ipynb)** (Notebooks implementing parametric UMAP to obtain coherent feature images for the sagittal and coronal datasets) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter nbconvert --to notebook --execute embed_3DCoronal.ipynb  # from github.com/richardxie1119/multiscale_analysis, with coronal3D_propagated_decoded.h5 as input
```

## Evaluation signals

- parametric_umap_feature_images.npy exists and contains exactly one feature image per tissue section
- Grayscale feature images show structurally informative, morphologically consistent, MRI-like anatomical contrast across serial sections
- All sections' pixels embed into a single shared UMAP space (consistent spatial neighborhoods across sections, since training pools 10% of pixels from all sections)
- Feature images are usable as moving images in the subsequent affine + B-spline registration, i.e. registration converges within 200 optimization steps with mutual-information similarity
- Downstream Dice-index quality check between atlas region labels and registered-section annotations overlaps as in the paper's Supplementary Fig. 10

## Limitations

- No numeric quality threshold is reported for the downstream registration (Dice index) — accept when DI is computed for all sections and masks overlap visually
- Embedding quality depends on the shared space learned from only 10% of randomly sampled pixels over 20 epochs; rare or section-specific features may be poorly represented
- Prior 2D-section-based approaches can yield incoherent registration across serial sections in large 3D MSI datasets — this is the failure mode the shared-embedding design mitigates
- Repository documentation is still being expanded ('We are expanding our code documentations to make it friendly'), so practitioners rely on the Supplementary Protocol PDF

## Evidence

- [results] Parametric UMAP embedding produces registration feature images: [evidence span withheld — anchor status: not_found]
- [methods] Waxholm Space MRI registration target: [evidence span withheld — anchor status: not_found]
- [readme] Notebook implementing this skill: [evidence span withheld — anchor status: not_found]
- [results] Downstream 3D reconstruction pipeline: [evidence span withheld — anchor status: not_found]
