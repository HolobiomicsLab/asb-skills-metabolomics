---
name: block-scrambling-randomization-statistics
description: Use when analyzing background-corrected two-color (e.g., CFP/YFP) 2D
  or 3D confocal images to determine which pixels are truly colocalized versus random
  overlap, and to compute the proportion of each protein colocalized with the other.
when_to_use_negative:
- When the question is physical protein-protein binding distance rather than colocalization
  — FRET is required (FRET detects interaction only if average distance <10 nm, and
  FRET was negative for Rev-CRM1 despite colocalization, presumably due to CRM1's
  130 kDa size).
- When the input is a single-channel image or the two channels have not been background-corrected
  — the 2D histogram selection and random-overlap distribution require paired, corrected
  two-color data.
- When the image texture/PSF cannot be estimated (no measurable autocorrelation widths)
  — block size depends on the smaller autocorrelation width of the two channels.
edam_operation: http://edamontology.org/operation_3443
tools:
- name: MIPAV
  role: Implementation platform for the automatic colocalization algorithm (block
    scrambling, 200 randomizations, threshold selection) available to NIH researchers
- name: Bitplane
  role: Commercial implementation of the same colocalization algorithm used to apply
    the method to HeLa image sets
- name: Zeiss LSM 510 confocal microscope
  role: Acquires the CFP/YFP two-channel confocal stacks (458 and 514 nm excitation)
    that the randomization analysis consumes
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:a09e84d3c2b7bb49f6ab1ab20a3335069166167af77c0941d4f7c5d7ed76709a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Effectene
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# block-scrambling-randomization-statistics

## Summary

A statistical colocalization-threshold method from Costes et al. (Biophys J 2004) that builds a random-overlap distribution by scrambling image blocks (not pixels) and automatically selects colocalized pixels in a 2D red/green intensity histogram, yielding colocalization coefficients without user-set thresholds.

## When to use

Use when analyzing background-corrected two-color (e.g., CFP/YFP) 2D or 3D confocal images to determine which pixels are truly colocalized versus random overlap, and to compute the proportion of each protein colocalized with the other.

## When NOT to use

- When the question is physical protein-protein binding distance rather than colocalization — FRET is required (FRET detects interaction only if average distance <10 nm, and FRET was negative for Rev-CRM1 despite colocalization, presumably due to CRM1's 130 kDa size).
- When the input is a single-channel image or the two channels have not been background-corrected — the 2D histogram selection and random-overlap distribution require paired, corrected two-color data.
- When the image texture/PSF cannot be estimated (no measurable autocorrelation widths) — block size depends on the smaller autocorrelation width of the two channels.

## Inputs

- Background-corrected two-channel (e.g., CFP/YFP) 2D or 3D confocal image stacks (e.g., Zeiss LSM 510 acquisitions)
- Per-region region-of-interest definitions (e.g., nucleolus, whole nucleus) for live-cell time series
- Simulated two-color image pairs spanning 100% to 0% colocalization for validation

## Outputs

- Automatic colocalization threshold per region derived from the 200-randomization random-overlap distribution
- Per-pixel colocalized/non-colocalized classification in the 2D intensity histogram
- Colocalization coefficients: proportion of each protein colocalized with the other (per cell, per region, per time point)

## How to apply

First background-correct the two-channel image stack (e.g., CFP/YFP confocal data acquired with 458 and 514 nm Argon laser lines). Divide each image into independent blocks approximately the size of the texture or the PSF, measured as the smaller of the widths of the autocorrelations of the two channels — this accounts for correlated adjacent pixel intensities, so blocks are scrambled rather than individual pixels. Perform 200 randomizations for each region being analyzed to build the distribution of random overlap, then use that distribution as the automatic statistical criterion to select the colocalization threshold. Classify pixels in the two-dimensional red/green intensity histogram, selecting pixels closest to the diagonal line to distinguish low-intensity colocalized pixels from background. Compute the colocalization coefficients (the proportion of each protein colocalized with the other) from the selected pixels. Rationale: pixel-level scrambling would break the natural adjacent-pixel intensity correlations, inflating false thresholds; block scrambling preserves within-block texture while randomizing spatial relationship between channels.

## Related tools

- **MIPAV** (Implementation platform for the automatic colocalization algorithm (block scrambling, 200 randomizations, threshold selection) available to NIH researchers)
- **Bitplane** (Commercial implementation of the same colocalization algorithm used to apply the method to HeLa image sets)
- **Zeiss LSM 510 confocal microscope** (Acquires the CFP/YFP two-channel confocal stacks (458 and 514 nm excitation) that the randomization analysis consumes)

## Evaluation signals

- On simulated images spanning 100% to 0% colocalization, misclassified pixels (false positives + false negatives) should be ~5%.
- Coefficients for the random-overlap negative control (e.g., Rev-CFP + free YFP in fixed HeLa cells) must fall within the random-overlap distribution.
- Block size must approximate the smaller of the two channels' autocorrelation widths, and exactly 200 randomizations must be run per region analyzed.
- Group-level results on three groups of 40 fixed HeLa cells (known interaction, co-compartmentalization, random overlap) should correlate with independent FRET measurements.
- In the subvisual test case, the automatic detection should flag colocalization that is not obvious to the eye.

## Limitations

- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization, because real pixel intensities are sums of colocalized and non-colocalized signals.
- Scrambling individual pixels instead of blocks would ignore correlations in adjacent pixel intensities and invalidate the random-overlap distribution.
- Threshold choice depends on the randomization statistics; regions smaller than the texture/PSF block size cannot be reliably randomized.

## Evidence
