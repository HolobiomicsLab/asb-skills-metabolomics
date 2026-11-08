---
name: image-coordinate-similarity-registration
description: Use when detected target pixel positions from an optical image (flatbed
  scanner or cell phone camera) must be converted to instrument-relative coordinates
  via fiducial registration, e.g. to generate macroMS device geometry files for optically
  guided MALDI-TOF targeting.
when_to_use_negative:
- Samples smaller than 300 μm, where the article states microMS is more effective
- Images with distortion beyond the 'moderate distortion and imperfect perspective'
  that macroMS's optical correction tools are designed to resolve
- Target sets exceeding the ~10,000 target upper limit reported for macroMS
tools:
- name: macroMS
  role: Web platform providing fiducial marking, optical correction, and device coordinate
    generation that implements the pixel-to-instrument transform
  repo: https://github.com/macroMS/macroMS_web
- name: macroMS_web
  role: Source repository for macroMS web application and its sample data (fiducial
    pixel positions and images)
  repo: https://github.com/macroMS/macroMS_web
- name: microMS
  role: Predecessor optically guided MS workflow for sub-300 μm samples; macroMS adapts
    its fiducial-based coordinate approach to macroscopic objects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:fda13db162133819535ab46a7490e130f9a71ef83de8aea401d15cb2c07527ad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/macroMS/macroMS_web
  tools:
  - macroMS
  - macroMS_web
  - microMS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# image-coordinate-similarity-registration

## Summary

Fits a similarity transform (translation, rotation, uniform scale) plus perspective correction from three fiducial marks to map optical image pixel coordinates into the instrument-relative coordinate system used by macroMS, producing the custom geometry file consumed by the MS device.

## When to use

Use when detected target pixel positions from an optical image (flatbed scanner or cell phone camera) must be converted to instrument-relative coordinates via fiducial registration, e.g. to generate macroMS device geometry files for optically guided MALDI-TOF targeting.

## When NOT to use

- Samples smaller than 300 μm, where the article states microMS is more effective
- Images with distortion beyond the 'moderate distortion and imperfect perspective' that macroMS's optical correction tools are designed to resolve
- Target sets exceeding the ~10,000 target upper limit reported for macroMS

## Inputs

- Optical image of the sample plate (from a flatbed scanner or cell phone camera)
- Pixel positions of the three fiducial marks with known instrument coordinates
- Pixel coordinates of detected (and filtered) sample targets

## Outputs

- Instrument-relative X, Y positions for each target
- macroMS custom geometry file listing targets and fiducials in relative coordinates, consumable by the MS device

## How to apply

Load the sample plate optical image along with the pixel positions of the three corner fiducial marks whose instrument coordinates are known, taken from macroMS sample data in the macroMS_web repository. Solve the registration over the three fiducial correspondences to obtain a similarity transform (translation, rotation, uniform scale) with the perspective correction macroMS added to handle moderate distortion and imperfect perspective from common imaging devices. Apply the fitted transform to every detected target pixel coordinate to yield instrument-relative X, Y positions for each sample well. Export the result as the macroMS geometry file — a downloadable table listing targets and fiducials in relative coordinates — that the instrument interprets to acquire spectra at selected locations. Validate by checking the file exists, parses as a structured coordinate table, and that back-transforming the three fiducial points reproduces their expected instrument positions within tolerance.

## Related tools

- **macroMS** (Web platform providing fiducial marking, optical correction, and device coordinate generation that implements the pixel-to-instrument transform) — https://github.com/macroMS/macroMS_web
- **macroMS_web** (Source repository for macroMS web application and its sample data (fiducial pixel positions and images)) — https://github.com/macroMS/macroMS_web
- **microMS** (Predecessor optically guided MS workflow for sub-300 μm samples; macroMS adapts its fiducial-based coordinate approach to macroscopic objects)

## Evaluation signals

- The exported geometry file exists and parses as a structured coordinate table (targets plus fiducials in relative coordinates)
- Back-transforming the three fiducial points through the fitted transform reproduces their expected instrument positions within tolerance
- Samples of ~300 μm are targeted reliably through the full workflow
- Target count stays within the reported ~10,000 upper limit
- Resulting geometry file is accepted by the MS device, which then acquires spectra at the selected locations

## Limitations

- Optical correction resolves only moderate distortion and imperfect perspective inherent to common imaging devices; severe lens distortion is out of scope
- Reported upper limit of ~10,000 targets is stated without measurement details or uncertainty
- Registration relies on exactly three fiducial correspondences, so fiducial placement error propagates directly into the transform

## Evidence
