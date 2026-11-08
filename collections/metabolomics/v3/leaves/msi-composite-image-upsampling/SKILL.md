---
name: msi-composite-image-upsampling
description: Use when an MSI cluster/composite image must be resampled to the pixel
  dimensions of a co-registered H&E image, and the quality/speed tradeoff between
  interpolation methods needs to be assessed or reported.
when_to_use_negative:
- The goal is linear/non-linear co-registration of the MSI and H&E images (Figure
  S7 workflow), not simple dimension-matched upscaling of an already aligned composite
- Quantitative interpolation-error metrics against a ground-truth high-resolution
  MSI image are required — the paper's comparison is qualitative visual assessment
  only
tools:
- name: OpenCV
  role: Provides the four interpolation algorithms (INTER_NEAREST, INTER_LINEAR, INTER_CUBIC,
    INTER_LANCZOS4) used via cv2.resize to resample the MSI composite to H&E dimensions
- name: MSIght
  role: Supplies the MSI composite cluster image (via the refactored_mz_image_extract
    module and cluster_msi pipeline) whose dimensions define the upsampling source
  repo: github.com/lingjunli-research/MSIght
- name: Python
  role: Scripting language in which the OpenCV resampling and side-by-side figure
    composition are implemented
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.4c01140
    title: 'MSIght: A Modular Platform for Improved Confidence in Global, Untargeted
      Mass Spectrometry Imaging Annotation'
  source_content_hash: sha256:79427a7b974374d46ebf67f3acb4a2a37fd22542e4f28d8d1ad796a073d632ab
schema_version: 0.2.0
metadata:
  license_tier: unknown
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
yes_rate: 0.7333333333333333
n_claims: 15
---

# msi-composite-image-upsampling

## Summary

Upscales the MSI composite cluster image to the H&E image dimensions using the four interpolation algorithms supported by MSIght's comparison workflow (nearest neighbor, bilinear, bicubic, Lanczos), producing a side-by-side quality comparison as in the paper's supplementary Figure S5.

## When to use

Use when an MSI cluster/composite image must be resampled to the pixel dimensions of a co-registered H&E image, and the quality/speed tradeoff between interpolation methods needs to be assessed or reported.

## When NOT to use

- The goal is linear/non-linear co-registration of the MSI and H&E images (Figure S7 workflow), not simple dimension-matched upscaling of an already aligned composite
- Quantitative interpolation-error metrics against a ground-truth high-resolution MSI image are required — the paper's comparison is qualitative visual assessment only

## Inputs

- MSI composite cluster image (output of the MSIght cluster_msi segmentation pipeline)
- H&E histology image (pre-processed as in the H&E image pre-processing workflow, Figure S6)
- Target upscaling dimensions read from the H&E image via the MSIght refactored_mz_image_extract module

## Outputs

- Four upscaled MSI composite images, one per interpolation method (nearest neighbor, bilinear, bicubic, Lanczos)
- A single side-by-side comparison figure reproducing supplementary Figure S5
- A recorded quality/speed ordering of the four methods

## How to apply

Load the MSI composite cluster image and the H&E image, and read their dimensions to derive the target upsampling size from the MSIght refactored_mz_image_extract module (the MSI composite is produced by the cluster_msi clustering pipeline). Apply each of the four OpenCV interpolation algorithms in turn — cv2.INTER_NEAREST, cv2.INTER_LINEAR, cv2.INTER_CUBIC (bicubic), and cv2.INTER_LANCZOS4 — resampling the MSI composite to exactly the H&E width and height. Compose the four interpolated results into a single side-by-side comparison figure, then visually rank smoothing quality and speed. Rationale: nearest neighbor is fastest but produces the poorest, most pixelated upscale, while bilinear, bicubic, and Lanczos yield fairly similar smoothing quality, with Lanczos being the slowest.

## Related tools

- **OpenCV** (Provides the four interpolation algorithms (INTER_NEAREST, INTER_LINEAR, INTER_CUBIC, INTER_LANCZOS4) used via cv2.resize to resample the MSI composite to H&E dimensions)
- **MSIght** (Supplies the MSI composite cluster image (via the refactored_mz_image_extract module and cluster_msi pipeline) whose dimensions define the upsampling source) — github.com/lingjunli-research/MSIght
- **Python** (Scripting language in which the OpenCV resampling and side-by-side figure composition are implemented)

## Examples

```
import cv2; ups = [cv2.resize(msi, (he.shape[1], he.shape[0]), interpolation=m) for m in (cv2.INTER_NEAREST, cv2.INTER_LINEAR, cv2.INTER_CUBIC, cv2.INTER_LANCZOS4)]
```

## Evaluation signals

- All four upscaled images have exactly the H&E image dimensions (width and height match the H&E source)
- The comparison figure contains four panels, one per interpolation algorithm, composed side by side
- Visual ordering matches the paper: nearest neighbor is the most pixelated/poorest, while bilinear, bicubic, and Lanczos show fairly similar smoothing quality
- Speed ordering is consistent with the paper's report: nearest neighbor fastest, Lanczos slowest
- The saved figure reproduces the ordering described in supplementary Figure S5

## Limitations

- Nearest neighbor produces the poorest, most pixelated result and should not be chosen when image quality matters
- Lanczos is the slowest of the four methods, a speed/quality tradeoff to weigh for large images
- The paper's assessment is qualitative/visual; no quantitative similarity metric between interpolation outputs is provided

## Evidence

- [supplementary] task card, Figure S5 interpolation comparison: [evidence span withheld — anchor status: not_found]
- [readme] MSIght installation: [evidence span withheld — anchor status: not_found]
