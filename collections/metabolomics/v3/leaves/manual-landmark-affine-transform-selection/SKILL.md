---
name: manual-landmark-affine-transform-selection
description: Use when MSIght's automated affine registration of an MSI image to an
  H&E stain fails—e.g., when phase cross correlation does not minimize SSD sufficiently
  and tissue masks do not overlap—requiring manual landmark placement.
when_to_use_negative:
- When the automated phase-cross-correlation affine transform (register_he_msi) already
  yields a satisfactory overlap between H&E foreground and MSI tissue mask—manual
  landmarks are only the fallback.
- 'For non-linear distortions within the tissue (folds, tears, local warping): manual
  affine is coarse and should be followed by the B-spline refinement (perform_bspline),
  not used alone.'
- When H&E and MSI derive from adjacent, not identical, tissue sections with grossly
  different morphology, since manual landmark pairing becomes unreliable.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: MSIght
  role: Provides manual_register_he_msi and show_msi_he_coords for landmark-based
    manual affine registration of MSI to H&E, plus register_he_msi (automated phase
    cross correlation) whose failure triggers this skill.
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language of MSIght; this skill is exercised as Python calls
    from the Jupyter notebook pipeline.
- name: Jupyter
  role: Interactive environment in which landmark coordinates are visually inspected
    and the manual registration is iterated.
- name: SimpleITK
  role: Used downstream to perform the non-linear B-spline refinement (perform_bspline)
    after the manual affine transform is applied.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:c2a6d4699d5fa758dc805be65b489df4fe4fa2ce0149b894d981ebecb6c635e4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - Jupyter
  - MSIght
  - Python
  - SimpleITK
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# manual-landmark-affine-transform-selection

## Summary

A fallback registration procedure within MSIght in which the analyst hand-selects paired MSI/H&E landmarks to derive a manual affine transform when automated phase-cross-correlation minimization of SSD fails to align the moving MSI image to the fixed H&E image.

## When to use

Use when MSIght's automated affine registration of an MSI image to an H&E stain fails—e.g., when phase cross correlation does not minimize SSD sufficiently and tissue masks do not overlap—requiring manual landmark placement.

## When NOT to use

- When the automated phase-cross-correlation affine transform (register_he_msi) already yields a satisfactory overlap between H&E foreground and MSI tissue mask—manual landmarks are only the fallback.
- For non-linear distortions within the tissue (folds, tears, local warping): manual affine is coarse and should be followed by the B-spline refinement (perform_bspline), not used alone.
- When H&E and MSI derive from adjacent, not identical, tissue sections with grossly different morphology, since manual landmark pairing becomes unreliable.

## Inputs

- Pre-processed H&E image (from load_he_image / red_channel_extract / bin_he_image / smooth_he_image / foreground_mask_make)
- Pre-processed MSI dataset (load_and_preprocess_imzml + create_intensity_matrix)
- Representative MSI ion image (extract_mz_image_transform)
- Initial automated affine transform or its failure output (register_he_msi)
- MSI/H&E coordinate pairs supplied by the analyst (via show_msi_he_coords)

## Outputs

- Manually derived affine transform aligning MSI to H&E
- Transformed MSI image matrix (manual_register_he_msi output)
- Diagnostic coordinate display (show_msi_he_coords output)
- Co-registered MSI/H&E overlay image for review (overlay_msi_he)

## How to apply

First load and pre-process the H&E image (red_channel_extract, bin_he_image, smooth_he_image, foreground_extract) and the MSI data (load_and_preprocess_imzml, create_intensity_matrix) and extract a representative ion image via extract_mz_image_transform. Run the automated affine registration (register_he_msi) and inspect the overlay: if alignment is inadequate, switch to manual_register_he_msi, using show_msi_he_coords to identify corresponding coordinates between the MSI ion image and the H&E foreground. The tool computes an affine transform from these landmark pairs and applies it to the MSI image matrix so the transformed MSI tissue mask overlaps the H&E tissue foreground; the result should be refined afterward with perform_bspline (50 histograms, gradient tolerance 1e-5, 100 iterations, coarseness 50) via SimpleITK. Validate by overlaying with overlay_msi_he and confirming that the H&E foreground overlaps the MSI tissue mask before saving the registered pair for expert review.

## Related tools

- **MSIght** (Provides manual_register_he_msi and show_msi_he_coords for landmark-based manual affine registration of MSI to H&E, plus register_he_msi (automated phase cross correlation) whose failure triggers this skill.) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language of MSIght; this skill is exercised as Python calls from the Jupyter notebook pipeline.)
- **Jupyter** (Interactive environment in which landmark coordinates are visually inspected and the manual registration is iterated.)
- **SimpleITK** (Used downstream to perform the non-linear B-spline refinement (perform_bspline) after the manual affine transform is applied.)

## Examples

```
from MSIght.refactor_manual_affine import manual_register_he_msi, show_msi_he_coords
```

## Evaluation signals

- The manually selected coordinate pairs must produce an affine transform under which the H&E tissue foreground overlaps the MSI tissue mask (the article's stated validation criterion for the eight registered tissue pairs in Figure 3 / Supplementary S6–S7).
- Overlay images generated with overlay_msi_he must show aligned tissue boundaries and can be saved with display_and_save_image for expert visual review.
- The manual transform should equal or improve the SSD-minimizing outcome of the automated register_he_msi attempt; if it is worse, landmark selection was erroneous.
- After the manual affine is applied, the follow-up B-spline refinement (perform_bspline) should converge within its 100-iteration budget at gradient tolerance 1e-5 without producing tissue-mask expansion beyond the H&E foreground.
- The chosen ion image used for landmarking (extract_mz_image_transform) must show tissue-specific structure comparable to the H&E stain; if the ion image is diffuse or background-dominated, landmarks will be uninformative.

## Limitations

- Manual landmarking is subjective and depends on the analyst; MSIght documentation frames it as a complement to automated registration, not a replacement.
- The manual affine only corrects linear misalignment; local non-linear distortions must be addressed by the subsequent SimpleITK B-spline refinement.
- Quality is judged visually via overlays (Figure 3, Supplementary S6–S7) — no quantitative registration metric is reported in the article, so correctness must be confirmed by expert review.
- Applicability depends on a representative ion image with recognizable morphological correspondence to H&E; poor ion images make landmark pairing unreliable.

## Evidence

- [supplementary] manual affine fallback with automated affine comparison: [evidence span withheld — anchor status: not_found]
- [other] automated method description: [evidence span withheld — anchor status: not_found]
- [supplementary] preprocessing and ion image extraction: [evidence span withheld — anchor status: not_found]
- [supplementary] validation via overlay and save: [evidence span withheld — anchor status: not_found]
