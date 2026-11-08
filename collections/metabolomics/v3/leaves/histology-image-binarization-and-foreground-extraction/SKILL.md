---
name: histology-image-binarization-and-foreground-extraction
description: 'Use when preparing an H&E histology image (e.g. .tif/.imzML workflow
  inputs) for co-registration to MALDI-MSI data: you must isolate tissue from background
  before affine or B-spline registration.'
when_to_use_negative:
- The input is not an H&E-stained histology image (e.g. already a pre-computed tissue
  mask, a mass spectrum, or an mzML file).
- You only need an unregistered ion image or intensity matrix and will never overlay
  or register against histology — the H&E mask is unnecessary overhead.
- The histology image has no tissue/background contrast to binarize (e.g. it is already
  a single-class binary mask).
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_3382
- http://edamontology.org/topic_3444
tools:
- name: MSIght
  role: Provides the histology preprocessing functions (red_channel_extract, bin_he_image,
    smooth_he_image, foreground_extract, foreground_mask_make) that binarize the H&E
    image and extract the tissue foreground mask prior to registration
  repo: https://github.com/lingjunli-research/MSIght
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:7d6a208885348cff76a1a0c4541f9866b2bfdeb1f1524e4cd8e1e328e9c0c203
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_3382
  - http://edamontology.org/topic_3444
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Histology Image Binarization and Foreground Extraction

## Summary

MSIght's preprocessing stage that converts an H&E-stained histology image into a smoothed binary tissue mask (foreground extraction) so that it can be registered to an MSI ion image and used to constrain non-linear image alignment.

## When to use

Use when preparing an H&E histology image (e.g. .tif/.imzML workflow inputs) for co-registration to MALDI-MSI data: you must isolate tissue from background before affine or B-spline registration.

## When NOT to use

- The input is not an H&E-stained histology image (e.g. already a pre-computed tissue mask, a mass spectrum, or an mzML file).
- You only need an unregistered ion image or intensity matrix and will never overlay or register against histology — the H&E mask is unnecessary overhead.
- The histology image has no tissue/background contrast to binarize (e.g. it is already a single-class binary mask).

## Inputs

- H&E-stained histology image file loaded via MSIght.refactor_histology_preprocess.load_he_image
- Optional paired MSI data loaded with MSIght.refactor_common_functions.load_and_preprocess_imzml for downstream registration

## Outputs

- Smoothed binary foreground/tissue mask (via foreground_extract and foreground_mask_make)
- Red-channel-extracted and binarized H&E image used as the fixed image for registration

## How to apply

Load the H&E image with MSIght.refactor_histology_preprocess.load_he_image, then convert it to a single-channel representation via red_channel_extract, binarize it with bin_he_image, smooth the binary mask with smooth_he_image, and finally isolate the tissue from background using foreground_extract/foreground_mask_make. The rationale is that automated registration of the moving MSI image to the fixed, higher-resolution H&E image relies on the H&E foreground overlapping the MSI tissue mask, so a clean binarized mask is a prerequisite for both the affine (phase cross correlation, SSD-minimizing) step and the subsequent SimpleITK B-spline refinement (50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50). The smoothed binary mask is consumed downstream by register_he_msi, manual_register_he_msi (landmark-based fallback), perform_bspline, and overlay_msi_he.

## Related tools

- **MSIght** (Provides the histology preprocessing functions (red_channel_extract, bin_he_image, smooth_he_image, foreground_extract, foreground_mask_make) that binarize the H&E image and extract the tissue foreground mask prior to registration) — https://github.com/lingjunli-research/MSIght

## Examples

```
from MSIght.refactor_histology_preprocess import load_he_image; from MSIght.refactor_histology_preprocess import red_channel_extract, bin_he_image, smooth_he_image, foreground_extract, foreground_mask_make
```

## Evaluation signals

- The extracted foreground mask overlaps the MSI tissue mask when overlaid with overlay_msi_he — H&E foreground and MSI tissue signal occupy the same tissue regions.
- The binarized H&E image (after red_channel_extract and bin_he_image) cleanly separates tissue from background with no large spurious regions left after smooth_he_image.
- Downstream registration: applying the affine transform (register_he_msi) followed by B-spline refinement (perform_bspline with 50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50) yields a transformed MSI image whose ion signal aligns with the H&E foreground.
- Visual inspection of the saved registered pairs (display_and_save_image) matches the alignment quality shown in Figure 3 and Supplementary Figures S6–S7 of the source paper.

## Limitations

- Automated phase cross correlation minimizing SSD may be insufficient for some tissue pairs, requiring the manual landmark-based fallback (manual_register_he_msi, show_msi_he_coords) instead of relying solely on the binary foreground mask.
- The smoothed binary mask is a simplified representation of tissue; fine sub-cellular structures in the H&E stain are discarded and cannot be recovered during registration.
- B-spline refinement is bounded to the parameters used in the paper (50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50); other parameter choices were not validated in this workflow.

## Evidence

- [supplementary] Preprocessing of H&E for registration: "Figure S6: H&E image pre-processing workflow"
- [supplementary] Registration requires a clean H&E foreground: "Figure S7: Linear and non-linear registration workflow"
- [other] Source card workflow for foreground extraction: [evidence span withheld — anchor status: not_found]
- [other] Registration rationale from the card: [evidence span withheld — anchor status: not_found]
- [abstract] Purpose of co-registration from abstract: "Histological stains of tissue sections are coregistered to their respective MSI data sets to improve spatial localization and resolution of identified peptides."
- [readme] Tool availability and installation from README: "MSIght is a program produced by the Lingjun Li Lab at the University of Wisconsin-Madison for automated integration of H&E, MSI, and LC-MS/MS datasets"
