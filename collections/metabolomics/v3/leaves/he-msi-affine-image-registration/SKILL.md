---
name: he-msi-affine-image-registration
description: Use when raw imzML MSI data and a matching high-resolution H&E image
  must be spatially aligned (affine plus B-spline refinement) before overlaying ion
  images on tissue histology.
when_to_use_negative:
- When the goal is peptide identification or LC-MS/MS confirmation rather than spatial
  co-registration of H&E and MSI data
- When the MSI dataset already contains a registered/overlaid H&E image or the H&E
  and MSI are already spatially aligned
- When no matching H&E histology image exists for the MSI sample
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: MSIght
  role: Python package providing the full H&E/MSI co-registration pipeline (affine
    registration, B-spline refinement, overlay, and saving of registered images)
  repo: https://github.com/lingjunli-research/MSIght
- name: SimpleITK
  role: Backend for the non-linear B-spline registration refinement (perform_bspline)
    with 50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:03dc657d9691633a6ed8217a4874aa5db606d28053bb847c8e262a2256d060d3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  - SimpleITK
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# he-msi-affine-image-registration

## Summary

Co-register an H&E histology image to an MSI dataset by first applying an affine transformation (guided by phase cross correlation) and then a non-linear B-spline refinement in SimpleITK, as implemented in MSIght.

## When to use

Use when raw imzML MSI data and a matching high-resolution H&E image must be spatially aligned (affine plus B-spline refinement) before overlaying ion images on tissue histology.

## When NOT to use

- When the goal is peptide identification or LC-MS/MS confirmation rather than spatial co-registration of H&E and MSI data
- When the MSI dataset already contains a registered/overlaid H&E image or the H&E and MSI are already spatially aligned
- When no matching H&E histology image exists for the MSI sample

## Inputs

- H&E histology image (high-resolution fixed image, loaded via load_he_image)
- MSI data in imzML format (load_and_preprocess_imzml, create_intensity_matrix)
- A representative ion image (extracted with extract_mz_image_transform)
- Optional manual landmark coordinates for manual_register_he_msi

## Outputs

- Affine transformation aligning H&E to MSI
- B-spline non-linear transform applied to the MSI data
- Optionally interpolated MSI data on the histology pixel grid
- Overlay image of co-registered MSI and H&E (overlay_msi_he)
- Saved registered image pairs (display_and_save_image)

## How to apply

Preprocess the H&E image with MSIght.refactor_histology_preprocess: load_he_image, red_channel_extract, bin_he_image, smooth_he_image, and foreground_extract/foreground_mask_make to isolate tissue from background. Load and preprocess the MSI data (imzML) with load_and_preprocess_imzml, create_intensity_matrix, then extract a representative ion image via extract_mz_image_transform. Run affine registration of the H&E image to the MSI image with MSIght.refactor_affine_transform (register_he_msi); when automated phase cross correlation minimizing SSD is insufficient, use manual_register_he_msi with show_msi_he_coords for landmark-based manual alignment. Refine the registration non-linearly with MSIght.refactor_bspline (perform_bspline) using SimpleITK with 50 histograms, gradient tolerance 1e-5, 100 iterations, and coarseness 50, then apply the transform to the MSI data with apply_bspline_transform_to_msi and optionally interpolate_MSI to match the histology pixel grid. Finally overlay the co-registered MSI and H&E images with overlay_msi_he and save the registered pairs with display_and_save_image.

## Related tools

- **MSIght** (Python package providing the full H&E/MSI co-registration pipeline (affine registration, B-spline refinement, overlay, and saving of registered images)) — https://github.com/lingjunli-research/MSIght
- **SimpleITK** (Backend for the non-linear B-spline registration refinement (perform_bspline) with 50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50)

## Examples

```
pip install msight
```

## Evaluation signals

- Registered pairs for all eight tissue types must reproduce the alignment quality shown in Figure 3 and Supplementary Figures S6–S7
- H&E foreground mask must overlap the MSI tissue mask after registration
- Overlay images saved for expert visual review should show consistent tissue alignment
- B-spline refinement must converge within 100 iterations at gradient tolerance 1e-5 with coarseness 50

## Limitations

- Automated phase cross correlation minimizing SSD may be insufficient for some tissue types, requiring landmark-based manual alignment via manual_register_he_msi
- Registration quality depends on proper H&E pre-processing (red channel extraction, binning, smoothing, foreground masking) to isolate tissue from background
- Requires a matching H&E histology image at higher resolution than the MSI data; registration is one-directional (moving MSI to fixed H&E)

## Evidence

- [supplementary] MSIght registration is performed by aligning the moving MSI image to the higher-resolution fixed H&E image via affine transformation, followed by a non-linear B-spline registration refinement implemented in SimpleITK.: [evidence span withheld — anchor status: not_found]
- [supplementary] Refine the registration non-linearly with MSIght.refactor_bspline (perform_bspline) using SimpleITK with 50 histograms, gradient tolerance 1e-5, 100 iterations, and coarseness 50.: [evidence span withheld — anchor status: not_found]
- [abstract] Histological stains of tissue sections are coregistered to their respective MSI data sets to improve spatial localization and resolution of identified peptides.: "Histological stains of tissue sections are coregistered to their respective MSI data sets to improve spatial localization and resolution of identified peptides."
- [supplementary] Linear and non-linear registration workflow: "Figure S7: Linear and non-linear registration workflow"
