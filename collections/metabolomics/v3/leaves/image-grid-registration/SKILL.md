---
name: image-grid-registration
description: Use when target coordinates are derived from optical images acquired
  with imperfect devices (flatbed scanners, cell phone cameras) that introduce moderate
  distortion or imperfect perspective before MS targeting.
when_to_use_negative:
- Samples are smaller than 300 μm — the article states microMS is more effective at
  that scale
- The image geometry is already known-good (e.g., coordinates generated directly by
  instrument software with no optical distortion)
tools:
- name: macroMS
  role: Web platform providing the optical correction tools (perspective correction
    and distortion-map subtraction) for target coordinates
  repo: github.com/macroMS/macroMS_web
- name: OpenCV
  role: Supplies the affine/perspective warp (cv2.warpPerspective) used to rectify
    the image using the four corner points
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:86dacd80b87069f928a477a156e58e6fb46bf867da152725767bd9580e4400ee
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - OpenCV
  - macroMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# image-grid-registration

## Summary

Corrects optical distortion and imperfect perspective in macroscopic sample images so that MALDI-TOF target coordinates are accurate, by combining a four-point perspective warp with a reference-grid distortion map, as implemented in macroMS.

## When to use

Use when target coordinates are derived from optical images acquired with imperfect devices (flatbed scanners, cell phone cameras) that introduce moderate distortion or imperfect perspective before MS targeting.

## When NOT to use

- Samples are smaller than 300 μm — the article states microMS is more effective at that scale
- The image geometry is already known-good (e.g., coordinates generated directly by instrument software with no optical distortion)

## Inputs

- Distorted optical image of the sample plate (from a flatbed scanner or cell phone camera)
- Four user-specified corner points of the imaging region
- Reference grid array mapping known distorted dot positions to true positions

## Outputs

- Perspective- and distortion-corrected target coordinates
- Spatial distortion map (measured vs. expected reference-grid displacement field)
- Per-target positional error metrics (e.g., recovered_coordinates.csv)

## How to apply

Load the distorted optical image and have the user specify the four corner points; apply OpenCV's perspective warp (cv2.warpPerspective) to rectify the image to a canonical frame. Next, exploit the macroMS reference grid: compare measured versus expected reference-grid dot positions to build a spatial map of optical distortion, then subtract (with interpolation) the displacement expected at each target location from the target coordinate. This residual-correction step is needed because the perspective warp alone does not remove lens/mechanical distortion inherent to common imaging devices. Validate by running the corrected pipeline on a held-out distorted test image, extracting target coordinates, and comparing them to ground-truth corrected coordinates to record per-target positional error. Note macroMS is scoped to macroscopic (>300 μm) samples; microMS is more effective for smaller targets.

## Related tools

- **macroMS** (Web platform providing the optical correction tools (perspective correction and distortion-map subtraction) for target coordinates) — github.com/macroMS/macroMS_web
- **OpenCV** (Supplies the affine/perspective warp (cv2.warpPerspective) used to rectify the image using the four corner points)

## Examples

```
import cv2; M = cv2.getPerspectiveTransform(corners, dst); rectified = cv2.warpPerspective(img, M, (w, h))
```

## Evaluation signals

- Extracted target coordinates from a held-out distorted test image match ground-truth corrected coordinates within small per-target positional error
- Reference-grid dots after correction align with their expected true positions (residual displacement ≈ 0)
- Corrected geometry file drives MALDI-TOF spectra acquisition at the intended sample locations (e.g., yeast colonies or nL droplet arrays)
- Target count and identities are preserved through correction (no targets lost or misassigned)

## Limitations

- Correction handles only moderate distortion and imperfect perspective; severe lens defects may not be recoverable
- Requires a reference grid with known true dot positions to build the distortion map
- Depends on accurate user-specified corner points; errors there propagate into the warp

## Evidence
