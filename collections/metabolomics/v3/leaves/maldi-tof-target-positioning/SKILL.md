---
name: maldi-tof-target-positioning
description: Use when you have an optical image (flatbed scanner or cell phone camera)
  of a macroscopic sample plate with detected target positions and need instrument-relative
  X, Y coordinates for MALDI-TOF spectra acquisition at selected locations.
when_to_use_negative:
- Samples smaller than 300 μm — microMS with microscope imaging is more effective
  for such targets
- Images with severe (not moderate) optical distortion beyond macroMS's correction
  capabilities
- Target sets exceeding the ~10,000-target upper limit reported for macroMS
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3399
tools:
- name: macroMS
  role: Web platform performing fiducial marking, optical correction, and device coordinate
    generation that converts target pixels to instrument-relative coordinates
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Predecessor optically guided MS workflow whose approach macroMS adapts; alternative
    for sub-300 μm samples
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:6142d941ddf6c23c66a07990b38fd8791ea834a7911d42005a237db0416d28fd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - macroMS
  - microMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# maldi-tof-target-positioning

## Summary

Converts pixel coordinates of optically detected targets into the physical instrument coordinate system of a MALDI-TOF mass spectrometer by registering three fiducial marks and exporting a macroMS geometry file consumable by the MS device.

## When to use

Use when you have an optical image (flatbed scanner or cell phone camera) of a macroscopic sample plate with detected target positions and need instrument-relative X, Y coordinates for MALDI-TOF spectra acquisition at selected locations.

## When NOT to use

- Samples smaller than 300 μm — microMS with microscope imaging is more effective for such targets
- Images with severe (not moderate) optical distortion beyond macroMS's correction capabilities
- Target sets exceeding the ~10,000-target upper limit reported for macroMS

## Inputs

- Optical image of the sample plate (from a flatbed scanner or cell phone camera)
- Pixel positions of three fiducial marks with known instrument coordinates
- Detected/filtered target pixel coordinates from macroMS sample finding

## Outputs

- Instrument-relative X, Y coordinates for every target/sample well
- macroMS geometry file (structured coordinate table listing targets and fiducials in relative coordinates) for the MS device

## How to apply

Load the sample optical image plus the pixel positions of the three fiducial marks with known instrument coordinates from the macroMS web repository sample data. Solve the registration from the three fiducial correspondences, fitting a similarity transform (translation, rotation, uniform scale) with perspective correction to map image pixels into the rectangular relative coordinate system defined by the fiducials — macroMS added optical correction tools to handle moderate distortion and imperfect perspective inherent to common imaging devices. Apply the fitted transform to all detected target pixel coordinates to produce instrument-relative X, Y positions for each sample well, honoring macroMS's ~10,000-target upper limit. Export the transformed coordinates in the macroMS geometry file format and feed this custom geometry file into the MS device so spectra are acquired at the selected locations. Validate by confirming the file parses as a structured coordinate table and that back-transforming the three fiducials reproduces their expected instrument positions within tolerance.

## Related tools

- **macroMS** (Web platform performing fiducial marking, optical correction, and device coordinate generation that converts target pixels to instrument-relative coordinates) — github.com/macroMS/macroMS_web
- **microMS** (Predecessor optically guided MS workflow whose approach macroMS adapts; alternative for sub-300 μm samples)

## Evaluation signals

- Output geometry file exists and parses as a structured coordinate table listing targets and fiducials in relative coordinates
- Back-transforming the three fiducial points through the fitted transform reproduces their expected instrument positions within tolerance
- Number of exported targets matches the number of detected/filtered targets and does not exceed ~10,000
- Geometry file is accepted by the MS device so spectra are acquired at the selected locations

## Limitations

- Optical correction only handles moderate distortion and imperfect perspective typical of flatbed scanners/cell phone cameras, not severe lens defects
- Upper target count limit of ~10,000 reported without measurement details or uncertainty
- Three-point fiducial registration constrains the transform model (no dense control-point refinement)

## Evidence
