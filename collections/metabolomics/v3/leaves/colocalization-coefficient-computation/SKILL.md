---
name: colocalization-coefficient-computation
description: Use when you need an automatic, statistically justified pixel-level threshold
  and M1/M2 colocalization coefficients from a background-corrected red/green (e.g.,
  CFP/YFP) two-channel microscopy image, including cases where colocalization is not
  visually obvious.
when_to_use_negative:
- When pixel intensities are sums of colocalized and non-colocalized signals and an
  unbiased magnitude is required — assuming the selected histogram points are entirely
  colocalized can overpredict the amount of colocalization
- When the interaction question requires sub-10-nm proximity evidence — FRET, not
  pixel-overlap coefficients, is the appropriate measurement
- When the two channels are not background-corrected or the image texture/PSF size
  cannot be estimated from autocorrelation widths, since block size determines randomization
  validity
tools:
- name: MIPAV
  role: Software implementing the automatic colocalization algorithm for computing
    thresholds and coefficients
- name: Bitplane
  role: Commercial implementation of the algorithm available to the NIH research community
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of the two-color (CFP/YFP) confocal images the algorithm is applied
    to
- name: Carl Zeiss Axiovert 200
  role: Microscope platform equipped with the LSM 510 for live-cell image acquisition
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_002
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:11c98f09c12d2302a52a1e58f853551be6ac7369f120eaa159ec2c34fae7f4a8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Carl Zeiss Axiovert 200
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# colocalization-coefficient-computation

## Summary

Computes Costes-style colocalization coefficients (the proportion of each protein colocalized with the other) from background-corrected two-color images by thresholding colocalized pixels in the two-dimensional red/green intensity histogram using a block-scrambled randomization test, validated at ~5% pixel misclassification.

## When to use

Use when you need an automatic, statistically justified pixel-level threshold and M1/M2 colocalization coefficients from a background-corrected red/green (e.g., CFP/YFP) two-channel microscopy image, including cases where colocalization is not visually obvious.

## When NOT to use

- When pixel intensities are sums of colocalized and non-colocalized signals and an unbiased magnitude is required — assuming the selected histogram points are entirely colocalized can overpredict the amount of colocalization
- When the interaction question requires sub-10-nm proximity evidence — FRET, not pixel-overlap coefficients, is the appropriate measurement
- When the two channels are not background-corrected or the image texture/PSF size cannot be estimated from autocorrelation widths, since block size determines randomization validity

## Inputs

- Background-corrected two-channel microscopy image pair (e.g., CFP/YFP acquired on a Zeiss LSM 510 confocal microscope with 458/514 nm Argon laser excitation), as red and green intensity arrays
- Simulated two-color image series spanning full (100%) to no (0%) colocalization for validation
- Mixed-signal two-channel variants where each pixel intensity is a sum of colocalized and non-colocalized components with known fractions

## Outputs

- Per-pixel binary colocalization classification (colocalized vs. non-colocalized) derived from the selected region of the two-dimensional histogram
- M1/M2 colocalization coefficients (proportion of each protein colocalized with the other)
- Automatic significance threshold selected from the 200-randomization random-overlap distribution
- CSV tables such as colocalization_results.csv (per-pixel classification and coefficients) and overprediction_analysis.csv (computed coefficient minus true mixed fraction)

## How to apply

Load the background-corrected two-color image pair and build the two-dimensional histogram of pixel red/green intensity pairs. Determine the colocalization threshold by dividing the image into independent blocks approximately the size of the texture or PSF — measured as the smaller of the widths of the autocorrelations of the two images — scrambling the blocks (not individual pixels) to account for correlations in adjacent pixel intensities, and performing 200 randomizations per analyzed region to generate the distribution of random overlap. Select colocalized pixels from the region of the two-dimensional histogram above the random-overlap significance cutoff, additionally selecting pixels closest to the diagonal line to distinguish low-intensity colocalized pixels from background. From the classified pixels, compute the proportion of each protein colocalized with the other (M1/M2 coefficients). Validate on simulated series spanning 100% to 0% colocalization, where the algorithm should misclassify only about 5% of pixels as false-positives/false-negatives and detect subvisually obvious colocalization.

## Related tools

- **MIPAV** (Software implementing the automatic colocalization algorithm for computing thresholds and coefficients)
- **Bitplane** (Commercial implementation of the algorithm available to the NIH research community)
- **Zeiss LSM 510 confocal microscope** (Acquisition of the two-color (CFP/YFP) confocal images the algorithm is applied to)
- **Carl Zeiss Axiovert 200** (Microscope platform equipped with the LSM 510 for live-cell image acquisition)

## Examples

```
import numpy as np; from PIL import Image; red=np.array(Image.open('ch1.tif')); green=np.array(Image.open('ch2.tif')); H,_,_=np.histogram2d(red.ravel(),green.ravel(),bins=256)
```

## Evaluation signals

- On simulated series spanning 100% to 0% colocalization, the proportion of misclassified pixels (false-positives plus false-negatives) should be approximately 5%
- The automatic threshold should match a high percentile (e.g., 95th) of the 200-randomization random-overlap distribution for each analyzed region
- Computed coefficients should increase monotonically across the simulated colocalization series from 0% to 100%
- The algorithm should flag colocalization in the simulated subvisual case where colocalization was not obvious visually
- Overprediction (computed coefficient minus true colocalized fraction) should be non-negative across the mixed-signal conditions

## Limitations

- Forcing each pixel to be either entirely colocalized or entirely non-colocalized assumes selected histogram points are entirely colocalized, which can lead to overprediction of the amount of colocalization because real pixel intensities are sums of colocalized and non-colocalized signals
- Randomization must scramble blocks approximately the size of the texture or PSF rather than individual pixels; incorrectly sized blocks invalidate the random-overlap distribution
- FRET-based validation may miss interactions due to protein size (e.g., FRET was negative for Rev-CRM1, presumably due to the large size of CRM1, 130 kDa), so coefficient agreement with FRET is not a universal ground truth

## Evidence

- [full_text] Block scrambling rationale: [evidence span withheld — anchor status: not_found]
