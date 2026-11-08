---
name: multimodal-image-overlay-visualization
description: Use when an MSI composite/cluster image must be upscaled to match the
  pixel dimensions of a co-registered H&E stain for side-by-side or overlay visualization,
  and an interpolation method must be chosen.
when_to_use_negative:
- The MSI and H&E images are already at matching dimensions and no resampling is needed
  — direct co-registration or overlay is sufficient
- Quantitative spatial statistics (not visual comparison) are required, since interpolation
  choice introduces smoothing artifacts that distort pixel-level intensities
- The input is raw MSI spectral data rather than an already-computed composite/cluster
  image — clustering (e.g., MSIght cluster_msi) must occur first
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: MSIght
  role: Source platform that generates the MSI composite cluster image and provides
    the refactored_mz_image_extract module for determining target upsampling dimensions
    relative to the H&E image
  repo: https://github.com/lingjunli-research/MSIght
- name: OpenCV
  role: Provides the four interpolation algorithms (nearest neighbor, bilinear, bicubic,
    Lanczos) used via cv2.resize to upscale the MSI composite to H&E dimensions
- name: Python
  role: Implementation language for the interpolation comparison script and figure
    composition
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:78c1b8e721aa59c43e4ee3cf324f6a9266b56c0dcb34a2b18f008f6cde4cad68
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - OpenCV
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# multimodal-image-overlay-visualization

## Summary

Prepares MSI composite cluster images for visual overlay and comparison against co-registered H&E histology images by resampling the lower-resolution MSI image to the H&E dimensions, comparing OpenCV interpolation algorithms (bilinear, bicubic, nearest neighbor, Lanczos) as benchmarked in MSIght's supplementary Figure S5.

## When to use

Use when an MSI composite/cluster image must be upscaled to match the pixel dimensions of a co-registered H&E stain for side-by-side or overlay visualization, and an interpolation method must be chosen.

## When NOT to use

- The MSI and H&E images are already at matching dimensions and no resampling is needed — direct co-registration or overlay is sufficient
- Quantitative spatial statistics (not visual comparison) are required, since interpolation choice introduces smoothing artifacts that distort pixel-level intensities
- The input is raw MSI spectral data rather than an already-computed composite/cluster image — clustering (e.g., MSIght cluster_msi) must occur first

## Inputs

- MSI composite cluster image (2D array/image output from MSIght clustering)
- H&E histology stain image (target-resolution reference image)
- Image dimension metadata from both images (determines upscaling target)

## Outputs

- Four upscaled MSI composite images (one per interpolation method: nearest neighbor, bilinear, bicubic, Lanczos)
- Side-by-side comparison figure of the four interpolated images (analogous to supplementary Figure S5)
- Recorded quality/speed ordering of the interpolation methods

## How to apply

Load the MSI composite cluster image (e.g., produced by MSIght's refactored_mz_image_extract/cluster_msi pipeline) and the H&E image, reading each image's dimensions to determine the target upsampling size. Apply each of the four OpenCV interpolation algorithms in turn — cv2.INTER_NEAREST, cv2.INTER_LINEAR, cv2.INTER_CUBIC, and cv2.INTER_LANCZOS4 — to resample the MSI composite to the H&E dimensions, then compose the results into a single side-by-side comparison figure. The rationale, per the paper's supplementary Figure S5, is that nearest neighbor is fastest but produces the poorest, most pixelated result, while bilinear, bicubic, and Lanczos yield fairly similar smoothing quality with Lanczos being slowest; this trade-off informs which interpolation to use for final co-registered MSI/histology overlays.

## Related tools

- **MSIght** (Source platform that generates the MSI composite cluster image and provides the refactored_mz_image_extract module for determining target upsampling dimensions relative to the H&E image) — https://github.com/lingjunli-research/MSIght
- **OpenCV** (Provides the four interpolation algorithms (nearest neighbor, bilinear, bicubic, Lanczos) used via cv2.resize to upscale the MSI composite to H&E dimensions)
- **Python** (Implementation language for the interpolation comparison script and figure composition)

## Examples

```
import cv2; upscaled = cv2.resize(msi_composite, (he_image.shape[1], he_image.shape[0]), interpolation=cv2.INTER_LANCZOS4)
```

## Evaluation signals

- The upscaled MSI image dimensions exactly match the H&E image dimensions in both axes
- Visual ordering reproduces supplementary Figure S5: nearest neighbor is the most pixelated, while bilinear, bicubic, and Lanczos show fairly similar smoothing quality
- Nearest neighbor completes fastest and Lanczos slowest among the four methods (runtime comparison)
- All four resampled outputs derive from the same source composite, so cluster color assignments are preserved (only spatial resampling differs)
- The saved comparison figure contains all four methods labeled side by side for direct visual inspection

## Limitations

- Nearest neighbor upsampling is fast but produces the poorest, most pixelated result; Lanczos gives comparable quality to bicubic/bilinear at the highest computational cost
- Interpolation smooths or replicates pixel values and is intended for visualization/co-registration support, not for recovering true spatial resolution of the MSI data
- The quality comparison is visual; the paper's validation relies on matching the ordering reported in supplementary Figure S5 rather than a quantitative metric

## Evidence

- [supplementary] Nearest neighbor interpolation produced the poorest, most pixelated upscaled MSI image, while bilinear, bicubic, and Lanczos methods yielded images of fairly similar smoothing quality.: [evidence span withheld — anchor status: not_found]
