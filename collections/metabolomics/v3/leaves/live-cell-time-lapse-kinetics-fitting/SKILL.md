---
name: live-cell-time-lapse-kinetics-fitting
description: Use when a background-corrected two-color (e.g. CFP/YFP) live-cell time-lapse
  confocal series is available and you need time-resolved colocalization proportions
  plus a fitted dissociation rate after a perturbation such as LMB.
when_to_use_negative:
- The question requires direct measurement of physical binding distance rather than
  colocalization dynamics — FRET was negative for the Rev-CRM1 complex because CRM1
  is 130 kDa, and FRET alone only detects interaction when the average distance is
  <10, so neither approach alone substitutes for the other.
- The input is a static single-time-point image with no temporal dimension — the kinetics-fitting
  step requires a time series spanning before and after perturbation.
- Images cannot be background-corrected or lack independent red/green channels, since
  thresholding relies on scrambling blocks of a two-color image to build the random-overlap
  distribution.
tools:
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the live-cell Rev-CFP/YFP-CRM1 time-lapse series (Argon laser 458/514
    nm excitation, 37°C, 5% CO2) whose time course feeds the kinetics fitting
- name: MIPAV
  role: NIH software platform hosting the Costes colocalization algorithm used to
    compute the per-time-point nucleolar proportions that are then fitted
- name: Bitplane
  role: Commercial implementation of the automatic colocalization thresholding algorithm
    applied to the time-lapse images
- name: Effectene
  role: Transfection reagent used to cotransfect HeLa cells with pRev-CFP and pYFP-CRM1
    to generate the live-cell samples measured over time
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:867d39d5c77c658c1e0822c2cb289b3e2e2c4b553b2ec18f1e008d449a44d4a3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Effectene
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# live-cell-time-lapse-kinetics-fitting

## Summary

Quantifies how a perturbation (e.g. leptomycin B) changes the nucleolar colocalization proportions of two fluorescently tagged proteins across a live-cell confocal time-lapse, then fits the decay of colocalization to extract a protein-protein interaction dissociation rate.

## When to use

Use when a background-corrected two-color (e.g. CFP/YFP) live-cell time-lapse confocal series is available and you need time-resolved colocalization proportions plus a fitted dissociation rate after a perturbation such as LMB.

## When NOT to use

- The question requires direct measurement of physical binding distance rather than colocalization dynamics — FRET was negative for the Rev-CRM1 complex because CRM1 is 130 kDa, and FRET alone only detects interaction when the average distance is <10, so neither approach alone substitutes for the other.
- The input is a static single-time-point image with no temporal dimension — the kinetics-fitting step requires a time series spanning before and after perturbation.
- Images cannot be background-corrected or lack independent red/green channels, since thresholding relies on scrambling blocks of a two-color image to build the random-overlap distribution.

## Inputs

- Background-corrected two-color (Rev-CFP green / YFP-CRM1 red) time-lapse confocal image series from a Zeiss LSM 510
- Time point (frame) at which leptomycin B (LMB) was added to the live HeLa cells
- Segmented nucleolar and whole-nucleus regions of interest for each time point

## Outputs

- Per-time-point proportion of Rev in the nucleolus and proportion of CRM1 in the nucleolus
- Colocalized-pixel classification from the two-dimensional histogram (pixels closest to the diagonal line)
- Fitted dissociation rate of the Rev-CRM1 interaction from the decay of colocalization

## How to apply

Load the background-corrected Rev-CFP/YFP-CRM1 image series acquired on a Zeiss LSM 510 (CFP and YFP excited with Argon laser lines at 458 and 514 nm, cells held at 37°C and 5% CO2). Determine the automatic colocalization threshold by dividing each image into independent blocks approximately the size of the texture or PSF, scrambling blocks (not individual pixels) with 200 randomizations per region to build the random-overlap distribution. Classify colocalized pixels in the two-dimensional red/green intensity histogram by selecting pixels closest to the diagonal line, then compute, per time point, the proportion of Rev in the nucleolus and the proportion of CRM1 in the nucleolus. Track the two nucleolar proportions across the series before and after LMB addition and fit the decay of colocalization to obtain the dissociation rate of the Rev-CRM1 interaction. Validate by reproducing the reported directionality: after LMB, the Rev nucleolar proportion rises while the CRM1 proportion falls and diffuses into the whole nucleus.

## Related tools

- **Zeiss LSM 510 confocal microscope** (Acquires the live-cell Rev-CFP/YFP-CRM1 time-lapse series (Argon laser 458/514 nm excitation, 37°C, 5% CO2) whose time course feeds the kinetics fitting)
- **MIPAV** (NIH software platform hosting the Costes colocalization algorithm used to compute the per-time-point nucleolar proportions that are then fitted)
- **Bitplane** (Commercial implementation of the automatic colocalization thresholding algorithm applied to the time-lapse images)
- **Effectene** (Transfection reagent used to cotransfect HeLa cells with pRev-CFP and pYFP-CRM1 to generate the live-cell samples measured over time)

## Evaluation signals

- Directional check passes: after LMB addition the proportion of Rev in the nucleolus increases while the proportion of CRM1 decreases in the nucleolus and diffuses into the whole nucleus.
- The fitted dissociation rate is consistent with the shape of the measured colocalization time-course (decay after LMB addition).
- Automatic thresholding used 200 block-scrambling randomizations per region, with blocks sized approximately to the texture or PSF rather than individual pixels.
- Colocalized pixels were selected as those closest to the diagonal line in the two-dimensional intensity histogram, avoiding overprediction from assuming all diagonal-adjacent points are fully colocalized.
- Misclassification of colocalized pixels stays near the reported ~5% false-positive/false-negative rate on the analyzed regions.

## Limitations

- Assuming that selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization.
- The algorithm misclassified ~5% of pixels as false positives and false negatives on simulated data.
- The method reports colocalization, not direct binding: FRET was negative for the Rev-CRM1 complex, presumably due to the large size of CRM1 (130 kDa), so a positive colocalization result does not prove molecular contact.

## Evidence
