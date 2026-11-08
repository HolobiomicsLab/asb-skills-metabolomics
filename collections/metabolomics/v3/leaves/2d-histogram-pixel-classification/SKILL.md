---
name: 2d-histogram-pixel-classification
description: Use when you have background-corrected two-channel (e.g. red/green) confocal
  images and need to separate colocalized pixels from non-colocalized signal or background
  to compute per-protein colocalization coefficients.
when_to_use_negative:
- Single-channel images or images without a paired second fluorophore — the 2D histogram
  requires two color channels
- Uncorrected images with substantial background — threshold selection depends on
  background-corrected intensities
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MIPAV
  role: Software implementing the colocalization algorithm for intramural NIH research
- name: Bitplane
  role: Commercial implementation of the automatic colocalization threshold algorithm
- name: Zeiss LSM 510 confocal microscope
  role: Acquisition of the two-channel live-cell images that feed the classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1529/biophysj.103.038422
    title: Automatic and Quantitative Measurement of Protein-Protein Colocalization
      in Live Cells
  source_content_hash: sha256:9e69bb2145c3042c37d502c15c983c3d6ff6b5c9b6b712ab0b70bd90a5baccac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Bitplane
  - MIPAV
  - Zeiss LSM 510 confocal microscope
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# 2D Histogram Pixel Classification

## Summary

Classifies pixels in a two-color image as colocalized or non-colocalized using a thresholded two-dimensional red/green intensity histogram, enabling quantitative colocalization coefficients to be computed from live-cell confocal data.

## When to use

Use when you have background-corrected two-channel (e.g. red/green) confocal images and need to separate colocalized pixels from non-colocalized signal or background to compute per-protein colocalization coefficients.

## When NOT to use

- Single-channel images or images without a paired second fluorophore — the 2D histogram requires two color channels
- Uncorrected images with substantial background — threshold selection depends on background-corrected intensities

## Inputs

- Background-corrected two-channel confocal image (e.g. red and green)
- PSF or texture size estimate for block dimensions

## Outputs

- Per-pixel colocalized / non-colocalized classification
- Two-dimensional intensity histogram with thresholded region marked
- Colocalization coefficients (proportion of each protein colocalized with the other)

## How to apply

Build a two-dimensional histogram of pixel intensities across the two channels (e.g. red and green). Apply the Costes automatic threshold by dividing the image into blocks approximately the size of the texture or PSF — sized from the narrower autocorrelation width — scrambling those blocks rather than individual pixels, and performing 200 randomizations per region to generate the random-overlap distribution from which the threshold is selected. Classify pixels falling within the thresholded region of the 2D histogram as colocalized; to distinguish low-intensity colocalized pixels from background, select pixels closest to the diagonal line in the histogram. Report the proportion of each protein colocalized with the other, and be aware that treating selected pixels as entirely colocalized can overpredict colocalization when real pixel intensities are sums of colocalized and non-colocalized signals.

## Related tools

- **MIPAV** (Software implementing the colocalization algorithm for intramural NIH research)
- **Bitplane** (Commercial implementation of the automatic colocalization threshold algorithm)
- **Zeiss LSM 510 confocal microscope** (Acquisition of the two-channel live-cell images that feed the classification)

## Evaluation signals

- On simulated data with known ground truth, misclassified pixels (false positives plus false negatives) should be ~5% of total pixels
- Results should correlate with FRET measurements of the same protein groups
- Non-negative overprediction (computed coefficient minus true colocalized fraction) across mixed-signal conditions
- Background correction applied before histogram construction — uncorrected intensities invalidate threshold selection

## Limitations

- Assuming pixels in the selected histogram region are entirely colocalized can overpredict colocalization, since real pixel intensities may sum colocalized and non-colocalized components
- FRET alone would only detect interaction if the average distance between binding proteins is <10 nm — colocalization coefficients do not imply direct binding
- Low-intensity colocalized pixels are easily confused with background; the diagonal-line selection step is required to separate them

## Evidence

- [full_text] Colocalization in a two-color image can be visualized in a two-dimensional histogram, where the number of pixels with red and green intensities: "Colocalization in a two-color image (e.g., red and green) can be visualized in a two-dimensional histogram, where the number of pixels with red and green intensities"
- [full_text] Images divided into blocks and blocks scrambled: [evidence span withheld — anchor status: not_found]
- [full_text] 200 randomizations performed per region: "We performed 200 randomizations for each region being analyzed"
- [full_text] Pixels closest to the diagonal line selected: "We selected pixels closest to the diagonal line in the two-dimensional histogram"
- [full_text] Overprediction from assuming entirely colocalized pixels: "assuming that the selected points in the two-dimensional histogram are entirely colocalized, this can actually lead to overprediction of the amount of colocalization (data not shown)"
- [full_text] 5% misclassified pixels: "Our algorithm resulted in a small proportion (∼5%) of pixels being misclassified as false-positives and false-negatives"
