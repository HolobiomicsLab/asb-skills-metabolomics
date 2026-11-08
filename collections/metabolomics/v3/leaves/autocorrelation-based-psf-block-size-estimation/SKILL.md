---
name: autocorrelation-based-psf-block-size-estimation
description: Use when a colocalization significance test requires scrambled randomizations
  of a two-color image (e.g., CFP/YFP or red/green confocal pairs) and the independent
  scrambling unit (block size) must match the image texture or PSF rather than individual
  pixels.
when_to_use_negative:
- The image is too small to contain many independent blocks at the estimated PSF/texture
  size, so block scrambling cannot generate a meaningful random-overlap distribution
- The goal is only M1/M2 coefficient computation from already-classified pixels, with
  no need for a scrambling-based significance threshold
- Pixels are actually independent (no measurable texture or PSF correlation), where
  individual-pixel scrambling would be the appropriate randomization unit
tools:
- name: MIPAV
  role: NIH software implementing the colocalization algorithm, including the autocorrelation-derived
    block size and 200 block-scrambled randomizations
- name: Bitplane
  role: Commercial implementation of the same colocalization algorithm whose block
    size is set from autocorrelation widths
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:d26f9a7c800a33bc432647cafcca7a7ba34e877bcfd994d1cbb8a436ca82fb70
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Carl Zeiss Axiovert 200
  - MIPAV
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# autocorrelation-based-psf-block-size-estimation

## Summary

Estimates the scrambled-block size used in the Costes et al. colocalization significance test by measuring the widths of the autocorrelations of the two color channels, so that randomization mimics image texture/PSF-scale correlation instead of destroying it.

## When to use

Use when a colocalization significance test requires scrambled randomizations of a two-color image (e.g., CFP/YFP or red/green confocal pairs) and the independent scrambling unit (block size) must match the image texture or PSF rather than individual pixels.

## When NOT to use

- The image is too small to contain many independent blocks at the estimated PSF/texture size, so block scrambling cannot generate a meaningful random-overlap distribution
- The goal is only M1/M2 coefficient computation from already-classified pixels, with no need for a scrambling-based significance threshold
- Pixels are actually independent (no measurable texture or PSF correlation), where individual-pixel scrambling would be the appropriate randomization unit

## Inputs

- Background-corrected two-color image pair (channel 1 and channel 2 intensity arrays, e.g., CFP/YFP or red/green from Zeiss LSM 510 acquisition)
- Autocorrelation profiles of each channel (used to derive the texture/PSF width)

## Outputs

- Block size in pixels equal to the smaller of the two autocorrelation widths
- 200 block-scrambled randomized image pairs per analyzed region
- Distribution of random colocalization overlap used to set the significance cutoff (e.g., 95th percentile)

## How to apply

Load the background-corrected two-color image pair (channels 1 and 2 as red and green intensity arrays, e.g., acquired on a Zeiss LSM 510 confocal microscope) and compute the autocorrelation of each channel separately. Measure the width of each autocorrelation and take the smaller of the two widths as the approximate size of the texture or PSF; this sets the edge length of the independent blocks. Divide the image into non-overlapping blocks of this size and, instead of scrambling individual pixels, scramble the blocks — 200 randomizations are performed for each region being analyzed. The rationale is that pixels within a texture- or PSF-sized region are not independent, so block scrambling preserves within-block correlation while generating a distribution of random overlap against which the real image's colocalization is compared (e.g., a 95th-percentile significance cutoff). The resulting threshold then feeds pixel classification (pixels closest to the diagonal line in the two-dimensional red-green histogram) and M1/M2 coefficient computation, written to colocalization_results.csv.

## Related tools

- **MIPAV** (NIH software implementing the colocalization algorithm, including the autocorrelation-derived block size and 200 block-scrambled randomizations)
- **Bitplane** (Commercial implementation of the same colocalization algorithm whose block size is set from autocorrelation widths)

## Examples

```
import numpy as np; from PIL import Image; red, green = np.array(Image.open('channel1.tif')), np.array(Image.open('channel2.tif'))
```

## Evaluation signals

- Block edge length equals the smaller of the two channel autocorrelation widths, not an arbitrary fixed size
- Exactly 200 block randomizations are performed for each analyzed region
- Selected significance cutoff matches the expected percentile of the random-overlap distribution (e.g., 95th)
- On simulated data with known colocalization, the downstream algorithm misclassifies only ~5% of pixels as false positives/false negatives
- Threshold selection is reproduced consistently across independent regions of the same image pair

## Limitations

- The algorithm still misclassified ~5% of pixels as false positives and false negatives on simulated data
- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can overpredict the amount of colocalization
- The block size is only an approximation of texture/PSF size, so images with unusual or anisotropic correlation structure may need manual verification

## Evidence

- [full_text] Images divided into independent blocks approximately the size of the texture or PSF, and blocks scrambled instead of individual pixels: "images are divided into independent blocks which are approximately the size of the texture or the PSF"
- [full_text] Autocorrelation width as the block-size criterion: [evidence span withheld — anchor status: not_found]
- [full_text] Block scrambling, not pixel scrambling: "Then instead of scrambling individual pixels, the blocks are scrambled."
- [full_text] Number of randomizations per region: "We performed 200 randomizations for each region being analyzed"
- [full_text] Validation error rate on simulated data: "Our algorithm resulted in a small proportion (∼5%) of pixels being misclassified as false-positives and false-negatives"
