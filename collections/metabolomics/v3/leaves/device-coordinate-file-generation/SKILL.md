---
name: device-coordinate-file-generation
description: Use when optical target pixel coordinates and three fiducial marks of
  known instrument position must be registered and exported as a device-consumable
  coordinate/geometry file for optically guided MS.
when_to_use_negative:
- Samples smaller than ~300 μm, where microMS with microscope imaging is more effective
  than macroMS scanner/camera imaging
- You already have coordinates in the instrument/device frame — no pixel-to-instrument
  registration is needed
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: macroMS
  role: Web platform performing target recognition, target filtering, fiducial marking,
    optical correction, and device coordinate generation; exports the geometry file
    for the MS device
  repo: github.com/macroMS/macroMS_web
- name: macroMS_web
  role: Web repository hosting macroMS and its sample data (sample plate image and
    fiducial pixel positions) used for calibration and testing
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Predecessor optically guided MS software for sub-300 μm samples; not used
    for this macroscopic coordinate-generation skill
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:afee7669072798006d9a64d13c3d0b63fdf02e8e78da0312c2d7d94c415b2e32
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - macroMS_web
  - microMS
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# device-coordinate-file-generation

## Summary

Convert optically detected target pixel positions into instrument-relative X,Y coordinates and export a macroMS geometry file consumable by the MALDI-TOF MS device, using fiducial-based registration to map image pixels into the instrument coordinate frame.

## When to use

Use when optical target pixel coordinates and three fiducial marks of known instrument position must be registered and exported as a device-consumable coordinate/geometry file for optically guided MS.

## When NOT to use

- Samples smaller than ~300 μm, where microMS with microscope imaging is more effective than macroMS scanner/camera imaging
- You already have coordinates in the instrument/device frame — no pixel-to-instrument registration is needed

## Inputs

- Optical image of the sample plate (e.g., flatbed scanner or cell phone camera capture)
- Pixel coordinates of detected (and filtered) sample targets/wells
- Pixel positions of three fiducial marks with known instrument coordinates
- macroMS_web sample data for fiducial calibration

## Outputs

- MacroMS geometry file listing targets and fiducials in instrument-relative coordinates, downloadable and interpretable by the MS device
- Instrument-relative X,Y coordinate table for each sample well

## How to apply

Load the sample plate optical image plus the pixel positions of the three corner fiducial marks whose instrument relative coordinates are known (macroMS_web sample data provides an example). Fit the mapping from image pixels to instrument relative coordinates from the three fiducial correspondences, incorporating the translation, rotation, uniform scale, and perspective/distortion correction that macroMS provides for optical defects of flatbed scanners and cell phone cameras. Apply the fitted transform to every detected, filtered target pixel coordinate to yield instrument-relative X,Y for each sample well. Export the transformed targets and fiducials as the macroMS geometry file listing relative coordinates; the instrument interprets this file to acquire spectra at the selected locations. Validate by checking the file parses as a structured coordinate table and that back-transforming fiducials reproduces their expected instrument positions within tolerance.

## Related tools

- **macroMS** (Web platform performing target recognition, target filtering, fiducial marking, optical correction, and device coordinate generation; exports the geometry file for the MS device) — github.com/macroMS/macroMS_web
- **macroMS_web** (Web repository hosting macroMS and its sample data (sample plate image and fiducial pixel positions) used for calibration and testing) — github.com/macroMS/macroMS_web
- **microMS** (Predecessor optically guided MS software for sub-300 μm samples; not used for this macroscopic coordinate-generation skill)

## Evaluation signals

- Output geometry file exists and parses as a structured coordinate table listing targets and fiducials
- Back-transforming the three fiducial points reproduces their expected instrument coordinates within tolerance
- Transform applied to all detected targets yields instrument-relative X,Y for every well (up to ~10,000 targets)
- Coordinates survive the optical correction for distortion/imperfect perspective inherent to scanner/camera images
- MS device accepts the custom geometry file and acquires spectra at the selected locations

## Limitations

- Upper limit of ~10,000 targets, reported without measurement details or uncertainty
- Optical correction handles only moderate distortion and imperfect perspective from common imaging devices, not severe optics defects
- Fiducial-based transform relies on exactly three corner fiducials; mis-marked fiducials propagate error to all target coordinates

## Evidence
