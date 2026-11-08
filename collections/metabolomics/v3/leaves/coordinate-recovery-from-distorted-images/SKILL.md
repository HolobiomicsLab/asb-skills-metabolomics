---
name: coordinate-recovery-from-distorted-images
description: Use when target coordinates for optically guided MS are extracted from
  images acquired with flatbed scanners or cell phone cameras that introduce moderate
  lens distortion or imperfect perspective, and accurate stage coordinates are required
  before generating device geometry files.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for those
  sizes
- Images already acquired in a distortion-free canonical frame where no perspective
  warp or grid correction is needed
- Severely distorted images beyond the 'moderate distortion' level the macroMS correction
  tools were designed for
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3382
tools:
- name: OpenCV
  role: Provides the affine/perspective warp (cv2.warpPerspective or similar) used
    to rectify the image from four corner points before distortion-map subtraction
- name: macroMS
  role: Web platform implementing the optical correction tools (distortion-map subtraction
    from reference-grid dot positions) that yield accurate sample locations
  repo: github.com/macroMS/macroMS_web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:a91a19b58cb777bd51bc4368c700825782376ba504a1fa22e358149425bd6dca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - OpenCV
  - macroMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# coordinate-recovery-from-distorted-images

## Summary

Corrects optical distortion and imperfect perspective in optical images of macroscopic (>300 μm) samples so that extracted target coordinates map accurately to MALDI-TOF MS stage positions. It combines OpenCV perspective warping from four corner points with a distortion map built from a reference grid of dots.

## When to use

Use when target coordinates for optically guided MS are extracted from images acquired with flatbed scanners or cell phone cameras that introduce moderate lens distortion or imperfect perspective, and accurate stage coordinates are required before generating device geometry files.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for those sizes
- Images already acquired in a distortion-free canonical frame where no perspective warp or grid correction is needed
- Severely distorted images beyond the 'moderate distortion' level the macroMS correction tools were designed for

## Inputs

- Distorted optical image of a macroscopic sample (from flatbed scanner or cell phone camera)
- Four user-specified corner points defining the perspective warp
- Reference grid image with dots at known positions (to compute the measured-vs-expected displacement field)
- Target coordinates from the macroMS sample-finding step

## Outputs

- Perspective-rectified image in a canonical frame
- Corrected target coordinates for MS device geometry generation
- recovered_coordinates.csv with recovered coordinates and per-target positional error metrics

## How to apply

Load the distorted optical image plus four user-specified corner points and apply OpenCV's perspective warp (cv2.warpPerspective or similar) to rectify the image to a canonical frame. Next, apply macroMS's distortion-map subtraction step: read the reference grid array mapping known distorted positions to true positions, interpolate the displacement field, and subtract the displacement expected at each target location from its coordinate. This two-stage correction (perspective then residual distortion) is what macroMS added to resolve 'optical defects inherent to common imaging devices'. Validate by running the corrected pipeline on a held-out distorted test image, extracting target coordinates, comparing against ground-truth corrected coordinates, and recording per-target positional error in recovered_coordinates.csv.

## Related tools

- **OpenCV** (Provides the affine/perspective warp (cv2.warpPerspective or similar) used to rectify the image from four corner points before distortion-map subtraction)
- **macroMS** (Web platform implementing the optical correction tools (distortion-map subtraction from reference-grid dot positions) that yield accurate sample locations) — github.com/macroMS/macroMS_web

## Examples

```
import cv2; M = cv2.getPerspectiveTransform(src_corners, dst_corners); rectified = cv2.warpPerspective(distorted_img, M, (w, h))
```

## Evaluation signals

- Per-target positional error recorded in recovered_coordinates.csv should fall below the targeted sample size (samples of 300 μm were targeted reliably with the full workflow)
- Reference-grid dots, after correction, should map from measured distorted positions to their expected true positions
- Corrected coordinates should be consistent when re-run on a held-out distorted test image versus the calibration image
- The target set should stay within the platform's ~10,000-target upper limit
- The device geometry file generated from corrected coordinates should place the MALDI laser on sample spots, not between them

## Limitations

- Correction is designed for moderate distortion; severe lens or mechanical errors may not be fully resolved
- Requires a reference grid of dots at known positions to build the displacement field
- Target count is limited to ~10,000, stated without measurement details or uncertainty

## Evidence

- [intro] new optical correction tools were added to resolve optical defects inherent to common imaging devices, including correction of moderate distortion and imperfect perspective: "new optical correction tools were added to resolve optical defects inherent to common imaging devices, including correction of moderate distortion and imperfect perspective"
