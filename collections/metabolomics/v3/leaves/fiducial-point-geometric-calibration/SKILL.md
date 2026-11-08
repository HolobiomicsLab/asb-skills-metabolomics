---
name: fiducial-point-geometric-calibration
description: Use when an optical image with three fiducial marks of known instrument
  coordinates must be registered so detected target pixels can be output as instrument-relative
  X, Y positions in an MS geometry file.
when_to_use_negative:
- Samples smaller than 300 μm — the article states 'For samples smaller than 300 μm,
  microMS is more effective', so the microMS microscope-based workflow should be used
  instead
- Images with severe (not moderate) optical distortion or heavily imperfect perspective
  beyond the correction tools macroMS provides
- Target sets exceeding ~10,000 points, the reported upper limit for target number
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: macroMS
  role: Web platform providing fiducial marking, optical correction, target recognition,
    and device coordinate generation that this skill reimplements
  repo: github.com/macroMS/macroMS_web
- name: macroMS_web
  role: Source repository supplying the sample data (plate image and fiducial pixel
    positions) and geometry file format used to reconstruct and validate the transform
  repo: github.com/macroMS/macroMS_web
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:66c5fa16bc9a814a566f98c5b04dcb172c87745178e4f2c867d56322eb249f2b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - macroMS_web
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fiducial-point-geometric-calibration

## Summary

Reconstructs macroMS's fiducial-based coordinate transformation, which maps optical-image pixel positions of detected targets into instrument-relative physical coordinates using three corner fiducial marks, and exports the result as a device-consumable geometry file.

## When to use

Use when an optical image with three fiducial marks of known instrument coordinates must be registered so detected target pixels can be output as instrument-relative X, Y positions in an MS geometry file.

## When NOT to use

- Samples smaller than 300 μm — the article states 'For samples smaller than 300 μm, microMS is more effective', so the microMS microscope-based workflow should be used instead
- Images with severe (not moderate) optical distortion or heavily imperfect perspective beyond the correction tools macroMS provides
- Target sets exceeding ~10,000 points, the reported upper limit for target number

## Inputs

- Optical image of the sample plate (flatbed scanner or cell phone camera capture)
- Pixel positions of the three corner fiducial marks with known instrument coordinates (from macroMS_web sample data)
- Pixel coordinates of detected/filtered targets on the sample image

## Outputs

- Instrument-relative X, Y coordinates for each detected sample-well target
- macroMS custom geometry file listing targets and fiducials in relative coordinates, consumable by the MS device

## How to apply

Load the sample plate optical image together with the pixel positions of the three fiducial marks whose instrument coordinates are known, taken from the macroMS_web repository sample data. Solve the registration from these three fiducial pixel-to-instrument correspondences to obtain a similarity transform comprising translation, rotation, uniform scale, and perspective correction — the perspective term exists because macroMS explicitly adds optical correction tools for 'moderate distortion and imperfect perspective' of flatbed-scanner or cell-phone-camera images. Apply the fitted transform to every detected target pixel coordinate to yield instrument-relative X, Y positions for each sample well. Export the transformed coordinates in the macroMS custom geometry file format that the MS device ingests to acquire spectra at the selected locations. Validate by checking the output file parses as a structured coordinate table and that back-transforming the three fiducial points reproduces their expected instrument positions within tolerance.

## Related tools

- **macroMS** (Web platform providing fiducial marking, optical correction, target recognition, and device coordinate generation that this skill reimplements) — github.com/macroMS/macroMS_web
- **macroMS_web** (Source repository supplying the sample data (plate image and fiducial pixel positions) and geometry file format used to reconstruct and validate the transform) — github.com/macroMS/macroMS_web

## Evaluation signals

- Back-transforming the three fiducial points through the fitted transform reproduces their expected instrument positions within a stated tolerance
- Exported geometry file exists and parses as a structured coordinate table (targets plus fiducials in relative coordinates)
- All detected target pixel coordinates map to finite instrument-relative X, Y values with no dropped targets
- Residual registration error is consistent with macroMS's reported reliable targeting of samples of 300 μm using the full workflow

## Limitations

- Optical correction only handles moderate distortion and imperfect perspective inherent to common imaging devices; severe lens distortion may not be resolvable
- The transform is anchored on exactly three fiducial correspondences, so fiducial localization error propagates directly into all target coordinates
- Reported upper limit of ~10,000 targets is presented without measurement details or uncertainty; sample-size claims (300 μm targeting) lack replicate/statistical detail

## Evidence
