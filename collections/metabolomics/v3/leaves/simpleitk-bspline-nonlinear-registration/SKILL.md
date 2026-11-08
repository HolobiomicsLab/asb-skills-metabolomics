---
name: simpleitk-bspline-nonlinear-registration
description: Use when an initial affine (or manually landmarked) H&E-to-MSI alignment
  leaves residual local mismatch between the H&E foreground and the MSI tissue mask
  that must be corrected with a non-linear transform before overlaying histology onto
  mass spectrometry imaging data.
when_to_use_negative:
- When the H&E foreground already fully overlaps the MSI tissue mask after the affine
  step alone, the B-spline refinement adds unnecessary complexity
- When no preprocessed MSI (imzML-derived) data or representative ion image is available,
  since the refinement requires a moving MSI image to register to the fixed H&E image
edam_operation: http://edamontology.org/operation_0297
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3361
tools:
- name: MSIght
  role: Python package providing refactor_bspline.perform_bspline (SimpleITK-based
    B-spline registration), refactor_affine_transform.register_he_msi for the preceding
    affine step, refactor_mz_image_extract.apply_bspline_transform_to_msi to apply
    the transform, and refactor_interpolation.interpolate_MSI to resample onto the
    histology grid
  repo: https://github.com/lingjunli-research/MSIght
- name: Jupyter
  role: Notebook environment in which the MSIght registration pipeline is run step-by-step
  repo: https://github.com/lingjunli-research/MSIght
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:796684cb375e10f30b8ef8ff51211de01a381519eeacd76b46edf60862d5f849
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_0297
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# SimpleITK B-spline non-linear registration

## Summary

Refines the MSIght H&E–MSI affine co-registration by applying a non-linear B-spline transform in SimpleITK so that the higher-resolution histology image aligns with the MSI pixel grid, producing co-registered image pairs suitable for overlay and peptide spatial localization.

## When to use

Use when an initial affine (or manually landmarked) H&E-to-MSI alignment leaves residual local mismatch between the H&E foreground and the MSI tissue mask that must be corrected with a non-linear transform before overlaying histology onto mass spectrometry imaging data.

## When NOT to use

- When the H&E foreground already fully overlaps the MSI tissue mask after the affine step alone, the B-spline refinement adds unnecessary complexity
- When no preprocessed MSI (imzML-derived) data or representative ion image is available, since the refinement requires a moving MSI image to register to the fixed H&E image

## Inputs

- H&E histology image loaded and preprocessed with MSIght.refactor_histology_preprocess (load_he_image, red_channel_extract, bin_he_image, smooth_he_image, foreground_extract/foreground_mask_make)
- MSI imzML data loaded and preprocessed with MSIght.refactor_common_functions (load_and_preprocess_imzml, create_intensity_matrix) with a representative ion image extracted via extract_mz_image_transform
- An initial affine transform (automated phase cross correlation or manual landmark-based) between the MSI and H&E images

## Outputs

- A refined non-linear B-spline transform aligning the MSI data to the H&E image
- B-spline-transformed MSI data (apply_bspline_transform_to_msi)
- Optionally, interpolated MSI data on the histology pixel grid and saved overlay images (overlay_msi_he, display_and_save_image)

## How to apply

After MSIght.refactor_affine_transform (register_he_msi) has produced a coarse affine alignment of the moving MSI image to the fixed H&E image — optionally via MSIght.refactor_manual_affine landmark alignment when automated phase cross correlation minimizing SSD is insufficient — refine the transform non-linearly with MSIght.refactor_bspline (perform_bspline). perform_bspline runs SimpleITK's B-spline registration configured with 50 histograms, a gradient tolerance of 1e-5, 100 iterations, and a coarseness of 50. Apply the resulting transform to the MSI intensity data with MSIght.refactor_mz_image_extract (apply_bspline_transform_to_msi), then optionally interpolate the transformed MSI data (MSIght.refactor_interpolation.interpolate_MSI) to match the histology pixel grid and produce overlay images via overlay_msi_he. The rationale is that the affine step only corrects global translation, rotation, and scaling, while the B-spline step absorbs local tissue deformation between the H&E stain and the MSI section, which is necessary for the eight tissue types in Figure 3.

## Related tools

- **MSIght** (Python package providing refactor_bspline.perform_bspline (SimpleITK-based B-spline registration), refactor_affine_transform.register_he_msi for the preceding affine step, refactor_mz_image_extract.apply_bspline_transform_to_msi to apply the transform, and refactor_interpolation.interpolate_MSI to resample onto the histology grid) — https://github.com/lingjunli-research/MSIght
- **Jupyter** (Notebook environment in which the MSIght registration pipeline is run step-by-step) — https://github.com/lingjunli-research/MSIght

## Examples

```
pip install msight  # then in Jupyter: from MSIght.refactor_bspline import perform_bspline; refined = perform_bspline(he_image, msi_image, num_hist=50, grad_tol=1e-5, iters=100, coarseness=50)
```

## Evaluation signals

- The refined registration should reproduce the alignment quality shown in Figure 3 and Supplementary Figures S6–S7 for all eight tissue types, with the H&E foreground overlapping the MSI tissue mask.
- Overlay images produced via overlay_msi_he must be saved (via display_and_save_image) and pass expert visual review for local tissue-boundary correspondence.
- The transform must be a valid SimpleITK B-spline deformation that, when applied with apply_bspline_transform_to_msi, maps the MSI ion image onto the H&E coordinate system without degrading the representative ion image's structural features.
- If the preceding affine step used phase cross correlation, the B-spline refinement should further reduce the SSD between the preprocessed H&E binary/smoothed image and the MSI tissue mask.

## Limitations

- The B-spline refinement is non-linear and assumes local tissue deformation is smooth; it cannot recover gross misalignment left by the affine step, so manual landmark alignment (manual_register_he_msi, show_msi_he_coords) may still be needed when automated phase cross correlation minimizing SSD is insufficient.
- Validated only on the eight tissue types presented in the paper; performance on other tissue architectures is not demonstrated.
- Registration quality is ultimately judged by visual expert review rather than a quantitative registration metric, so automated assessment alone is insufficient.

## Evidence

- [methods] affine + B-spline refinement workflow: [evidence span withheld — anchor status: not_found]
- [supplementary] B-spline parameters: [evidence span withheld — anchor status: not_found]
- [readme] README installation: [evidence span withheld — anchor status: not_found]
