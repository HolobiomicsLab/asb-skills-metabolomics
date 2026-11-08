---
name: distortion-map-subtraction
description: Use when optical images of macroscopic samples were captured with flatbed
  scanners or cell phone cameras and target coordinates show moderate distortion or
  imperfect perspective before MALDI-MS coordinate generation.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for these
- Images with severe (not moderate) distortion beyond what common imaging-device correction
  tools resolve
- When the image already comes from the instrument-native geometry file and no optical
  re-imaging was performed
edam_topics:
- http://edamontology.org/topic_3383
- http://edamontology.org/topic_0092
tools:
- name: macroMS
  role: Web platform providing the optical correction tools that compute the distortion
    map and apply coordinate correction
  repo: github.com/macroMS/macroMS_web
- name: OpenCV
  role: Affine/perspective warp of the distorted image to a canonical frame using
    the four corner points before distortion-map subtraction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:6a6733f5d67af3df09964b8a1a3a130f9a83b8acd5bb7cd3692ebbd0088f86cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - OpenCV
  - macroMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# distortion-map-subtraction

## Summary

macroMS procedure for correcting optical distortion of target coordinates introduced by imperfect lenses in cameras or mechanical errors in flatbed scanners. It builds a spatial distortion map from reference-grid dots and subtracts the local displacement from each target coordinate.

## When to use

Use when optical images of macroscopic samples were captured with flatbed scanners or cell phone cameras and target coordinates show moderate distortion or imperfect perspective before MALDI-MS coordinate generation.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for these
- Images with severe (not moderate) distortion beyond what common imaging-device correction tools resolve
- When the image already comes from the instrument-native geometry file and no optical re-imaging was performed

## Inputs

- Distorted optical image of the sample plate (from a flatbed scanner or cell phone camera)
- Four user-specified corner points of the image
- Reference grid array mapping known distorted dot positions to true positions
- Target coordinates extracted from the rectified image

## Outputs

- Corrected (undistorted) target coordinates for MS device geometry files
- Distortion map of displacement vectors between measured and expected reference-grid dot positions
- recovered_coordinates.csv with recovered coordinates and per-target positional error metrics

## How to apply

Load the distorted optical image with four user-specified corner points, then rectify it to a canonical frame using OpenCV's perspective warp (cv2.warpPerspective). Next, compare the measured positions of reference-grid dots against their expected true positions to calculate a spatial map of optical distortion. Interpolate the displacement field at each target location and subtract the local distortion from the target coordinate to yield corrected positions. Run the corrected-image pipeline on a held-out distorted test image, extract target coordinates, and compare them with ground-truth corrected coordinates to record per-target positional error before writing recovered_coordinates.csv.

## Related tools

- **macroMS** (Web platform providing the optical correction tools that compute the distortion map and apply coordinate correction) — github.com/macroMS/macroMS_web
- **OpenCV** (Affine/perspective warp of the distorted image to a canonical frame using the four corner points before distortion-map subtraction)

## Examples

```
M = cv2.getPerspectiveTransform(corners); corrected = cv2.warpPerspective(img, M, (w, h))
```

## Evaluation signals

- Per-target positional error between extracted and ground-truth corrected coordinates falls near zero after correction
- Distortion map interpolates smoothly: displacement at reference dots matches measured-vs-expected offsets exactly
- recovered_coordinates.csv contains both recovered coordinates and per-target error metrics as columns
- Corrected target positions remain accurate on a held-out distorted test image, not just the calibration image
- Number of corrected targets does not exceed the reported ~10,000 target upper limit

## Limitations

- Correction is intended for moderate distortion and imperfect perspective inherent to common imaging devices; severe optical defects are out of scope
- The comparison to microMS is qualitative with no quantitative benchmark given
- The ~10,000 target upper limit is reported without measurement details or uncertainty

## Evidence
