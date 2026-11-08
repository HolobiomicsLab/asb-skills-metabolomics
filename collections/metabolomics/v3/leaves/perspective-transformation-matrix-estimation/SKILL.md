---
name: perspective-transformation-matrix-estimation
description: Use when optical images of macroscopic (>300 μm) targets are distorted
  by camera lenses or scanner mechanics and accurate spatial coordinates are needed
  for optically guided MALDI-TOF MS targeting.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS with microscope imaging
  is more effective at that scale
- Images with severe distortion beyond the 'moderate distortion' the correction tools
  were designed to resolve
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: macroMS
  role: Web platform providing the optical correction (perspective and distortion-map
    subtraction) applied to target coordinates
  repo: github.com/macroMS/macroMS_web
- name: OpenCV
  role: Applies the affine/perspective warp (e.g., cv2.warpPerspective) using the
    four corner points to rectify the image
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:1dfd481ea9438dcd34ef4810876f3ede49a5a116649920def0ddd115ca1b6b08
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - OpenCV
  - macroMS
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# perspective-transformation-matrix-estimation

## Summary

Estimates a perspective/affine transformation to rectify optical images of macroscopic samples (imaged with flatbed scanners or cell phone cameras) so that target coordinates fed to MALDI-TOF MS reflect true sample positions despite lens distortion and imperfect perspective.

## When to use

Use when optical images of macroscopic (>300 μm) targets are distorted by camera lenses or scanner mechanics and accurate spatial coordinates are needed for optically guided MALDI-TOF MS targeting.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS with microscope imaging is more effective at that scale
- Images with severe distortion beyond the 'moderate distortion' the correction tools were designed to resolve

## Inputs

- Optical image of macroscopic sample (flatbed scanner or cell phone camera)
- Four user-specified corner points of the region of interest
- Reference-grid array mapping known distorted dot positions to true positions

## Outputs

- Rectified (perspective-corrected) image
- Corrected target coordinates / device geometry file for MALDI-TOF MS
- Recovered coordinates and per-target positional error metrics (recovered_coordinates.csv)

## How to apply

Load the distorted optical image and four user-specified corner points, then compute and apply a perspective transform (e.g., OpenCV cv2.warpPerspective) to rectify the image to a canonical frame. To correct residual lens distortion beyond perspective, macroMS builds a spatial distortion map by comparing measured versus expected reference-grid dot positions, then subtracts the displacement expected at each target location from its coordinate. Output the corrected target coordinates as a custom geometry file for the MS device. Validate by re-extracting target positions from a held-out distorted image and recording per-target positional error against ground-truth corrected coordinates (e.g., recovered_coordinates.csv).

## Related tools

- **macroMS** (Web platform providing the optical correction (perspective and distortion-map subtraction) applied to target coordinates) — github.com/macroMS/macroMS_web
- **OpenCV** (Applies the affine/perspective warp (e.g., cv2.warpPerspective) using the four corner points to rectify the image)

## Examples

```
import cv2; M = cv2.getPerspectiveTransform(src_corners, dst_corners); rectified = cv2.warpPerspective(img, M, (w, h))
```

## Evaluation signals

- Corrected image shows the four corner points mapped to the canonical frame (square/rectangle with parallel edges)
- Per-target positional error computed against ground-truth corrected coordinates falls within the accuracy needed to reliably target 300 μm samples
- Corrected coordinates produce a valid custom geometry file accepted by the MS device for spectra acquisition at selected locations
- Distortion-map subtraction yields near-zero residual displacement at reference-grid dot positions

## Limitations

- Correction tools address only 'moderate distortion and imperfect perspective' inherent to common imaging devices, not severe optical defects
- No version information for macroMS is stated
- The ~10,000 target upper limit is presented without measurement details or uncertainty

## Evidence
