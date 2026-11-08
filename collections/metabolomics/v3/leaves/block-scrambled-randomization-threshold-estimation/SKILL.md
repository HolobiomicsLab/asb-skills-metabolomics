---
name: block-scrambled-randomization-threshold-estimation
description: Use when a background-corrected two-color (red/green) fluorescence image
  pair needs an automated colocalization significance threshold, and visual inspection
  is ambiguous.
when_to_use_negative:
- 'Measuring protein-protein interaction proximity: colocalization coefficients quantify
  overlap, not binding — FRET is required to detect interaction when the average distance
  between binding proteins is <10 nm, and FRET was negative for the Rev-CRM1 complex
  even where the algorithm applied.'
- Input is a single-channel image or a non-registered two-color pair with no red-green
  intensity pairing to build the 2D histogram from.
- Scrambling individual pixels instead of texture/PSF-sized blocks — this violates
  the randomization design and inflates significance of spatially autocorrelated signals.
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIPAV
  role: Software implementation made available to the NIH intramural community in
    which this colocalization algorithm (block scrambling, threshold selection, M1/M2
    computation) runs.
- name: Bitplane
  role: Commercial software package in which the algorithm was made available as an
    alternative implementation.
- name: Zeiss LSM 510 confocal microscope
  role: Instrument used to acquire the two-color (e.g. CFP/YFP excited at 458 and
    514 nm) images the threshold estimation operates on.
- name: Carl Zeiss Axiovert 200
  role: Microscope base (40x 1.3 NA plan-NEOFLUAR oil DIC objective) on which the
    LSM 510 acquisition was performed.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:9555fa2bd524efe86818f23fc6dccdf71d989ccba9e22bae5696ca5fe6bcc828
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - Carl Zeiss Axiovert 200
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# block-scrambled-randomization-threshold-estimation

## Summary

Estimates a statistically significant colocalization threshold for two-color fluorescence images by scrambling image blocks (sized by the texture/PSF from image autocorrelations) 200 times and selecting a cutoff (e.g., the 95th percentile of the random overlap distribution), as described by Costes et al. 2004.

## When to use

Use when a background-corrected two-color (red/green) fluorescence image pair needs an automated colocalization significance threshold, and visual inspection is ambiguous.

## When NOT to use

- Measuring protein-protein interaction proximity: colocalization coefficients quantify overlap, not binding — FRET is required to detect interaction when the average distance between binding proteins is <10 nm, and FRET was negative for the Rev-CRM1 complex even where the algorithm applied.
- Input is a single-channel image or a non-registered two-color pair with no red-green intensity pairing to build the 2D histogram from.
- Scrambling individual pixels instead of texture/PSF-sized blocks — this violates the randomization design and inflates significance of spatially autocorrelated signals.

## Inputs

- Background-corrected two-color fluorescence image pair (channel 1 red intensity array, channel 2 green intensity array; e.g. CFP/YFP channels from a Zeiss LSM 510 confocal microscope)
- Image texture or PSF size estimated from the smaller of the widths of the two images' autocorrelations
- Number of randomizations (200) and significance percentile (e.g. 95th) for the cutoff

## Outputs

- Per-pixel colocalization classification (colocalized vs non-colocalized) in the red-green 2D histogram
- M1/M2 colocalization coefficients written to colocalization_results.csv

## How to apply

Load the background-corrected two-color image pair (channels 1 and 2 as red and green intensity arrays, e.g. from a Zeiss LSM 510 confocal acquisition) with NumPy/PIL and build the two-dimensional histogram of red-green intensity pairs. Measure the widths of the autocorrelations of the two images, take the smaller of the two widths as the size of independent image blocks approximating the texture or PSF, and scramble these blocks (not individual pixels) to generate 200 randomizations per analyzed region. From the distribution of random overlap, select the significance cutoff, e.g. the 95th percentile, and classify pixels as colocalized or non-colocalized in the 2D histogram. When classifying, select pixels closest to the diagonal line of the histogram so that low-intensity colocalized signal is distinguished from background. Finally compute the M1/M2 coefficients — the proportion of each protein colocalized with the other — from the classified pixel intensities.

## Related tools

- **MIPAV** (Software implementation made available to the NIH intramural community in which this colocalization algorithm (block scrambling, threshold selection, M1/M2 computation) runs.)
- **Bitplane** (Commercial software package in which the algorithm was made available as an alternative implementation.)
- **Zeiss LSM 510 confocal microscope** (Instrument used to acquire the two-color (e.g. CFP/YFP excited at 458 and 514 nm) images the threshold estimation operates on.)
- **Carl Zeiss Axiovert 200** (Microscope base (40x 1.3 NA plan-NEOFLUAR oil DIC objective) on which the LSM 510 acquisition was performed.)

## Examples

```
import numpy as np; H, red_edges, green_edges = np.histogram2d(red.ravel(), green.ravel(), bins=256)  # build the red-green 2D histogram before block-scrambled thresholding
```

## Evaluation signals

- On simulated data, ~5% of pixels should be misclassified as false positives and false negatives — substantially more misclassification indicates the block size or percentile cutoff is wrong.
- Exactly 200 randomizations of autocorrelation-sized blocks should be generated for each analyzed region, not pixel-level scrambling.
- The selected cutoff should match the chosen percentile (e.g., 95th) of the 200-randomization random overlap distribution.
- M1 and M2 coefficients must each be computable per channel and derivable from the classified pixel intensities; verify they are consistent with the per-pixel masks.
- The algorithm should detect colocalization in images where colocalization was not obvious visually, and results on known protein-protein interactions vs random overlap should correlate with independent FRET measurements.

## Limitations

- Assuming that the selected points in the two-dimensional histogram are entirely colocalized can lead to overprediction of the amount of colocalization.
- The algorithm misclassifies a small proportion (~5%) of pixels as false positives and false negatives even on simulated data.
- Colocalization detected here does not establish direct molecular interaction at <10 nm distances; FRET measurements are needed for that and may be negative for large proteins such as CRM1 (130 kDa).

## Evidence

- [methods] 200 randomizations define the null distribution: "We performed 200 randomizations for each region being analyzed"
- [methods] blocks sized by texture/PSF are scrambled rather than pixels: [evidence span withheld — anchor status: not_found]
- [results] expected misclassification rate on simulated data: "Our algorithm resulted in a small proportion (∼5%) of pixels being misclassified as false-positives and false-negatives"
