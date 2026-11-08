---
name: grayscale-feature-image-conversion
description: Use when serial 2D MSI sections need a single-channel anatomically contrasting
  image (e.g., to register against T2*-weighted MRI atlas slices) from a high-dimensional
  hyperspectral data cube or UMAP embedding.
when_to_use_negative:
- Input data are already single-channel intensity images (e.g., optical microscopy
  TIFFs from the Axio Imager M2) — no embedding-to-grayscale conversion is needed
  before registration.
- The goal is pixel-level chemical interpretation of the UMAP embedding rather than
  image registration — collapsing three UMAP dimensions to grayscale discards chemical
  separation information.
- Sections lack a trained parametric UMAP embedding space shared across serial sections;
  the grayscale conversion presupposes the coherent cross-section embedding from the
  UMAP autoencoder.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: MEISTER
  role: Integrative framework whose parametric UMAP embedding produces the multi-dimensional
    feature images converted to grayscale for registration
  repo: https://github.com/richardxie1119/MEISTER
- name: embed_3D.ipynb / embed_3DCoronal.ipynb notebooks
  role: Notebooks in multiscale_analysis that perform parametric UMAP to obtain coherent
    feature images for sagittal and coronal datasets
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:eca48ac85e8448877dba2f6115426992423b064d2245cf5eeb071685be7975a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - embed_3D.ipynb / embed_3DCoronal.ipynb notebooks
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# grayscale-feature-image-conversion

## Summary

Converts a 2D MSI section's parametric UMAP embedding into a single grayscale feature image by encoding three UMAP dimensions as red/green/blue channels and collapsing them to grayscale, providing anatomical contrast suitable for multimodal image registration to MRI atlas slices.

## When to use

Use when serial 2D MSI sections need a single-channel anatomically contrasting image (e.g., to register against T2*-weighted MRI atlas slices) from a high-dimensional hyperspectral data cube or UMAP embedding.

## When NOT to use

- Input data are already single-channel intensity images (e.g., optical microscopy TIFFs from the Axio Imager M2) — no embedding-to-grayscale conversion is needed before registration.
- The goal is pixel-level chemical interpretation of the UMAP embedding rather than image registration — collapsing three UMAP dimensions to grayscale discards chemical separation information.
- Sections lack a trained parametric UMAP embedding space shared across serial sections; the grayscale conversion presupposes the coherent cross-section embedding from the UMAP autoencoder.

## Inputs

- Parametric UMAP embeddings of each 2D MSI tissue section (from reconstructed 3D coronal data, e.g., coronal3D_propagated_decoded.h5)
- Coronal or sagittal rat brain MSI hyperspectral data cubes (~2 million total pixels per 3D dataset, 37 coronal and 39 sagittal sections)

## Outputs

- Single grayscale feature image per 2D MSI section (three UMAP dimensions encoded as RGB then collapsed to grayscale) with anatomical contrast for registration
- Feature images used as moving images for affine + B-spline registration to Waxholm Space MRI slices

## How to apply

After training a parametric UMAP autoencoder (encoder minimizing UMAP cross-entropy loss, decoder minimizing reconstruction loss; fully connected layers of 256, 128, and 64 neurons on 10% of randomly sampled pixels for 20 epochs) to obtain a coherent embedding space across all serial sections, select three UMAP dimensions of each 2D tissue section and encode them into red/green/blue channels. Convert the resulting RGB feature image to a single grayscale image, which yields anatomical contrast between brain regions. This grayscale image serves as the 'moving' image for rigid affine registration (nine hand-selected initial transformation points) followed by nonrigid cubic B-spline registration (mutual information similarity, 200 maximum optimization steps) to the corresponding Waxholm Space T2*-weighted MRI slice; the learned transformations are then applied to every ion image of that section. This step is implemented in the embed_3D.ipynb / embed_3DCoronal.ipynb notebooks of the multiscale_analysis repository.

## Related tools

- **MEISTER** (Integrative framework whose parametric UMAP embedding produces the multi-dimensional feature images converted to grayscale for registration) — https://github.com/richardxie1119/MEISTER
- **embed_3D.ipynb / embed_3DCoronal.ipynb notebooks** (Notebooks in multiscale_analysis that perform parametric UMAP to obtain coherent feature images for sagittal and coronal datasets) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter nbconvert --to notebook --execute embed_3DCoronal.ipynb  # converts three UMAP dimensions per section to grayscale feature images for MRI registration
```

## Evaluation signals

- The grayscale feature image shows visible anatomical contrast between brain regions matching the corresponding T2*-weighted Waxholm Space MRI slice structure.
- Downstream affine registration with nine hand-selected initial transformation points and B-spline registration (mutual information, 200 max optimization steps) converges without gross misalignment.
- Dice index computed between atlas region labels and human-annotated masks on the registered section confirms overlap for all registered sections and regions (per Supplementary Fig. 10; no numeric threshold reported).
- Exactly three UMAP dimensions are encoded per section (RGB), and the same coherent embedding space (trained on 10% of pixels, 20 epochs) is used across all serial sections.

## Limitations

- The grayscale image is only for registration support; chemical specificity of individual UMAP dimensions is lost in the RGB-to-grayscale collapse.
- Registration quality depends on the prior coherent parametric UMAP embedding; prior 2D-section-based approaches can produce incoherent image registration across serial sections in large 3D MSI datasets.
- No numeric Dice-index threshold is reported; validation relies on visual overlap of region masks per Supplementary Fig. 10.

## Evidence

- [methods] Encode three UMAP dimensions of each 2D tissue section into red/green/blue channels, convert the resulting feature images to grayscale for registration.: [evidence span withheld — anchor status: not_found]
- [results] Parametric UMAP embeddings of each 2D tissue section were converted to a single grayscale image yielding anatomical contrast, which was then registered to its corresponding MRI slice by affine and B-spline registration: [evidence span withheld — anchor status: not_found]
- [results] Images of three selected UMAP dimensions of each two-dimensional (2D) tissue section were converted to a single grayscale image: [evidence span withheld — anchor status: not_found]
- [readme] embed3D.ipynb and embed_3DCoronal: Parametric UMAP to obtain coherent feature images for sagittal and coronal data sets.: [evidence span withheld — anchor status: external_doc]
