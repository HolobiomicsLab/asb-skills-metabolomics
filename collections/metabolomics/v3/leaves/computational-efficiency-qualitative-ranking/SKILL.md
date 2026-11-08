---
name: computational-efficiency-qualitative-ranking
description: Use when a low-resolution MSI composite/cluster image must be resampled
  to the pixel dimensions of a co-registered H&E image and a defensible choice among
  interpolation kernels (nearest neighbor, bilinear, bicubic, Lanczos) is needed.
when_to_use_negative:
- The MSI composite image is already at the H&E resolution, so resampling (and hence
  interpolation choice) is unnecessary
- Sub-pixel-accurate co-registration between MSI and histology is the goal — this
  skill is a resampling/visual-comparison step, not the MSIght linear/non-linear registration
  workflow
- A quantitative (numeric) interpolation benchmark is required; the paper's evaluation
  is a qualitative visual and speed comparison only
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3361
tools:
- name: OpenCV
  role: Provides the four interpolation algorithms (nearest neighbor, bilinear, bicubic,
    Lanczos) applied via image resize calls
- name: MSIght
  role: Source of the MSI composite cluster image and reference dimensions via its
    refactored_mz_image_extract module
  repo: https://github.com/lingjunli-research/MSIght
- name: Python
  role: Implementation language for the interpolation loop, figure composition, and
    timing of each method
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:713b4dce6ac9cc416b643a06e228615438d2daf620c9c09d0eed9da4db6055bd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/lingjunli-research/MSIght
  tools:
  - MSIght
  - OpenCV
  - Python
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# computational-efficiency-qualitative-ranking

## Summary

Reproduces the MSIght paper's Figure S5 comparison of the four interpolation algorithms (bilinear, bicubic, nearest neighbor, Lanczos) used to upscale an MSI composite cluster image to H&E dimensions, and ranks them qualitatively on smoothing quality and speed.

## When to use

Use when a low-resolution MSI composite/cluster image must be resampled to the pixel dimensions of a co-registered H&E image and a defensible choice among interpolation kernels (nearest neighbor, bilinear, bicubic, Lanczos) is needed.

## When NOT to use

- The MSI composite image is already at the H&E resolution, so resampling (and hence interpolation choice) is unnecessary
- Sub-pixel-accurate co-registration between MSI and histology is the goal — this skill is a resampling/visual-comparison step, not the MSIght linear/non-linear registration workflow
- A quantitative (numeric) interpolation benchmark is required; the paper's evaluation is a qualitative visual and speed comparison only

## Inputs

- MSI composite cluster image (output of the MSIght clustering pipeline, e.g. from MSIght.refactor_segment.cluster_msi)
- H&E-stained histology image (source of target pixel dimensions)
- Python/OpenCV environment

## Outputs

- Four upscaled MSI composite images, each resampled to the exact H&E pixel dimensions
- Single side-by-side comparison figure reproducing supplementary Figure S5
- Recorded qualitative ordering of interpolation methods on smoothing quality and runtime

## How to apply

Load the MSI composite cluster image (produced by the MSIght clustering pipeline, e.g. via the refactored_mz_image_extract module) and the H&E image, reading each image's dimensions to set the target upsampling size. Apply each of the four OpenCV interpolation algorithms in turn — cv2.INTER_NEAREST, cv2.INTER_LINEAR, cv2.INTER_CUBIC, and cv2.INTER_LANCZOS4 — to resample the MSI composite to the H&E width/height. Compose the four results into a single side-by-side comparison figure mirroring Figure S5. Judge the outputs qualitatively: nearest neighbor is expected to be fastest but yield the poorest, most pixelated upscaled MSI image, while bilinear, bicubic, and Lanczos yield fairly similar smoothing quality, with Lanczos being the slowest. Save the figure and record the observed quality/speed ordering against this expectation.

## Related tools

- **OpenCV** (Provides the four interpolation algorithms (nearest neighbor, bilinear, bicubic, Lanczos) applied via image resize calls)
- **MSIght** (Source of the MSI composite cluster image and reference dimensions via its refactored_mz_image_extract module) — https://github.com/lingjunli-research/MSIght
- **Python** (Implementation language for the interpolation loop, figure composition, and timing of each method)

## Examples

```
import cv2; up_lanczos = cv2.resize(msi_composite, (he_img.shape[1], he_img.shape[0]), interpolation=cv2.INTER_LANCZOS4)
```

## Evaluation signals

- Each upscaled MSI image has exactly the same pixel dimensions as the H&E image
- The nearest-neighbor panel is visibly the most pixelated/blocky, matching the paper's reported finding
- The bilinear, bicubic, and Lanczos panels show fairly similar smoothing quality with no single obvious winner among them
- Speed ordering matches the paper: nearest neighbor fastest, Lanczos slowest
- The saved figure is a four-panel side-by-side layout equivalent to supplementary Figure S5

## Limitations

- The comparison is qualitative (visual quality and relative speed); no numeric image-similarity metric is reported in the paper
- Nearest neighbor is fastest but produces the poorest, most pixelated upscale, so it should not be chosen for final visualizations
- Lanczos is the slowest of the four methods, which matters for large H&E-sized targets
- The finding was established for upscaling MSI composite cluster images to H&E dimensions; it may not transfer to other image types or downscaling

## Evidence

- [supplementary] Nearest neighbor interpolation produced the poorest, most pixelated upscaled MSI image: [evidence span withheld — anchor status: not_found]
- [supplementary] Reported quality/speed ordering: [evidence span withheld — anchor status: not_found]
- [readme] MSIght role in workflow: "MSIght is a program produced by the Lingjun Li Lab at the University of Wisconsin-Madison for automated integration of H&E, MSI, and LC-MS/MS datasets"
- [methods] Clustering pipeline module referenced for composite image generation: "from MSIght.refactor_segment import cluster_msi"
