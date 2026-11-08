---
name: msi-to-mri-multimodal-image-registration
description: Use when many misaligned serial MSI tissue sections (e.g., H5-serialized
  3D MSI like coronal3D_propagated_decoded.h5) must be aligned to a reference MRI/atlas
  for volumetric reconstruction and per-brain-region profiling.
when_to_use_negative:
- Input is a single already-aligned 2D MSI section with no serial-section misalignment
  to correct — the multimodal registration pipeline addresses incoherent registration
  across serial sections in large 3D MSI datasets
- Reference modality is not an anatomical MRI/atlas (e.g., single-cell or tissue-only
  analysis such as SCMS cross-annotation does not require MRI registration)
- Modalities cannot yield mutually informative feature images (e.g., no parametric
  UMAP embedding can produce grayscale anatomical contrast between MSI and MRI)
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3520
tools:
- name: MEISTER
  role: Integrative framework whose parametric UMAP feature images and registration
    procedure implement the MSI-to-MRI/atlas alignment; code and notebooks are in
    the MEISTER and multiscale_analysis repositories
  repo: https://github.com/richardxie1119/MEISTER
- name: multiscale_analysis notebooks
  role: embed_3D.ipynb/embed_3DCoronal.ipynb produce the parametric UMAP feature images;
    regmri_atlas_coronal.ipynb performs brain regional analysis of registered 3D coronal
    data with the MRI atlas
  repo: https://github.com/richardxie1119/multiscale_analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-024-02171-3
    title: Multiscale biochemical mapping of the brain through deep-learning-enhanced
      high-throughput mass spectrometry
  source_content_hash: sha256:a762d9ba1d7b65ddfb7b85c8166db9301900112a65c326615403232e6c822f72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/richardxie1119/MEISTER
  tools:
  - MEISTER
  - multiscale_analysis notebooks
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msi-to-mri-multimodal-image-registration

## Summary

Registers serial 2D mass spectrometry imaging (MSI) sections of a reconstructed 3D rat brain volume to T2*-weighted MRI slices from the Waxholm Space atlas via affine plus cubic B-spline registration, enabling coherent volumetric reconstruction and quantitative regional chemical profiling.

## When to use

Use when many misaligned serial MSI tissue sections (e.g., H5-serialized 3D MSI like coronal3D_propagated_decoded.h5) must be aligned to a reference MRI/atlas for volumetric reconstruction and per-brain-region profiling.

## When NOT to use

- Input is a single already-aligned 2D MSI section with no serial-section misalignment to correct — the multimodal registration pipeline addresses incoherent registration across serial sections in large 3D MSI datasets
- Reference modality is not an anatomical MRI/atlas (e.g., single-cell or tissue-only analysis such as SCMS cross-annotation does not require MRI registration)
- Modalities cannot yield mutually informative feature images (e.g., no parametric UMAP embedding can produce grayscale anatomical contrast between MSI and MRI)

## Inputs

- Reconstructed 3D coronal MSI data (coronal3D_propagated_decoded.h5) from serial rat brain sections
- T2*-weighted anatomical MRI slices from the Waxholm Space atlas of the Sprague-Dawley rat brain
- Hand-selected initial transformation points (nine per section) for affine initialization

## Outputs

- Registered volumetric MSI reconstruction (registered_msi_volume.h5) with transformations applied to all ion images
- Per-section transformation matrices from affine and B-spline registration
- Per-region Dice-index scores (dice_scores.csv) comparing atlas labels to human-annotated masks on registered sections

## How to apply

Load the reconstructed 3D coronal MSI data (coronal3D_propagated_decoded.h5) and select corresponding T2*-weighted MRI slices from the Waxholm Space atlas of the Sprague-Dawley rat brain based on tissue-sectioning distance with manual inspection. Train a parametric UMAP autoencoder (encoder minimizing UMAP cross-entropy loss, decoder minimizing reconstruction loss; fully connected layers of 256, 128 and 64 neurons) on 10% of randomly sampled pixels for 20 epochs to obtain one coherent embedding space for all serial sections. Encode three UMAP dimensions of each section into RGB channels, convert to a single grayscale feature image, then perform rigid affine registration (moving MSI image to reference MRI slice) using nine hand-selected initial transformation points, followed by nonrigid cubic B-spline registration with mutual information as the similarity measure and 200 maximum optimization steps. Apply the obtained transformation matrices to every ion image of each section to produce the registered volumetric reconstruction (registered_msi_volume.h5), then compute Dice indices between atlas region labels and human-annotated masks per registered section (dice_scores.csv) to assess quality.

## Related tools

- **MEISTER** (Integrative framework whose parametric UMAP feature images and registration procedure implement the MSI-to-MRI/atlas alignment; code and notebooks are in the MEISTER and multiscale_analysis repositories) — https://github.com/richardxie1119/MEISTER
- **multiscale_analysis notebooks** (embed_3D.ipynb/embed_3DCoronal.ipynb produce the parametric UMAP feature images; regmri_atlas_coronal.ipynb performs brain regional analysis of registered 3D coronal data with the MRI atlas) — https://github.com/richardxie1119/multiscale_analysis

## Examples

```
jupyter notebook regmri_atlas_coronal.ipynb  # from richardxie1119/multiscale_analysis, loading coronal3D_propagated_decoded.h5 and Waxholm MRI slices
```

## Evaluation signals

- Dice-index values between ith brain-region labels from the Waxholm atlas and human-annotated masks on the jth registered section confirm registration quality (Supplementary Fig. 10); DI must be computed for all registered sections with region masks overlapping as visualized in the paper's quality check
- Registered output must be a complete volumetric reconstruction: transformations applied to every ion image of each MSI section, saved as registered_msi_volume.h5
- Parametric UMAP autoencoder must be trained on 10% of randomly sampled pixels for 20 epochs and yield a single coherent embedding space across all serial sections
- Each grayscale MSI feature image must first be affine-registered using nine hand-selected points before nonrigid cubic B-spline registration (mutual information, 200 max optimization steps)
- Corresponding MRI slices must be selected by tissue-sectioning distance with manual inspection to ensure slice-to-section correspondence

## Limitations

- No numeric Dice-index threshold is reported in the paper; quality is validated by visual overlap of region masks (Supplementary Fig. 10), so acceptance is qualitative
- Prior 2D-section-based registration approaches can produce incoherent image registration across serial sections in large 3D MSI datasets, motivating this 3D approach
- MRI slice correspondence depends on manual inspection of sectioning distance, introducing operator subjectivity

## Evidence

- [methods] Parametric UMAP embeddings converted to grayscale for registration: [evidence span withheld — anchor status: not_found]
- [results] Affine and B-spline registration description: [evidence span withheld — anchor status: not_found]
- [methods] Registration to Waxholm atlas stated in methods: [evidence span withheld — anchor status: not_found]
- [readme] Notebook for regional analysis of registered data: "regmri_atlas_coronal.ipynb | Brain regional analysis of registered 3D coronal data with MRI atlas."
- [discussion] Motivation: incoherent serial-section registration: [evidence span withheld — anchor status: not_found]
