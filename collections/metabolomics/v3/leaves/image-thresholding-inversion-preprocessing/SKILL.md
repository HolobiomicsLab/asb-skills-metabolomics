---
name: image-thresholding-inversion-preprocessing
description: Use when an optical image of scattered macroscopic samples (e.g., colonies,
  droplets) must be converted to a binary foreground mask before feature finding,
  as the first step of generating an MS target list.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for these,
  so the macroMS optical workflow should not be used
- Target coordinates are already known (e.g., from a liquid-handler droplet layout)
  so image-based feature finding is unnecessary
- Image is already binarized or targets have already been detected and filtered
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: macroMS
  role: Web platform whose sample-finding module performs the thresholding-plus-inversion
    preprocessing of optical images
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Source code reused by macroMS for feature finding; implements the scikit-image
    routines used here
- name: scikit-image
  role: Python library supplying the thresholding operators and image inversion used
    to isolate sample features as foreground
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:0df732d2083fde2c8dc2886eb50e3b5a84ad1381f543ef8674285918b3d232bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  - scikit-image
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# image-thresholding-inversion-preprocessing

## Summary

Preprocessing step in the macroMS target-list workflow that converts a raw optical image of scattered macroscopic samples (flatbed scanner or cell phone camera) into a binary mask via scikit-image thresholding followed by image inversion, so sample features become foreground regions ready for connected-component detection.

## When to use

Use when an optical image of scattered macroscopic samples (e.g., colonies, droplets) must be converted to a binary foreground mask before feature finding, as the first step of generating an MS target list.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for these, so the macroMS optical workflow should not be used
- Target coordinates are already known (e.g., from a liquid-handler droplet layout) so image-based feature finding is unnecessary
- Image is already binarized or targets have already been detected and filtered

## Inputs

- Optical image of macroscopic samples (flatbed scanner or cell phone camera capture)
- Optionally, an optically corrected version of the image (distortion/perspective-corrected)

## Outputs

- Inverted binary foreground mask with sample features as foreground
- Feature regions passed to connected-component detection for morphological measurement

## How to apply

Load the sample optical image (from a flatbed scanner or cell phone camera) and apply scikit-image thresholding to segment it, then invert the result so sample features are isolated as foreground rather than background. Rationale: macroMS reuses the microMS source code, which implements the scikit-image Python library, so this thresholding-plus-inversion step reproduces the microMS feature-finding behavior for objects larger than 300 μm. The output binary mask feeds directly into connected-region detection, where per-region descriptors (area/size, circularity, nearest-neighbor distances) are computed for downstream target filtering. Images from common imaging devices carry moderate distortion and imperfect perspective, so optical correction should be applied before or alongside this step to preserve accurate spatial locations of the selected targets.

## Related tools

- **macroMS** (Web platform whose sample-finding module performs the thresholding-plus-inversion preprocessing of optical images) — github.com/macroMS/macroMS_web
- **microMS** (Source code reused by macroMS for feature finding; implements the scikit-image routines used here)
- **scikit-image** (Python library supplying the thresholding operators and image inversion used to isolate sample features as foreground)

## Examples

```
from skimage.filters import threshold_otsu; mask = img > threshold_otsu(img); foreground = ~mask
```

## Evaluation signals

- Inspect the binarized mask: sample features (not background) must appear as foreground after inversion
- Number of connected components in the mask should approximately match the expected number of physical samples on the image
- After filtering, the detected target list should match the expected target count/locations on the sample image
- Downstream target filtering should be able to discard debris and merged/broken regions, indicating the mask is not over-segmented

## Limitations

- Images from common devices (flatbed scanners, cell phone cameras) contain moderate distortion and imperfect perspective that must be corrected for accurate target locations
- Thresholding is sensitive to uneven illumination/background in uncontrolled camera or scanner images
- The platform is intended for macroscopic objects; performance below the 300 μm scale is not established

## Evidence
