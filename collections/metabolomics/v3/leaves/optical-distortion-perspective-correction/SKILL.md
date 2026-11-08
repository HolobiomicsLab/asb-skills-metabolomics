---
name: optical-distortion-perspective-correction
description: Use when sample positions were detected in images from a flatbed scanner
  or cell phone camera of a macroscopic (>300 μm) plate and must be converted to device-relative
  coordinates for MS geometry files, especially when the imaging device introduces
  moderate distortion or imperfect perspective.
when_to_use_negative:
- Samples are smaller than 300 μm — the article states microMS is more effective at
  that scale and microscope-based imaging is the appropriate route
- The image already conforms to the instrument coordinate system with no fiducial
  correspondences needed or the imaging device has severe (not moderate) distortion
  beyond what the correction tools address
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: macroMS
  role: Web platform whose optical correction tools fix moderate distortion and imperfect
    perspective and generate device coordinate files from fiducial correspondences
  repo: github.com/macroMS/macroMS_web
- name: microMS
  role: Prior optically guided MS software whose workflow macroMS adapted for macroscopic
    samples; alternative for samples below 300 μm
- name: flexImaging
  role: Commercial instrument software suite that lacks support for large numbers
    of samples, motivating this fiducial-based correction approach
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.1c00013
    title: 'macroMS: Image-Guided Analysis of Random Objects by Matrix-Assisted Laser
      Desorption/Ionization Time-of-Flight Mass Spectrometry'
  source_content_hash: sha256:f7e4633e3cece195723524d88ed24810529973d7947b96bfb4e36b6aaba6251c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - flexImaging
  - macroMS
  - microMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# optical-distortion-perspective-correction

## Summary

Corrects moderate lens distortion and imperfect perspective in optical images of macroscopic sample plates using fiducial-based registration, so detected target pixel coordinates can be converted into accurate instrument-relative coordinates for optically guided MALDI-TOF MS.

## When to use

Use when sample positions were detected in images from a flatbed scanner or cell phone camera of a macroscopic (>300 μm) plate and must be converted to device-relative coordinates for MS geometry files, especially when the imaging device introduces moderate distortion or imperfect perspective.

## When NOT to use

- Samples are smaller than 300 μm — the article states microMS is more effective at that scale and microscope-based imaging is the appropriate route
- The image already conforms to the instrument coordinate system with no fiducial correspondences needed or the imaging device has severe (not moderate) distortion beyond what the correction tools address

## Inputs

- Optical image of the sample plate from a flatbed scanner or cell phone camera
- Pixel positions of the three corner fiducial marks with known instrument coordinates
- Detected target pixel coordinates (up to ~10,000 targets) from the sample-finding workflow

## Outputs

- Custom macroMS geometry file listing targets and fiducials in instrument-relative coordinates
- Perspective- and distortion-corrected spatial locations for each selected target

## How to apply

Mark the three corner fiducial points of known instrument coordinates on the sample optical image, establishing fiducial correspondences between pixels and the instrument-relative coordinate system. Solve the registration (translation, rotation, uniform scale, and perspective correction) from these three correspondences using the macroMS optical correction tools, which were added specifically to resolve defects inherent to common imaging devices such as moderate distortion and imperfect perspective. Apply the fitted transform to all detected target pixel coordinates so each sample well gets an instrument-relative X, Y position. Export the transformed coordinates as a custom geometry file that the MS device consumes; macroMS generates these device coordinates through its web interface. Back-transform the three fiducial points through the fitted transform and confirm they reproduce their expected instrument positions within tolerance before trusting the target coordinates.

## Related tools

- **macroMS** (Web platform whose optical correction tools fix moderate distortion and imperfect perspective and generate device coordinate files from fiducial correspondences) — github.com/macroMS/macroMS_web
- **microMS** (Prior optically guided MS software whose workflow macroMS adapted for macroscopic samples; alternative for samples below 300 μm)
- **flexImaging** (Commercial instrument software suite that lacks support for large numbers of samples, motivating this fiducial-based correction approach)

## Evaluation signals

- Back-transforming the three fiducial pixel points through the fitted transform reproduces their expected instrument positions within tolerance
- Exported geometry file exists and parses as a structured coordinate table (targets plus fiducials in relative coordinates)
- Every transformed target X, Y falls within the instrument-relative coordinate bounds implied by the three fiducials
- Corrected positions no longer exhibit the moderate distortion/imperfect perspective present in the raw scanner or phone-camera image

## Limitations

- Correction handles only moderate distortion; severe lens defects may not be fully resolved
- Upper limit of ~10,000 targets, reported without measurement details or uncertainty
- No quantitative benchmark against microMS for coordinate accuracy; targeting reliability demonstrated at the 300 μm sample scale

## Evidence
