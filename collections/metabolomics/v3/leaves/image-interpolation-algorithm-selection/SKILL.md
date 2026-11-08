---
name: image-interpolation-algorithm-selection
description: Use when a low-resolution MSI composite or cluster image must be resampled
  to the pixel dimensions of a co-registered H&E (or other histology) image and you
  need to select or justify an interpolation method.
when_to_use_negative:
- The MSI and histology images are already at matching dimensions and no resampling
  is needed (use the registration workflow instead)
- The task is geometric co-registration (linear/non-linear registration, Figure S7)
  rather than simple pixel upscaling of the composite image
- Only one interpolation method is prescribed by the downstream pipeline and no comparison/selection
  is required
tools:
- name: MSIght
  role: Provides the MSI composite cluster image (via cluster_msi) and image dimensions
    (via refactored_mz_image_extract) that are upscaled and compared
  repo: https://github.com/lingjunli-research/MSIght
- name: OpenCV
  role: Supplies the four interpolation algorithms (nearest neighbor, bilinear, bicubic,
    Lanczos) used to resample the MSI composite image to H&E dimensions
- name: Python
  role: Scripting language used to orchestrate the interpolation, figure composition,
    and comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:88b29510ae4c33820b14b686e90500fac9178a9fdbbeef307e445515e11e6246
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - OpenCV
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# image-interpolation-algorithm-selection

## Summary

Comparative benchmarking of the four interpolation algorithms supported by MSIght (bilinear, bicubic, nearest neighbor, and Lanczos) for upscaling an MSI composite cluster image to match H&E image dimensions, as in the paper's supplementary Figure S5.

## When to use

Use when a low-resolution MSI composite or cluster image must be resampled to the pixel dimensions of a co-registered H&E (or other histology) image and you need to select or justify an interpolation method.

## When NOT to use

- The MSI and histology images are already at matching dimensions and no resampling is needed (use the registration workflow instead)
- The task is geometric co-registration (linear/non-linear registration, Figure S7) rather than simple pixel upscaling of the composite image
- Only one interpolation method is prescribed by the downstream pipeline and no comparison/selection is required

## Inputs

- MSI composite cluster image (output of MSIght's cluster_msi segmentation/refactor_segment module)
- Co-registered H&E histology image defining the target upscale dimensions
- Image dimension metadata from MSIght's refactored_mz_image_extract module

## Outputs

- Four upscaled MSI composite images (one per interpolation algorithm) at H&E image dimensions
- Side-by-side comparison figure reproducing supplementary Figure S5
- Recorded quality/speed ordering of the four interpolation methods

## How to apply

Load the MSI composite cluster image and the H&E image (e.g., via MSIght's refactored_mz_image_extract module) and read their dimensions to determine the target upscaling size. Apply each of the four OpenCV interpolation algorithms — cv2.INTER_NEAREST, cv2.INTER_LINEAR, cv2.INTER_CUBIC, and cv2.INTER_LANCZOS4 — in turn to resample the MSI composite to the H&E dimensions, then compose the four results into a single side-by-side comparison figure. The expected ordering from the paper is that nearest neighbor is fastest but produces the poorest, most pixelated upscaled image, while bilinear, bicubic, and Lanczos yield images of fairly similar smoothing quality, with Lanczos slowest. Record the observed ordering and save the figure as validation against supplementary Figure S5.

## Related tools

- **MSIght** (Provides the MSI composite cluster image (via cluster_msi) and image dimensions (via refactored_mz_image_extract) that are upscaled and compared) — https://github.com/lingjunli-research/MSIght
- **OpenCV** (Supplies the four interpolation algorithms (nearest neighbor, bilinear, bicubic, Lanczos) used to resample the MSI composite image to H&E dimensions)
- **Python** (Scripting language used to orchestrate the interpolation, figure composition, and comparison)

## Examples

```
import cv2; up_lanczos = cv2.resize(msi_composite, (he_img.shape[1], he_img.shape[0]), interpolation=cv2.INTER_LANCZOS4)
```

## Evaluation signals

- The nearest neighbor result visibly shows pixelation/blockiness relative to the other three upscaled images
- Bilinear, bicubic, and Lanczos outputs are of fairly similar smoothing quality (per the paper's finding)
- All four resampled images exactly match the H&E image dimensions
- The saved comparison figure reproduces the ordering described in supplementary Figure S5 (nearest neighbor fastest but most pixelated; Lanczos slowest)

## Limitations

- The comparison is qualitative/visual; the paper reports quality and speed characteristics rather than quantitative similarity metrics
- Lanczos is the slowest of the four methods, which may matter for large images or high-throughput runs
- Nearest neighbor, though fastest, is explicitly identified as the poorest choice for producing usable upscaled MSI images

## Evidence

- [supplementary] Nearest neighbor interpolation produced the poorest, most pixelated upscaled MSI image, while bilinear, bicubic, and Lanczos methods yielded images of fairly similar smoothing quality.: [evidence span withheld — anchor status: not_found]
- [supplementary] Apply each of the four OpenCV interpolation algorithms in turn to resample the MSI composite image to the H&E dimensions.: [evidence span withheld — anchor status: not_found]
