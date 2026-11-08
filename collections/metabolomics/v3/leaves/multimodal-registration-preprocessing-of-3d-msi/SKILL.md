---
name: multimodal-registration-preprocessing-of-3d-msi
description: Use when you have a reconstructed multi-section 3D MSI dataset whose
  serial sections are misaligned and you need contrast-rich 'feature' images to register
  them to an MRI/atlas (e.g. rat brain coronal/sagittal sections).
when_to_use_negative:
- When the MSI sections are already registered or you only need ion-image visualization
  without multimodal co-registration — this skill exists to produce registration-ready
  feature images, not final ion maps.
- When only a single 2D section is available and there is no volumetric reconstruction
  or atlas/MRI target, since the embedding is trained on pixels pooled across all
  serial sections to form a single shared embedding space.
- When the input is raw low-resolution transient data (e.g. 64k-point FT-ICR transients)
  — reconstruction/processing (deep-learning decode, m/z binning, TIC normalization)
  must be completed first, since this skill expects processed peak data.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MEISTER
  role: Provides the parametric UMAP autoencoder pipeline that embeds the reconstructed
    MSI data cube into feature images for co-registration with MRI anatomical images;
    multiscale_analysis notebooks embed3D.ipynb / embed_3DCoronal.ipynb implement
    it.
  repo: https://github.com/richardxie1119/MEISTER
- name: multiscale_analysis notebooks (embed_3D.ipynb / embed_3DCoronal.ipynb)
  role: Run parametric UMAP to obtain coherent feature images for sagittal and coronal
    data sets — the concrete implementation of this skill.
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:2bede797f1221a676fc24d95d8c152ff3c9c66a954af468f0c4c2ce653d62ea8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis notebooks (embed_3D.ipynb / embed_3DCoronal.ipynb)
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multimodal-registration-preprocessing-of-3d-msi

## Summary

MEISTER preprocessing step that converts the reconstructed 3D MSI data cube (m/z-binned 3 ppm, TIC-normalized serial-section peak data, e.g. coronal3D_propagated_decoded.h5) into parametric-UMAP grayscale feature images with MRI-like anatomical contrast so the 2D serial sections can be registered to a rat brain MRI atlas.

## When to use

Use when you have a reconstructed multi-section 3D MSI dataset whose serial sections are misaligned and you need contrast-rich 'feature' images to register them to an MRI/atlas (e.g. rat brain coronal/sagittal sections).

## When NOT to use

- When the MSI sections are already registered or you only need ion-image visualization without multimodal co-registration — this skill exists to produce registration-ready feature images, not final ion maps.
- When only a single 2D section is available and there is no volumetric reconstruction or atlas/MRI target, since the embedding is trained on pixels pooled across all serial sections to form a single shared embedding space.
- When the input is raw low-resolution transient data (e.g. 64k-point FT-ICR transients) — reconstruction/processing (deep-learning decode, m/z binning, TIC normalization) must be completed first, since this skill expects processed peak data.

## Inputs

- Reconstructed and processed 3D MSI dataset of serial sections (e.g. coronal3D_propagated_decoded.h5 from deposit 10.13012/B2IDB-9740536_V1), after m/z binning (3 ppm) and TIC normalization
- Optionally the pretrained encoder weights from saved_model.zip for the parametric UMAP autoencoder

## Outputs

- Per-section grayscale parametric UMAP feature images saved as a stack (e.g. parametric_umap_feature_images.npy), one image per 2D section, with MRI-like anatomical contrast to serve as moving images for affine and B-spline registration to MRI/atlas slices

## How to apply

Load the reconstructed, processed 3D MSI peak data (m/z-binned in 3-ppm increments and TIC-normalized, e.g. coronal3D_propagated_decoded.h5). Train a parametric UMAP autoencoder whose input and output size equals the number of m/z features with fully connected layers of 256, 128 and 64 neurons, training the encoder with the UMAP cross-entropy loss and the decoder with a reconstruction loss; randomly sample 10% of all pixels across all serial sections as training data and train for 20 epochs so every 3D pixel embeds into one shared embedding space. Apply the trained encoder to embed all pixels of every section, take the first three UMAP embedding dimensions per section as red/green/blue channels to form RGB feature images, convert each to grayscale, and save the stack (e.g. parametric_umap_feature_images.npy). The rationale is that grayscale contrast from selected UMAP dimensions yields anatomically informative images that serve as the moving images for the subsequent affine and B-spline registration to the corresponding MRI slice (Waxholm Space atlas T2*-weighted anatomy). Validation: confirm the output file exists, contains one feature image per section, and the images show morphologically consistent, structurally informative contrast suitable as registration inputs.

## Related tools

- **MEISTER** (Provides the parametric UMAP autoencoder pipeline that embeds the reconstructed MSI data cube into feature images for co-registration with MRI anatomical images; multiscale_analysis notebooks embed3D.ipynb / embed_3DCoronal.ipynb implement it.) — https://github.com/richardxie1119/MEISTER
- **multiscale_analysis notebooks (embed_3D.ipynb / embed_3DCoronal.ipynb)** (Run parametric UMAP to obtain coherent feature images for sagittal and coronal data sets — the concrete implementation of this skill.) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter nbconvert --to notebook --execute embed_3DCoronal.ipynb # loads coronal3D_propagated_decoded.h5, trains parametric UMAP (256/128/64 layers, 10% sample, 20 epochs), outputs parametric_umap_feature_images.npy
```

## Evaluation signals

- Output stack (e.g. parametric_umap_feature_images.npy) exists and contains exactly one grayscale feature image per input 2D tissue section (37 coronal / 39 sagittal in the paper's 3D datasets).
- Feature images exhibit structurally informative, morphologically consistent contrast across serial sections — visually comparable to MRI anatomical contrast (e.g. recognizable ventricles, cortex/white-matter boundaries).
- Embedding is trained on a randomly sampled 10% subset of total pixels across all sections for 20 epochs — check that network architecture is 256→128→64 fully connected layers and input/output size equals the number of m/z features.
- All 3D pixels are embedded into a single shared embedding space (no per-section separate embeddings) so that downstream affine + B-spline registration to the corresponding MRI slice behaves coherently across sections.
- Images are usable as the moving images for the subsequent affine and B-spline registration step to the Waxholm Space atlas T2*-weighted MRI — downstream regmri_atlas_coronal.ipynb runs without contrast/misalignment failures.

## Limitations

- Feature contrast depends on the selected UMAP dimensions; poor or non-informative grayscale contrast can compromise downstream registration quality.
- Because training samples only 10% of pixels, rare pixel types may be underrepresented in the shared embedding space.
- The feature-image conversion is designed for MSI hyperspectral data cubes aligned to a corresponding MRI/atlas target; without an MRI/anatomy reference (e.g. Waxholm Space rat atlas) the registration step cannot proceed.
- Prior 2D-section-based registration approaches can produce incoherent registration across serial sections in large 3D MSI datasets — this skill addresses that by pooling pixels across sections, but it presumes the MSI data have been reconstructed and processed (m/z-binned 3 ppm, TIC-normalized) first.

## Evidence

- [results] how_to_apply: [evidence span withheld — anchor status: not_found]
- [discussion] when_to_use: [evidence span withheld — anchor status: not_found]
