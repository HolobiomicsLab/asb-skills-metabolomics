---
name: msi-ion-image-extraction-and-overlay-visualization
description: Use when you have a preprocessed imzML dataset and need to extract an
  ion image at a specific m/z, or to overlay that ion image on a co-registered H&E
  stain for spatial localization review.
when_to_use_negative:
- Before the affine and B-spline registration of MSI to H&E has been applied — overlaying
  unregistered MSI on histology produces misleading spatial correspondence.
- When the MSI data has not been preprocessed (loaded via load_and_preprocess_imzml
  with create_intensity_matrix) — the extraction functions assume an intensity matrix
  input.
- 'For co-registration itself: this skill only extracts and overlays images; the actual
  alignment is done by register_he_msi and perform_bspline.'
edam_operation: http://edamontology.org/operation_3643
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0610
tools:
- name: MSIght
  role: Python package whose refactor_mz_image_extract module (extract_mz_image_transform,
    apply_bspline_transform_to_msi, overlay_msi_he) and refactor_affine_transform
    module (display_and_save_image) perform ion-image extraction and H&E overlay
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language for the MSIght pipeline, chosen for accessibility
- name: Jupyter
  role: Notebook interface that walks through the extraction and overlay pipeline
    step-by-step
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:f7e794870798ac1206161a1d2e86ffca5a55f5fb25b02e29854a790b1acf7f6b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3643
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# msi-ion-image-extraction-and-overlay-visualization

## Summary

Extract representative ion images from preprocessed MALDI-MSI data (imzML) and visualize them as single-panel images or as co-registered overlays on H&E histology using MSIght's mz-image extraction and overlay functions.

## When to use

Use when you have a preprocessed imzML dataset and need to extract an ion image at a specific m/z, or to overlay that ion image on a co-registered H&E stain for spatial localization review.

## When NOT to use

- Before the affine and B-spline registration of MSI to H&E has been applied — overlaying unregistered MSI on histology produces misleading spatial correspondence.
- When the MSI data has not been preprocessed (loaded via load_and_preprocess_imzml with create_intensity_matrix) — the extraction functions assume an intensity matrix input.
- For co-registration itself: this skill only extracts and overlays images; the actual alignment is done by register_he_msi and perform_bspline.

## Inputs

- Preprocessed imzML mass spectrometry imaging dataset (via load_and_preprocess_imzml and create_intensity_matrix)
- m/z value or range for the ion of interest
- Preprocessed H&E histology image (via load_he_image, red_channel_extract, bin_he_image, smooth_he_image, foreground_extract/foreground_mask_make) — required only for overlay
- Registered/transformed MSI image from apply_bspline_transform_to_msi — required for accurate overlay

## Outputs

- Single-panel ion image for the selected m/z
- Overlay image of the co-registered MSI ion image on the H&E tissue foreground
- Registered image pair saved via display_and_save_image

## How to apply

Load and preprocess the imzML file with MSIght.refactor_common_functions (load_and_preprocess_imzml, create_intensity_matrix), then extract the ion image at the m/z of interest via MSIght.refactor_mz_image_extract (extract_mz_image_transform) so the image reflects the applied registration transform. To place the ion image in histological context, overlay it on the preprocessed H&E foreground using MSIght.refactor_mz_image_extract (overlay_msi_he). The overlay step presumes the affine + B-spline registration (50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50) has already been applied to the MSI data via apply_bspline_transform_to_msi; without that step the overlay will misrepresent the tissue correspondence.

## Related tools

- **MSIght** (Python package whose refactor_mz_image_extract module (extract_mz_image_transform, apply_bspline_transform_to_msi, overlay_msi_he) and refactor_affine_transform module (display_and_save_image) perform ion-image extraction and H&E overlay) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language for the MSIght pipeline, chosen for accessibility)
- **Jupyter** (Notebook interface that walks through the extraction and overlay pipeline step-by-step)

## Examples

```
from MSIght.refactor_mz_image_extract import extract_mz_image_transform, apply_bspline_transform_to_msi, overlay_msi_he; img = extract_mz_image_transform(intensity_matrix, mz=834.55); overlay_msi_he(img, he_foreground)
```

## Evaluation signals

- The extracted ion image shows a spatially coherent, non-uniform signal pattern consistent with a real ion, not a blank or saturated image.
- Overlaying the extracted ion image on the H&E foreground (overlay_msi_he) yields visually overlapping tissue boundaries — H&E foreground overlaps the MSI tissue mask.
- The registered image pairs reproduce the alignment quality shown in Figure 3 and Supplementary Figures S6–S7 of the paper.
- Overlay output is saved via display_and_save_image and can be opened for expert visual review.
- The m/z of the extracted image corresponds to the requested value within the mass accuracy of the imzML data.

## Limitations

- Automated phase cross correlation affine registration may be insufficient; MSIght provides manual landmark registration (manual_register_he_msi, show_msi_he_coords) as a fallback, implying automated alignment can fail on difficult tissue pairs.
- Overlay quality is bounded by the B-spline refinement; if the non-linear registration diverges (100 iterations, coarseness 50), the overlay will look plausible but be spatially wrong.
- Optional interpolation (interpolate_MSI) to match the histology pixel grid is a separate step; without it the overlay is drawn on the native MSI pixel grid and may be misaligned at histology resolution.

## Evidence
