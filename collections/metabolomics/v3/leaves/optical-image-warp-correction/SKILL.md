---
name: optical-image-warp-correction
description: Use when optical images of macroscopic (>300 μm) targets come from consumer
  devices (flatbed scanner, cell phone camera) with moderate distortion or imperfect
  perspective, and accurate MALDI-TOF MS targeting coordinates are needed.
when_to_use_negative:
- Samples smaller than 300 μm — the article states microMS is more effective for these;
  use microMS instead
- Images with severe (beyond 'moderate') optical distortion that the reference-grid-based
  correction cannot resolve
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: macroMS
  role: Web platform providing the optical correction tools for moderate distortion
    and imperfect perspectives
  repo: github.com/macroMS/macroMS_web
- name: OpenCV
  role: Affine/perspective warp of the image to a canonical frame using the four corner
    points
- name: microMS
  role: Alternative optically guided MS suite preferred for samples smaller than 300
    μm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:f269348a45bbb1000e4546c2a6db3bd9e600fbea91d47e46051a2abcf6576c29
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - OpenCV
  - macroMS
  - microMS
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# optical-image-warp-correction

## Summary

Corrects optical distortion and imperfect perspective in macroscopic sample images (from flatbed scanners or cell phone cameras) so that MALDI-TOF MS target coordinates map accurately onto the physical sample stage.

## When to use

Use when optical images of macroscopic (>300 μm) targets come from consumer devices (flatbed scanner, cell phone camera) with moderate distortion or imperfect perspective, and accurate MALDI-TOF MS targeting coordinates are needed.

## When NOT to use

- Samples smaller than 300 μm — the article states microMS is more effective for these; use microMS instead
- Images with severe (beyond 'moderate') optical distortion that the reference-grid-based correction cannot resolve

## Inputs

- Optical image of macroscopic sample (from flatbed scanner or cell phone camera)
- Four user-specified corner points of the image/sample region
- Reference-grid array mapping known distorted dot positions to true positions

## Outputs

- Perspective- and distortion-corrected target coordinates (custom geometry file for the MS device)
- Spatial distortion map derived from reference-grid comparison

## How to apply

Load the distorted optical image together with four user-specified corner points, then apply an affine/perspective warp (e.g. cv2.warpPerspective) to rectify the image into a canonical frame, removing imperfect perspective. Next, correct residual lens distortion: macroMS calculates a spatial map of optical distortion by comparing measured versus expected reference-grid dot positions, and subtracts the distortion expected at each target location from the target coordinate to correct it. The corrected coordinate geometry file is then fed to the MS device so spectra are acquired at the true sample locations.

## Related tools

- **macroMS** (Web platform providing the optical correction tools for moderate distortion and imperfect perspectives) — github.com/macroMS/macroMS_web
- **OpenCV** (Affine/perspective warp of the image to a canonical frame using the four corner points)
- **microMS** (Alternative optically guided MS suite preferred for samples smaller than 300 μm)

## Examples

```
import cv2; M = cv2.getPerspectiveTransform(corners, canonical); rectified = cv2.warpPerspective(img, M, (w, h))
```

## Evaluation signals

- Reference-grid dots align with expected positions after correction (measured vs. expected dot positions show minimal displacement)
- Corrected target coordinates place the MALDI-TOF laser on the intended sample, e.g. spectra acquired from 300 μm targets are hit reliably
- Perspective warp using the four corner points yields a rectangle in the canonical frame
- No target coordinates remain outside the device stage geometry after subtraction of the distortion field

## Limitations

- Correction targets moderate distortion and imperfect perspective only; severe lens defects may not be recoverable
- Requires a reference grid and four user-specified corner points; accuracy depends on quality of these inputs
- For samples smaller than 300 μm, microMS is more effective

## Evidence

- [full_text] macroMS calculates a spatial map of optical distortion by comparing measured versus expected reference-grid dot positions, and subtracts the distortion expected at each target location from the target coordinate to correct it.: [evidence span withheld — anchor status: not_found]
